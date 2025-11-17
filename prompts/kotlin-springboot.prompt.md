---
mode: 'agent'
tools: ['changes', 'search/codebase', 'edit/editFiles', 'problems', 'search']
description: 'Get best practices for developing applications with Spring Boot and Kotlin.'
---

# Spring Boot 與 Kotlin 最佳實踐

您的目標是幫我撰寫高品質、符合 Kotlin 慣例的 Spring Boot 應用程式。

## 專案設定與結構

- **建置工具：** 使用 Maven (`pom.xml`) 或 Gradle (`build.gradle`)，搭配 Kotlin 外掛程式 (`kotlin-maven-plugin` 或 `org.jetbrains.kotlin.jvm`)。
- **Kotlin 外掛程式：** 若使用 JPA，啟用 `kotlin-jpa` 外掛程式以自動開放實體類別，無需繁冗的程式碼。
- **啟動器：** 如常使用 Spring Boot 啟動器 (例如 `spring-boot-starter-web`、`spring-boot-starter-data-jpa`)。
- **套件結構：** 按功能/域 (例如 `com.example.app.order`、`com.example.app.user`) 組織程式碼，而非按層次。

## 依賴注入與元件

- **主要建構子：** 務必為必要的依賴注入使用主要建構子。這是 Kotlin 中最符合慣例且最簡潔的方式。
- **不可變性：** 在主要建構子中將依賴宣告為 `private val`。在各處優先使用 `val` 而非 `var`，以促進不可變性。
- **元件原型：** 如同在 Java 中一樣使用 `@Service`、`@Repository` 和 `@RestController` 註解。

## 設定

- **外部化設定：** 使用 `application.yml` 以獲得良好的可讀性和階層式結構。
- **型別安全的屬性：** 使用 `@ConfigurationProperties` 搭配 `data class` 建立不可變的、型別安全的設定物件。
- **設定檔：** 使用 Spring 設定檔 (`application-dev.yml`、`application-prod.yml`) 管理環境特定的設定。
- **祕密管理：** 絕對不要將祕密硬編碼。使用環境變數或專用的祕密管理工具，如 HashiCorp Vault 或 AWS Secrets Manager。

## Web 層 (控制器)

- **RESTful API：** 設計清晰且一致的 RESTful 端點。
- **DTO 的資料類別：** 為所有 DTO 使用 Kotlin `data class`。這可以免費提供 `equals()`、`hashCode()`、`toString()` 和 `copy()`，並促進不可變性。
- **驗證：** 使用 Java Bean Validation (JSR 380) 搭配您的 DTO 資料類別上的註解 (`@Valid`、`@NotNull`、`@Size`)。
- **錯誤處理：** 使用 `@ControllerAdvice` 和 `@ExceptionHandler` 實作全域例外處理器以提供一致的錯誤回應。

## 服務層

- **業務邏輯：** 在 `@Service` 類別內封裝業務邏輯。
- **無狀態性：** 服務應該是無狀態的。
- **事務管理：** 在服務方法上使用 `@Transactional`。在 Kotlin 中，可以應用於類別或函式層級。

## 資料層 (儲存庫)

- **JPA 實體：** 將實體定義為類別。務必記得它們必須是 `open` 的。強烈建議使用 `kotlin-jpa` 編譯器外掛程式以自動處理此問題。
- **空值安全性：** 利用 Kotlin 的空值安全性 (`?`) 在型別層級清楚定義哪些實體欄位是選擇性的或必要的。
- **Spring Data JPA：** 透過擴展 `JpaRepository` 或 `CrudRepository` 使用 Spring Data JPA 儲存庫。
- **協程：** 對於反應式應用程式，在資料層利用 Spring Boot 對 Kotlin 協程的支援。

## 日誌記錄

- **伴隨物件日誌程式：** 宣告日誌程式的習慣方式是在伴隨物件中。
  ```kotlin
  companion object {
      private val logger = LoggerFactory.getLogger(MyClass::class.java)
  }
  ```
- **參數化日誌記錄：** 使用參數化訊息 (`logger.info("Processing user {}...", userId)`) 以提高效能和清晰度。

## 測試

- **JUnit 5：** JUnit 5 是預設項目，與 Kotlin 無縫運作。
- **習慣式測試程式庫：** 若要更流暢且習慣式的測試，請考慮使用 **Kotest** 進行判斷以及 **MockK** 進行模擬。它們是為 Kotlin 設計的，提供更具表達力的語法。
- **測試切片：** 使用測試切片註解 (如 `@WebMvcTest` 或 `@DataJpaTest`) 測試應用程式的特定部分。
- **測試容器：** 使用測試容器進行可靠的整合測試，搭配實際的資料庫、訊息代理人等。

## 協程與非同步程式設計

- **`suspend` 函式：** 對於非阻塞的非同步程式碼，在控制器和服務中使用 `suspend` 函式。Spring Boot 對協程提供了卓越的支援。
- **結構化並行性：** 使用 `coroutineScope` 或 `supervisorScope` 管理協程的生命週期。
