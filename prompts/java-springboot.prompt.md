---
mode: 'agent'
tools: ['changes', 'search/codebase', 'edit/editFiles', 'problems', 'search']
description: '取得使用 Spring Boot 開發應用程式的最佳實踐。'
---

# Spring Boot 最佳實踐

您的目標是透過遵循既定的最佳實踐,協助我撰寫高品質的 Spring Boot 應用程式。

## 專案設定與結構

- **建置工具：** 使用 Maven (`pom.xml`) 或 Gradle (`build.gradle`) 進行相依性管理。
- **啟動器：** 使用 Spring Boot 啟動器（例如 `spring-boot-starter-web`、`spring-boot-starter-data-jpa`）簡化相依性管理。
- **套件結構：** 按功能/領域組織程式碼（例如 `com.example.app.order`、`com.example.app.user`），而不是按層（例如 `com.example.app.controller`、`com.example.app.service`）。

## 相依性注入與元件

- **建構函式注入：** 始終對必需的相依性使用基於建構函式的注入。這使元件更易於測試,相依性更明確。
- **不可變性：** 將相依性欄位宣告為 `private final`。
- **元件刻板印象：** 適當使用 `@Component`、`@Service`、`@Repository` 和 `@Controller`/`@RestController` 註解來定義 bean。

## 設定

- **外部化設定：** 使用 `application.yml`（或 `application.properties`）進行設定。YAML 因其可讀性和階層結構而通常更受青睞。
- **類型安全屬性：** 使用 `@ConfigurationProperties` 將設定繫結到強類型 Java 物件。
- **設定檔：** 使用 Spring 設定檔（`application-dev.yml`、`application-prod.yml`）管理環境特定的設定。
- **機密管理：** 不要硬編碼機密。使用環境變數,或專用的機密管理工具,如 HashiCorp Vault 或 AWS Secrets Manager。

## Web 層（控制器）

- **RESTful API：** 設計清晰且一致的 RESTful 端點。
- **DTO（資料傳輸物件）：** 使用 DTO 在 API 層中公開和使用資料。不要直接向客戶端公開 JPA 實體。
- **驗證：** 在 DTO 上使用 Java Bean Validation (JSR 380) 註解（`@Valid`、`@NotNull`、`@Size`）來驗證請求負載。
- **錯誤處理：** 使用 `@ControllerAdvice` 和 `@ExceptionHandler` 實作全域例外處理器,以提供一致的錯誤回應。

## 服務層

- **業務邏輯：** 將所有業務邏輯封裝在 `@Service` 類別中。
- **無狀態：** 服務應該是無狀態的。
- **交易管理：** 在服務方法上使用 `@Transactional` 以宣告方式管理資料庫交易。在必要的最細粒度級別應用它。

## 資料層（儲存庫）

- **Spring Data JPA：** 透過擴充 `JpaRepository` 或 `CrudRepository` 使用 Spring Data JPA 儲存庫進行標準資料庫操作。
- **自訂查詢：** 對於複雜查詢,使用 `@Query` 或 JPA Criteria API。
- **投影：** 使用 DTO 投影僅從資料庫擷取必要的資料。

## 日誌記錄

- **SLF4J：** 使用 SLF4J API 進行日誌記錄。
- **日誌記錄器宣告：** `private static final Logger logger = LoggerFactory.getLogger(MyClass.class);`
- **參數化日誌記錄：** 使用參數化訊息（`logger.info("Processing user {}...", userId);`）而不是字串串接來改善效能。

## 測試

- **單元測試：** 使用 JUnit 5 和 Mockito 等模擬框架為服務和元件撰寫單元測試。
- **整合測試：** 使用 `@SpringBootTest` 進行載入 Spring 應用程式情境的整合測試。
- **測試切片：** 使用測試切片註解,如 `@WebMvcTest`（用於控制器）或 `@DataJpaTest`（用於儲存庫）,以隔離測試應用程式的特定部分。
- **Testcontainers：** 考慮使用 Testcontainers 進行可靠的整合測試,包括真實的資料庫、訊息代理等。

## 安全性

- **Spring Security：** 使用 Spring Security 進行身份驗證和授權。
- **密碼編碼：** 始終使用強大的雜湊演算法（如 BCrypt）編碼密碼。
- **輸入清理：** 透過使用 Spring Data JPA 或參數化查詢防止 SQL 注入。透過正確編碼輸出防止跨網站腳本攻擊（XSS）。
