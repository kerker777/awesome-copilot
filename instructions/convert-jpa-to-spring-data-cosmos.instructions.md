---
description: 'Step-by-step guide for converting Spring Boot JPA applications to use Azure Cosmos DB with Spring Data Cosmos'
applyTo: '**/*.java,**/pom.xml,**/build.gradle,**/application*.properties'
---

# 將 Spring JPA 專案轉換為 Spring Data Cosmos

本通用指南適用於任何 JPA 到 Spring Data Cosmos DB 轉換的專案。

## 高階計畫

1. 替換建置相依性（移除 JPA，新增 Cosmos + Identity）。
2. 新增 `cosmos` 設定檔和屬性。
3. 新增具有正確 Azure 身分識別認證的 Cosmos 配置。
4. 轉換實體（ID → `String`，新增 `@Container` 和 `@PartitionKey`，移除 JPA 映射，調整關聯）。
5. 轉換儲存庫（`JpaRepository` → `CosmosRepository`）。
6. **建立服務層**以進行關聯管理和範本相容性。
7. **重要**：更新所有測試檔案以使用 String ID 和 Cosmos 儲存庫。
8. 透過 `CommandLineRunner` 植入資料。
9. **重要**：測試執行時功能並修復範本相容性問題。

## 逐步進行

### 步驟 1 — 建置相依性

- **Maven** (`pom.xml`):
  - 移除相依性 `spring-boot-starter-data-jpa`
  - 移除資料庫特定相依性（H2、MySQL、PostgreSQL），除非在其他地方需要
  - 新增 `com.azure:azure-spring-data-cosmos:5.17.0`（或最新相容版本）
  - 新增 `com.azure:azure-identity:1.15.4`（DefaultAzureCredential 所需）
- **Gradle**：對 Gradle 語法採用相同的相依性更改
- 移除 testcontainers 和 JPA 特定的測試相依性

### 步驟 2 — 屬性和配置

- 建立 `src/main/resources/application-cosmos.properties`：
  ```properties
  azure.cosmos.uri=${COSMOS_URI:https://localhost:8081}
  azure.cosmos.database=${COSMOS_DATABASE:petclinic}
  azure.cosmos.populate-query-metrics=false
  azure.cosmos.enable-multiple-write-locations=false
  ```
- 更新 `src/main/resources/application.properties`：
  ```properties
  spring.profiles.active=cosmos
  ```

### 步驟 3 — 具有 Azure 身分識別的配置類別

- 建立 `src/main/java/<rootpkg>/config/CosmosConfiguration.java`：
  ```java
  @Configuration
  @EnableCosmosRepositories(basePackages = "<rootpkg>")
  public class CosmosConfiguration extends AbstractCosmosConfiguration {

    @Value("${azure.cosmos.uri}")
    private String uri;

    @Value("${azure.cosmos.database}")
    private String dbName;

    @Bean
    public CosmosClientBuilder getCosmosClientBuilder() {
      return new CosmosClientBuilder().endpoint(uri).credential(new DefaultAzureCredentialBuilder().build());
    }

    @Override
    protected String getDatabaseName() {
      return dbName;
    }

    @Bean
    public CosmosConfig cosmosConfig() {
      return CosmosConfig.builder().enableQueryMetrics(false).build();
    }
  }

  ```
- **重要**：為了生產環境安全性，請使用 `DefaultAzureCredentialBuilder().build()` 而不是基於金鑰的驗證

### 步驟 4 — 實體轉換

- 鎖定所有具有 JPA 註解的類別（`@Entity`、`@MappedSuperclass`、`@Embeddable`）
- **基本實體更改**：
  - 將 `id` 欄位類型從 `Integer` 更改為 `String`
  - 新增 `@Id` 和 `@GeneratedValue` 註解
  - 新增 `@PartitionKey` 欄位（通常是 `String partitionKey`）
  - 移除所有 `jakarta.persistence` 匯入
- **重要 - Cosmos DB 序列化需求**：
  - **移除所有需要持久化到 Cosmos DB 的欄位上的 `@JsonIgnore` 註解**
  - **身分驗證實體（User、Authority）必須完全可序列化** - 密碼、權限或其他持久化欄位上沒有 `@JsonIgnore`
  - **在需要控制 JSON 欄位名稱但仍需持久化資料時使用 `@JsonProperty` 而不是 `@JsonIgnore`**
  - **常見的身分驗證序列化錯誤**：`Cannot pass null or empty values to constructor` 通常表示 `@JsonIgnore` 阻擋了必要的欄位序列化
- **實體特定更改**：
  - 將 `@Entity` 替換為 `@Container(containerName = "<plural-entity-name>")`
  - 移除 `@Table`、`@Column`、`@JoinColumn` 等
  - 移除關聯註解（`@OneToMany`、`@ManyToOne`、`@ManyToMany`）
  - 對於關聯：
    - 為一對多嵌入集合（例如，Owner 中的 `List<Pet> pets`）
    - 為多對一使用參考 ID（例如，Pet 中的 `String ownerId`）
    - **對於複雜關聯**：存儲 ID 但為範本新增暫時性屬性
  - 新增建構子設定分割鍵：`setPartitionKey("entityType")`
- **重要 - 身分驗證實體模式**：
  - **對於具有 Spring Security 的 User 實體**：將權限存儲為 `Set<String>` 而不是 `Set<Authority>` 物件
  - **範例 User 實體轉換**：
    ```java
    @Container(containerName = "users")
    public class User {

      @Id
      private String id;

      @PartitionKey
      private String partitionKey = "user";

      private String login;
      private String password; // NO @JsonIgnore - must be serializable

      @JsonProperty("authorities") // Use @JsonProperty, not @JsonIgnore
      private Set<String> authorities = new HashSet<>(); // Store as strings

      // Add transient property for Spring Security compatibility if needed
      // @JsonIgnore - ONLY for transient properties not persisted to Cosmos
      private Set<Authority> authorityObjects = new HashSet<>();

      // Conversion methods between string authorities and Authority objects
      public void setAuthorityObjects(Set<Authority> authorities) {
        this.authorityObjects = authorities;
        this.authorities = authorities.stream().map(Authority::getName).collect(Collectors.toSet());
      }
    }

    ```
- **重要 - 關聯更改的範本相容性**：
  - **將關聯轉換為 ID 參考時，保持範本存取能力**
  - **例**：如果實體具有 `List<Specialty> specialties` → 轉換為：
    - 存儲：`List<String> specialtyIds`（持久化到 Cosmos）
    - 範本：`@JsonIgnore private List<Specialty> specialties = new ArrayList<>()` （暫時性）
    - 為兩個屬性新增取得器/設定器
  - **更新實體方法邏輯**：`getNrOfSpecialties()` 應該使用暫時性列表
- **重要 - Thymeleaf/JSP 應用程式的範本相容性**：
  - **識別範本屬性存取**：搜尋 `.html` 檔案中的 `${entity.relationshipProperty}`
  - **對於在範本中存取的每個關聯屬性**：
    - **存儲**：保持基於 ID 的存儲（例如，`List<String> specialtyIds`）
    - **範本存取**：新增暫時性屬性，帶有 `@JsonIgnore`（例如，`private List<Specialty> specialties = new ArrayList<>()`）
    - **例**：

      ```java
      // Stored in Cosmos (persisted)
      private List<String> specialtyIds = new ArrayList<>();

      // For template access (transient)
      @JsonIgnore
      private List<Specialty> specialties = new ArrayList<>();

      // Getters/setters for both properties
      public List<String> getSpecialtyIds() {
        return specialtyIds;
      }

      public List<Specialty> getSpecialties() {
        return specialties;
      }

      ```

    - **更新計數方法**：`getNrOfSpecialties()` 應該使用暫時性列表，而不是 ID 列表
- **重要 - 方法簽章衝突**：
  - **將 ID 類型從 Integer 轉換為 String 時，檢查方法簽章衝突**
  - **常見衝突**：`getPet(String name)` vs `getPet(String id)` - 都具有相同簽章
  - **解決方案**：重新命名方法以具有特異性：
    - `getPet(String id)` 用於基於 ID 的查詢
    - `getPetByName(String name)` 用於基於名稱的查詢
    - `getPetByName(String name, boolean ignoreNew)` 用於條件式名稱查詢
  - **更新控制器和測試中重新命名的方法的所有呼叫者**
- **實體的方法更新**：
  - 將 `addVisit(Integer petId, Visit visit)` 更新為 `addVisit(String petId, Visit visit)`
  - 確保所有 ID 比較邏輯使用 `.equals()` 而不是 `==`

### 步驟 5 — 儲存庫轉換

- 變更所有儲存庫介面：
  - 從：`extends JpaRepository<Entity, Integer>`
  - 至：`extends CosmosRepository<Entity, String>`
- **查詢方法更新**：
  - 從自訂查詢中移除分頁參數
  - 將 `Page<Entity> findByX(String param, Pageable pageable)` 變更為 `List<Entity> findByX(String param)`
  - 更新 `@Query` 註解以使用 Cosmos SQL 語法
  - **替換自訂方法名稱**：`findPetTypes()` → `findAllOrderByName()`
  - **更新控制器和格式化程式中已變更的方法名稱的所有參考**

### 步驟 6 — **建立服務層**以進行關聯管理和範本相容性

- **重要**：建立服務類別以橋接 Cosmos 文件存儲與現有的範本期望
- **目的**：處理關聯填充並保持範本相容性
- **每個具有關聯的實體的服務模式**：
  ```java
  @Service
  public class EntityService {

    private final EntityRepository entityRepository;
    private final RelatedRepository relatedRepository;

    public EntityService(EntityRepository entityRepository, RelatedRepository relatedRepository) {
      this.entityRepository = entityRepository;
      this.relatedRepository = relatedRepository;
    }

    public List<Entity> findAll() {
      List<Entity> entities = entityRepository.findAll();
      entities.forEach(this::populateRelationships);
      return entities;
    }

    public Optional<Entity> findById(String id) {
      Optional<Entity> entityOpt = entityRepository.findById(id);
      if (entityOpt.isPresent()) {
        Entity entity = entityOpt.get();
        populateRelationships(entity);
        return Optional.of(entity);
      }
      return Optional.empty();
    }

    private void populateRelationships(Entity entity) {
      if (entity.getRelatedIds() != null && !entity.getRelatedIds().isEmpty()) {
        List<Related> related = entity
          .getRelatedIds()
          .stream()
          .map(relatedRepository::findById)
          .filter(Optional::isPresent)
          .map(Optional::get)
          .collect(Collectors.toList());
        // Set transient property for template access
        entity.setRelated(related);
      }
    }
  }

  ```

### 步驟 6.5 — **Spring Security 整合**（身分驗證的重要事項）

- **UserDetailsService 整合模式**：
  ```java
  @Service
  @Transactional
  public class DomainUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;
    private final AuthorityRepository authorityRepository;

    @Override
    public UserDetails loadUserByUsername(String login) {
      log.debug("Authenticating user: {}", login);

      return userRepository
        .findOneByLogin(login)
        .map(user -> createSpringSecurityUser(login, user))
        .orElseThrow(() -> new UsernameNotFoundException("User " + login + " was not found"));
    }

    private org.springframework.security.core.userdetails.User createSpringSecurityUser(String lowercaseLogin, User user) {
      if (!user.isActivated()) {
        throw new UserNotActivatedException("User " + lowercaseLogin + " was not activated");
      }

      // Convert string authorities back to GrantedAuthority objects
      List<GrantedAuthority> grantedAuthorities = user
        .getAuthorities()
        .stream()
        .map(SimpleGrantedAuthority::new)
        .collect(Collectors.toList());

      return new org.springframework.security.core.userdetails.User(user.getLogin(), user.getPassword(), grantedAuthorities);
    }
  }

  ```
- **關鍵身分驗證需求**：
  - User 實體必須完全可序列化（密碼/權限上沒有 `@JsonIgnore`）
  - 將權限存儲為 `Set<String>` 以便與 Cosmos DB 相容
  - 在 UserDetailsService 中在字串權限和 `GrantedAuthority` 物件之間轉換
  - 新增全面的偵錯日誌以追蹤身分驗證流程
  - 適當處理已啟用/已停用的使用者狀態

#### **範本關聯填充模式**

返回用於範本呈現的實體的每個服務方法都必須填充暫時性屬性：

```java
private void populateRelationships(Entity entity) {
  // For each relationship used in templates
  if (entity.getRelatedIds() != null && !entity.getRelatedIds().isEmpty()) {
    List<Related> relatedObjects = entity
      .getRelatedIds()
      .stream()
      .map(relatedRepository::findById)
      .filter(Optional::isPresent)
      .map(Optional::get)
      .collect(Collectors.toList());
    entity.setRelated(relatedObjects); // Set transient property
  }
}

```

#### **控制器中的關鍵服務使用**

- **在控制器中用服務呼叫替換所有直接儲存庫呼叫**
- **從不直接從儲存庫返回實體**到沒有關聯填充的範本
- **更新控制器**以使用服務層而不是直接使用儲存庫
- **控制器模式更改**：

  ```java
  // 舊式：直接儲存庫使用
  @Autowired
  private EntityRepository entityRepository;

  // 新式：服務層使用
  @Autowired
  private EntityService entityService;
  // 更新方法呼叫
  // 舊式：entityRepository.findAll()
  // 新式：entityService.findAll()

  ```

### 步驟 7 — 資料植入

- 建立實現 `CommandLineRunner` 的 `@Component`：
  ```java
  @Component
  public class DataSeeder implements CommandLineRunner {

    @Override
    public void run(String... args) throws Exception {
      if (ownerRepository.count() > 0) {
        return; // 資料已存在
      }
      // 植入具有 String ID 的全面測試資料
      // 使用有意義的 ID 模式："owner-1"、"pet-1"、"pettype-1" 等
    }
  }

  ```
- **重要 - JDK 17+ 的 BigDecimal 反射問題**：
  - **如果使用 BigDecimal 欄位**，植入期間可能會遇到反射錯誤
  - **錯誤模式**：`Unable to make field private final java.math.BigInteger java.math.BigDecimal.intVal accessible`
  - **解決方案**：
    1. 對於貨幣值，使用 `Double` 或 `String` 而不是 `BigDecimal`
    2. 新增 JVM 引數：`--add-opens java.base/java.math=ALL-UNNAMED`
    3. 將 BigDecimal 操作包裝在 try-catch 中並妥善處理
  - **應用程式將成功啟動，即使植入失敗** - 檢查日誌以取得植入錯誤

### 步驟 8 — 測試檔案轉換（重要部分）

**此步驟通常被忽視，但對於成功轉換至關重要**

#### A. **編譯檢查策略**

- **每次進行重大變更後，執行 `mvn test-compile` 以及早發現問題**
- **在進行之前系統地修復編譯錯誤**
- **不要依賴 IDE - Maven 編譯會揭示所有問題**

#### B. **系統地搜尋和更新所有測試檔案**

**使用搜尋工具尋找並更新每個出現的地方：**

- 搜尋：`int.*TEST.*ID` → 替換為：`String.*TEST.*ID = "test-xyz-1"`
- 搜尋：`setId\(\d+\)` → 替換為：`setId("test-id-X")`
- 搜尋：`findById\(\d+\)` → 替換為：`findById("test-id-X")`
- 搜尋：`\.findPetTypes\(\)` → 替換為：`.findAllOrderByName()`
- 搜尋：`\.findByLastNameStartingWith\(.*,.*Pageable` → 移除分頁參數

#### C. 更新測試註解和匯入

- 使用 `@SpringBootTest` 或適當的切片測試替換 `@DataJpaTest`
- 移除 `@AutoConfigureTestDatabase` 註解
- 從測試中移除 `@Transactional`（除非進行單一分割區操作）
- 移除 `org.springframework.orm` 套件的匯入

#### D. 在所有測試檔案中修復實體 ID 使用

**必須更新的關鍵檔案（搜尋整個測試目錄）：**

- `*ControllerTests.java` - 路徑變數、實體建立、模擬設定
- `*ServiceTests.java` - 儲存庫互動、實體 ID
- `EntityUtils.java` - ID 處理的公用程式方法
- `*FormatterTests.java` - 儲存庫方法呼叫
- `*ValidatorTests.java` - 使用 String ID 建立實體
- 整合測試類別 - 測試資料設定

#### E. **修復受儲存庫變更影響的控制器和服務類別**

- **更新呼叫具有已變更簽章的儲存庫方法的控制器**
- **更新使用儲存庫方法的格式化程式/轉換器**
- **要檢查的常見檔案**：
  - `PetTypeFormatter.java` - 經常呼叫 `findPetTypes()` 方法
  - `*Controller.java` - 可能需要移除分頁邏輯
  - 使用儲存庫方法的服務類別

#### F. 在測試中更新儲存庫模擬

- 從儲存庫模擬中移除分頁：
  - `given(repository.findByX(param, pageable)).willReturn(pageResult)`
  - → `given(repository.findByX(param)).willReturn(listResult)`
- 在模擬中更新方法名稱：
  - `given(petTypeRepository.findPetTypes()).willReturn(types)`
  - → `given(petTypeRepository.findAllOrderByName()).willReturn(types)`

#### G. 修復測試使用的公用程式類別

- 更新 `EntityUtils.java` 或類似的程式碼：
  - 移除 JPA 特定的例外匯入（`ObjectRetrievalFailureException`）
  - 將方法簽章從 `int id` 變更為 `String id`
  - 更新 ID 比較邏輯：`entity.getId() == entityId` → `entity.getId().equals(entityId)`
  - 用標準例外取代 JPA 例外（`IllegalArgumentException`）

#### H. 更新 String ID 的判斷提示

- 變更 ID 判斷提示：
  - `assertThat(entity.getId()).isNotZero()` → `assertThat(entity.getId()).isNotEmpty()`
  - `assertThat(entity.getId()).isEqualTo(1)` → `assertThat(entity.getId()).isEqualTo("test-id-1")`
  - JSON 路徑判斷提示：`jsonPath("$.id").value(1)` → `jsonPath("$.id").value("test-id-1")`

### 步驟 9 — **執行時測試和範本相容性**

#### **重要**：編譯成功後測試執行中的應用程式

- **啟動應用程式**：`mvn spring-boot:run`
- **瀏覽網路介面中的所有頁面**以識別執行時錯誤
- **轉換後的常見執行時問題**：
  - 範本嘗試存取不再存在的屬性（例如，`vet.specialties`）
  - 服務層未填充暫時性關聯屬性
  - 控制器不使用服務層進行關聯載入

#### **範本相容性修復**：

- **如果範本存取關聯屬性**（例如，`entity.relatedObjects`）：
  - 確保暫時性屬性存在於實體上，具有適當的取得器/設定器
  - 驗證服務層填充這些暫時性屬性
  - 更新 `getNrOfXXX()` 方法以使用暫時性列表而不是 ID 列表
- **檢查日誌中的 SpEL（Spring Expression Language）錯誤**：
  - `Property or field 'xxx' cannot be found` → 新增遺失的暫時性屬性
  - `EL1008E` 錯誤 → 服務層未填充關聯

#### **服務層驗證**：

- **確保所有控制器使用服務層**而不是直接儲存庫存取
- **驗證服務方法在返回實體前填充關聯**
- **透過網路介面測試所有 CRUD 操作**

### 步驟 9.5 — **範本執行時驗證**（重要）

#### **系統的範本測試流程**

編譯成功和應用程式啟動後：

1. **系統地瀏覽應用程式中的每一頁**
2. **測試顯示實體資料的每個範本**：
   - 列表頁面（例如，`/vets`、`/owners`）
   - 詳細頁面（例如，`/owners/{id}`、`/vets/{id}`）
   - 表單和編輯頁面
3. **尋找特定的範本錯誤**：
   - `Property or field 'relationshipName' cannot be found on object of type 'EntityName'`
   - `EL1008E` Spring Expression Language 錯誤
   - 關聯應該出現的地方沒有資料或資料遺失

#### **範本錯誤解決檢查清單**

遇到範本錯誤時：

- [ ] **從錯誤訊息識別遺失的屬性**
- [ ] **檢查屬性是否在實體中以暫時性欄位形式存在**
- [ ] **驗證服務層在返回實體前填充屬性**
- [ ] **確保控制器使用服務層**，而不是直接儲存庫存取
- [ ] **修復後再次測試特定頁面**

#### **常見的範本錯誤模式**

- `Property or field 'specialties' cannot be found` → 在 Vet 實體中新增 `@JsonIgnore private List<Specialty> specialties`
- `Property or field 'pets' cannot be found` → 在 Owner 實體中新增 `@JsonIgnore private List<Pet> pets`
- 顯示的關聯資料為空 → 服務未填充暫時性屬性

### 步驟 10 — **系統的錯誤解決流程**

#### 編譯失敗時：

1. **首先執行 `mvn compile`** - 在測試前修復主要源程式碼問題
2. **執行 `mvn test-compile`** - 系統地修復每個測試編譯錯誤
3. **專注於最常見的錯誤模式**：
   - `int cannot be converted to String` → 變更測試常數和實體設定器
   - `method X cannot be applied to given types` → 移除分頁參數
   - `cannot find symbol: method Y()` → 更新為新的儲存庫方法名稱
   - 方法簽章衝突 → 重新命名衝突的方法

#### 執行時失敗時：

1. **檢查應用程式日誌**以取得特定的錯誤訊息
2. **尋找範本/SpEL 錯誤**：
   - `Property or field 'xxx' cannot be found` → 在實體中新增暫時性屬性
   - 遺失的關聯資料 → 服務層未填充關聯
3. **驗證控制器中的服務層使用**
4. **測試瀏覽所有應用程式頁面**

#### 常見的錯誤模式和解決方案：

- **`method findByLastNameStartingWith cannot be applied`** → 移除 `Pageable` 參數
- **`cannot find symbol: method findPetTypes()`** → 變更為 `findAllOrderByName()`
- **`incompatible types: int cannot be converted to String`** → 更新測試 ID 常數
- **`method getPet(String) is already defined`** → 重新命名一個方法（例如，`getPetByName`）
- **`cannot find symbol: method isNotZero()`** → 對於 String ID，變更為 `isNotEmpty()`
- **`Property or field 'specialties' cannot be found`** → 新增暫時性屬性並在服務中填充
- **`ClassCastException: reactor.core.publisher.BlockingIterable cannot be cast to java.util.List`** → 修復儲存庫 `findAllWithEagerRelationships()` 方法以使用 StreamSupport
- **`Unable to make field...BigDecimal.intVal accessible`** → 在整個應用程式中用 Double 取代 BigDecimal
- **健康檢查資料庫失敗** → 從健康檢查就緒配置中移除 'db'

#### **範本特定的執行時錯誤**

- **`Property or field 'XXX' cannot be found on object of type 'YYY'`**：

  - 根本原因：範本存取已轉換為 ID 存儲的關聯屬性
  - 解決方案：在實體中新增暫時性屬性 + 在服務層中填充
  - 預防：在轉換關聯前始終檢查範本使用

- **`EL1008E` Spring Expression Language 錯誤**：

  - 根本原因：服務層未填充暫時性屬性
  - 解決方案：驗證 `populateRelationships()` 方法被呼叫且運作
  - 預防：在服務層實作後測試所有範本導航

- **範本中的空/null 關聯資料**：
  - 根本原因：控制器繞過服務層或服務未填充關聯
  - 解決方案：確保所有控制器方法使用服務層進行實體檢索
  - 預防：決不直接向範本返回儲存庫結果

### 步驟 11 — 驗證檢查清單

轉換後，驗證：

- [ ] **主要應用程式編譯**：`mvn compile` 成功
- [ ] **所有測試檔案編譯**：`mvn test-compile` 成功
- [ ] **無編譯錯誤**：解決每個編譯錯誤
- [ ] **應用程式成功啟動**：`mvn spring-boot:run` 無錯誤
- [ ] **所有網頁載入**：瀏覽所有應用程式頁面，無執行時錯誤
- [ ] **服務層填充關聯**：暫時性屬性正確設定
- [ ] **所有範本頁面無錯誤呈現**：瀏覽整個應用程式
- [ ] **關聯資料正確顯示**：列表、計數和相關物件正確顯示
- [ ] **日誌中沒有 SpEL 範本錯誤**：在導航期間檢查應用程式日誌
- [ ] **暫時性屬性帶有 @JsonIgnore 註解**：防止 JSON 序列化問題
- [ ] **服務層一致使用**：控制器中沒有直接儲存庫存取用於範本呈現
- [ ] 沒有剩餘的 `jakarta.persistence` 匯入
- [ ] 所有實體 ID 一致為 `String` 類型
- [ ] 所有儲存庫介面擴充 `CosmosRepository<Entity, String>`
- [ ] 配置使用 `DefaultAzureCredential` 進行驗證
- [ ] 資料植入元件存在且運作
- [ ] 測試檔案一致使用 String ID
- [ ] 儲存庫模擬針對 Cosmos 方法進行更新
- [ ] **實體類別中沒有方法簽章衝突**
- [ ] **呼叫者中更新所有重新命名的方法**（控制器、測試、格式化程式）

### 要避免的常見陷阱

1. **不經常檢查編譯** - 在每次進行重大變更後執行 `mvn test-compile`
2. **方法簽章衝突** - 轉換 ID 類型時的方法超載問題
3. **忘記更新方法呼叫者** - 重新命名方法時，更新所有呼叫者
4. **遺失儲存庫方法重新命名** - 自訂儲存庫方法必須在每個呼叫位置進行更新
5. **使用基於金鑰的驗證** - 改用 `DefaultAzureCredential`
6. **混合 Integer 和 String ID** - 在所有地方（特別是測試中）與 String ID 一致
7. **不更新控制器分頁邏輯** - 儲存庫變更時從控制器移除分頁
8. **留下 JPA 特定的測試註解** - 用 Cosmos 相容的替代品取代
9. **不完整的測試檔案更新** - 搜尋整個測試目錄，不僅是明顯的檔案
10. **跳過執行時測試** - 始終測試執行中的應用程式，而不僅是編譯
11. **遺失服務層** - 不要直接從控制器存取儲存庫
12. **忘記暫時性屬性** - 範本可能需要存取關聯資料
13. **不測試範本導航** - 編譯成功並不表示範本運作
14. **遺失範本的暫時性屬性** - 範本需要物件存取，而不僅是 ID
15. **服務層繞過** - 控制器必須使用服務，決不直接儲存庫存取
16. **不完整的關聯填充** - 服務方法必須填充範本使用的所有暫時性屬性
17. **忘記在暫時性屬性上使用 @JsonIgnore** - 防止序列化問題
18. **在持久化欄位上使用 @JsonIgnore** - **重要**：決不在需要存儲在 Cosmos DB 中的欄位上使用 `@JsonIgnore`
19. **身分驗證序列化錯誤** - User/Authority 實體必須完全可序列化，無 `@JsonIgnore` 阻擋必要的欄位
20. **BigDecimal 反射問題** - 對於 JDK 17+ 相容性，使用替代資料類型或 JVM 引數
21. **儲存庫反應式類型轉換** - 不要直接將 `findAll()` 轉換為 `List`，使用 `StreamSupport.stream().collect(Collectors.toList())`
22. **健康檢查資料庫參考** - 移除 JPA 後從 Spring Boot 健康檢查中移除資料庫相依性
23. **集合類型不匹配** - 更新服務方法以一致地處理 String 與物件集合

### 系統地調試編譯問題

如果轉換後編譯失敗：

1. **從主要編譯開始**：`mvn compile` - 首先修復實體和控制器問題
2. **然後進行測試編譯**：`mvn test-compile` - 系統地修復每個錯誤
3. **檢查整個程式碼庫中剩餘的 `jakarta.persistence` 匯入**
4. **驗證所有測試常數使用 String ID** - 搜尋 `int.*TEST.*ID`
5. **確保儲存庫方法簽章符合**新 Cosmos 介面
6. **檢查實體關聯和測試中的混合 Integer/String ID 使用**
7. **驗證所有模擬使用正確的方法名稱**（`findAllOrderByName()` 而不是 `findPetTypes()`）
8. **尋找方法簽章衝突** - 透過重新命名衝突的方法解決
9. **驗證判斷提示方法適用於 String ID**（`isNotEmpty()` 而不是 `isNotZero()`）

### 系統地調試執行時問題

如果編譯成功後執行時失敗：

1. **檢查應用程式啟動日誌**以尋找初始化錯誤
2. **瀏覽所有頁面**以識別範本/控制器問題
3. **在日誌中尋找 SpEL 範本錯誤**：
   - `Property or field 'xxx' cannot be found` → 遺失暫時性屬性
   - `EL1008E` → 服務層未填充關聯
4. **驗證服務層正在被使用**而不是直接儲存庫存取
5. **檢查暫時性屬性在服務方法中被填充**
6. **透過網路介面測試所有 CRUD 操作**
7. **驗證資料植入正確運作**且關聯被保持
8. **身分驗證特定的偵錯**：
   - `Cannot pass null or empty values to constructor` → 檢查必要欄位上的 `@JsonIgnore`
   - `BadCredentialsException` → 驗證 User 實體序列化和密碼欄位可存取性
   - 檢查日誌中的 "DomainUserDetailsService" 偵錯輸出以追蹤身分驗證流程

### **成功的專業秘訣**

- **頻繁地編譯** - 不要讓錯誤累積
- **使用全域搜尋和替換** - 找出所有要更新的模式出現位置
- **要系統化** - 在移至下一個之前修復所有檔案中的一種類型的錯誤
- **謹慎測試方法重新命名** - 確保所有呼叫者都被更新
- **使用有意義的 String ID** - "owner-1"、"pet-1" 而不是隨機字串
- **檢查控制器類別** - 它們經常呼叫簽章改變的儲存庫方法
- **始終測試執行時** - 編譯成功並不保證範本正常運作
- **服務層至關重要** - 文件存儲和範本期望之間的橋樑

### **身分驗證故障排除指南**（重要）

#### **常見的身分驗證序列化錯誤**：

1. **`Cannot pass null or empty values to constructor`**：

   - **根本原因**：`@JsonIgnore` 防止必要的欄位序列化到 Cosmos DB
   - **解決方案**：從所有持久化欄位（密碼、權限等）移除 `@JsonIgnore`
   - **驗證**：檢查 User 實體的存儲欄位上沒有 `@JsonIgnore`

2. **登入期間出現 `BadCredentialsException`**：

   - **根本原因**：密碼欄位在驗證期間無法存取
   - **解決方案**：確保密碼欄位可序列化且在 UserDetailsService 中可存取
   - **驗證**：在 `loadUserByUsername` 方法中新增偵錯日誌

3. **權限載入不正確**：

   - **根本原因**：Authority 物件以複雜實體而不是字串形式存儲
   - **解決方案**：將權限存儲為 `Set<String>` 並在 UserDetailsService 中轉換為 `GrantedAuthority`
   - **模式**：

     ```java
     // 在 User 實體中 - 存儲在 Cosmos 中
     @JsonProperty("authorities")
     private Set<String> authorities = new HashSet<>();

     // 在 UserDetailsService 中 - 為 Spring Security 轉換
     List<GrantedAuthority> grantedAuthorities = user
       .getAuthorities()
       .stream()
       .map(SimpleGrantedAuthority::new)
       .collect(Collectors.toList());

     ```

4. **驗證期間找不到 User 實體**：
   - **根本原因**：儲存庫查詢方法不適用於 String ID
   - **解決方案**：更新儲存庫 `findOneByLogin` 方法以適用於 Cosmos DB
   - **驗證**：獨立測試儲存庫方法

#### **身分驗證偵錯檢查清單**：

- [ ] User 實體完全可序列化（持久化欄位上沒有 `@JsonIgnore`）
- [ ] 密碼欄位可存取且不為 null
- [ ] 權限存儲為 `Set<String>`
- [ ] UserDetailsService 將字串權限轉換為 `GrantedAuthority`
- [ ] 儲存庫方法適用於 String ID
- [ ] 在驗證服務中啟用偵錯日誌
- [ ] 適當檢查 User 啟用狀態
- [ ] 使用已知的認證資訊測試登入（admin/admin）

### **常見的執行時問題和解決方案**

#### **問題 1：儲存庫反應式類型轉換錯誤**

**錯誤**：`ClassCastException: reactor.core.publisher.BlockingIterable cannot be cast to java.util.List`

**根本原因**：Cosmos 儲存庫傳回反應式類型（`Iterable`），但舊版 JPA 程式碼期望 `List`

**解決方案**：在儲存庫方法中正確轉換反應式類型：

```java
// 錯誤 - 直接轉換失敗
default List<Entity> customFindMethod() {
    return (List<Entity>) this.findAll(); // ClassCastException!
}

// 正確 - 將 Iterable 轉換為 List
default List<Entity> customFindMethod() {
    return StreamSupport.stream(this.findAll().spliterator(), false)
            .collect(Collectors.toList());
}
```

**要檢查的檔案**：

- 所有具有自訂預設方法的儲存庫介面
- 從 Cosmos 儲存庫呼叫傳回 `List<Entity>` 的任何方法
- 匯入 `java.util.stream.StreamSupport` 和 `java.util.stream.Collectors`

#### **問題 2：Java 17+ 中的 BigDecimal 反射問題**

**錯誤**：`Unable to make field private final java.math.BigInteger java.math.BigDecimal.intVal accessible`

**根本原因**：Java 17+ 模組系統在序列化期間限制對 BigDecimal 內部欄位的反射存取

**解決方案**：

1. **對於簡單情況，用 Double 取代**：

   ```java
   // 之前：BigDecimal 欄位
   private BigDecimal amount;

   // 之後：Double 欄位（如果精度要求允許）
   private Double amount;

   ```

2. **對於高精度要求，使用 String**：

   ```java
   // 以字串形式存儲，視需要轉換
   private String amount; // 存儲 "1500.00"

   public BigDecimal getAmountAsBigDecimal() {
     return new BigDecimal(amount);
   }

   ```

3. **新增 JVM 引數**（如果必須保留 BigDecimal）：
   ```
   --add-opens java.base/java.math=ALL-UNNAMED
   ```

#### **問題 3：健康檢查資料庫相依性**

**錯誤**：應用程式無法通過尋找已移除資料庫元件的健康檢查

**根本原因**：Spring Boot 健康檢查仍引用移除後的 JPA/資料庫相依性

**解決方案**：更新健康檢查配置：

```yaml
# 在 application.yml 中 - 從健康檢查中移除資料庫
management:
  health:
    readiness:
      include: 'ping,diskSpace' # 如果存在，移除 'db'
```

**要檢查的檔案**：

- 所有 `application*.yml` 配置檔案
- 移除任何資料庫特定的健康指標
- 檢查執行器端點配置

#### **問題 4：服務中的集合類型不匹配**

**錯誤**：將實體關聯轉換為基於字串的存儲時的類型不匹配錯誤

**根本原因**：服務方法在實體轉換後期望不同的集合類型

**解決方案**：更新服務方法以處理新的實體結構：

```java
// 之前：實體關聯
public Set<RelatedEntity> getRelatedEntities() {
    return entity.getRelatedEntities(); // 直接實體參考
}

// 之後：基於字串的關聯和轉換
public Set<RelatedEntity> getRelatedEntities() {
    return entity.getRelatedEntityIds()
        .stream()
        .map(relatedRepository::findById)
        .filter(Optional::isPresent)
        .map(Optional::get)
        .collect(Collectors.toSet());
}

### **增強的錯誤解決流程**

#### **常見的錯誤模式和解決方案**：

1. **反應式類型轉換錯誤**：
   - **模式**：`cannot be cast to java.util.List`
   - **修復**：使用 `StreamSupport.stream().collect(Collectors.toList())`
   - **檔案**：具有自訂預設方法的儲存庫介面

2. **BigDecimal 序列化錯誤**：
   - **模式**：`Unable to make field...BigDecimal.intVal accessible`
   - **修復**：用 Double、String 取代，或新增 JVM 模組開啟
   - **檔案**：實體類別、DTO、資料初始化類別

3. **健康檢查資料庫錯誤**：
   - **模式**：健康檢查尋找資料庫時失敗
   - **修復**：從健康檢查配置中移除資料庫參考
   - **檔案**：application.yml 配置檔案

4. **集合類型轉換錯誤**：
   - **模式**：實體關聯處理中的類型不匹配
   - **修復**：更新服務方法以處理基於字串的實體參考
   - **檔案**：服務類別、DTO、實體關聯方法

#### **增強的驗證檢查清單**：
- [ ] **儲存庫反應式轉換已處理**：集合傳回時沒有 ClassCastException
- [ ] **BigDecimal 相容性已解決**：Java 17+ 序列化運作
- [ ] **健康檢查已更新**：健康配置中沒有資料庫相依性
- [ ] **服務層集合處理**：基於字串的實體參考正常運作
- [ ] **資料植入完成**：日誌中出現 "Data seeding completed" 訊息
- [ ] **應用程式完全啟動**：前端和後端都可存取
- [ ] **身分驗證運作**：可以登入而不會出現序列化錯誤
- [ ] **CRUD 操作正常運作**：所有實體操作透過 UI 運作

## **快速參考：常見的遷移後修復**

### **要檢查的頂級執行時問題**

1. **儲存庫集合轉換**：
   ```java
   // 修復任何傳回集合的儲存庫方法：
   default List<Entity> customFindMethod() {
       return StreamSupport.stream(this.findAll().spliterator(), false)
               .collect(Collectors.toList());
   }

2. **BigDecimal 相容性（Java 17+）**：

   ```java
   // 用替代品取代 BigDecimal 欄位：
   private Double amount; // 或用於高精度的 String

   ```

3. **健康檢查配置**：
   ```yaml
   # 從健康檢查中移除資料庫相依性：
   management:
     health:
       readiness:
         include: 'ping,diskSpace'
   ```

### **身分驗證轉換模式**

- **從需要 Cosmos DB 持久化的欄位移除 `@JsonIgnore`**
- **將複雜物件存儲為簡單類型**（例如，權限為 `Set<String>`）
- **在服務/儲存庫層中在簡單和複雜類型之間轉換**

### **範本/UI 相容性模式**

- **新增暫時性屬性**，帶有 `@JsonIgnore` 以便 UI 存取相關資料
- **使用服務層**在呈現前填充暫時性關聯
- **決不直接向範本傳回儲存庫結果**，無需關聯填充
