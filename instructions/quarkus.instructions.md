---
applyTo: '*'
description: 'Quarkus 開發標準和指引'
---

- 使用 Java 17 或更高版本構建高品質 Quarkus 應用程式的指引。

## 專案背景

- 最新 Quarkus 版本：3.x
- Java 版本：17 或更高版本
- 使用 Maven 或 Gradle 進行構建管理。
- 專注於乾淨架構、可維護性和效能。

## 開發標準

  - 為每個類別、方法和複雜邏輯撰寫清晰簡潔的註解。
  - 為公開 API 和方法使用 Javadoc 以確保使用者清楚理解。
  - 在整個專案中保持一致的編碼風格，遵循 Java 慣例。
  - 遵循 Quarkus 編碼標準和最佳實踐，以獲得最佳效能和可維護性。
  - 遵循 Jakarta EE 和 MicroProfile 慣例，確保套件組織的清晰性。
  - 在適當的地方使用 Java 17 或更高版本的功能，例如 records 和 sealed classes。


## 命名慣例
  - 類別名稱使用 PascalCase（例如 `ProductService`、`ProductResource`）。
  - 方法和變數名稱使用 camelCase（例如 `findProductById`、`isProductAvailable`）。
  - 常數使用 ALL_CAPS（例如 `DEFAULT_PAGE_SIZE`）。

##  Quarkus
  - 利用 Quarkus Dev Mode 加快開發週期。
  - 使用 Quarkus 擴充功能和最佳實踐實作構建時優化。
  - 使用 GraalVM 配置原生構建以獲得最佳效能（例如使用 quarkus-maven-plugin）。
  - 使用 quarkus 日誌記錄功能（JBoss、SL4J 或 JUL）進行一致的日誌記錄實踐。

### Quarkus 特定模式
- 使用 `@ApplicationScoped` 而非 `@Singleton` 來建立單例 Bean
- 使用 `@Inject` 進行依賴注入
- 優先使用 Panache 儲存庫而非傳統 JPA 儲存庫
- 在修改資料的服務方法上使用 `@Transactional`
- 使用具有描述性 REST 端點路徑的 `@Path`
- 為 REST 資源使用 `@Consumes(MediaType.APPLICATION_JSON)` 和 `@Produces(MediaType.APPLICATION_JSON)`

### REST 資源
- 始終使用 JAX-RS 註解（`@Path`、`@GET`、`@POST` 等）
- 回傳適當的 HTTP 狀態碼（200、201、400、404、500）
- 對於複雜回應使用 `Response` 類別
- 使用 try-catch 區塊包含適當的錯誤處理
- 使用 Bean Validation 註解驗證輸入參數
- 為公開端點實作速率限制

### 資料存取
- 優先使用 Panache 實體（繼承 `PanacheEntity`）而非傳統 JPA
- 使用 Panache 儲存庫（`PanacheRepository<T>`）進行複雜查詢
- 對於資料修改始終使用 `@Transactional`
- 對複雜資料庫操作使用命名查詢
- 為列表端點實作適當的分頁


### 配置
- 對簡單配置使用 `application.properties` 或 `application.yaml`
- 使用 `@ConfigProperty` 進行型別安全的配置類別
- 對敏感資料優先使用環境變數
- 為不同環境使用設定檔（dev、test、prod）


### 測試
- 對整合測試使用 `@QuarkusTest`
- 對單元測試使用 JUnit 5
- 對原生構建測試使用 `@QuarkusIntegrationTest`
- 使用 `@QuarkusTestResource` 模擬外部依賴項
- 使用 RestAssured 進行 REST 端點測試（`@QuarkusTestResource`）
- 對修改資料庫的測試使用 `@Transactional`
- 對資料庫整合測試使用 test-containers

### 不要使用這些模式：
- 不要在測試中使用欄位注入（使用建構函式注入）
- 不要硬編碼配置值
- 不要忽略例外


## 開發工作流程

### 建立新功能時：
1. 建立具有適當驗證的實體
2. 建立具有自訂查詢的儲存庫
3. 建立具有業務邏輯的服務
4. 建立具有適當端點的 REST 資源
5. 撰寫全面的測試
6. 新增適當的錯誤處理
7. 更新文檔

## 安全考量

### 實作安全性時：
- 使用 Quarkus Security 擴充功能（例如 `quarkus-smallrye-jwt`、`quarkus-oidc`）。
- 使用 MicroProfile JWT 或 OIDC 實作基於角色的存取控制（RBAC）。
- 驗證所有輸入參數
