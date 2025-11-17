---
description: '構建 Spring Boot 基礎應用程式的指南'
applyTo: '**/*.java, **/*.kt'
---

# Spring Boot 開發

## 一般指引

- 在審查程式碼更改時僅提出高信心的建議。
- 撰寫具有良好可維護性實踐的程式碼，包括對某些設計決策原因的註解。
- 處理邊緣情況並撰寫清晰的例外處理。
- 對於函式庫或外部依賴項，在註解中提及其用途和目的。

## Spring Boot 指引

### 依賴注入

- 對所有必需的依賴項使用建構函式注入。
- 將依賴項欄位宣告為 `private final`。

### 配置

- 使用 YAML 檔案（`application.yml`）進行外部化配置。
- 環境設定檔：針對不同環境使用 Spring 設定檔（dev、test、prod）
- 配置屬性：使用 @ConfigurationProperties 進行型別安全的配置綁定
- 金鑰管理：使用環境變數或金鑰管理系統外部化金鑰

### 程式碼組織

- 套件結構：按功能/領域而非按層組織
- 關注點分離：保持控制器簡潔、服務專注、儲存庫簡單
- 工具類別：使工具類別為 final 且具有私有建構函式

### 服務層

- 將業務邏輯放在帶有 `@Service` 註解的類別中。
- 服務應該是無狀態且可測試的。
- 透過建構函式注入儲存庫。
- 服務方法簽名應使用領域 ID 或 DTO，除非必要，否則不直接公開儲存庫實體。

### 日誌記錄

- 所有日誌記錄使用 SLF4J（`private static final Logger logger = LoggerFactory.getLogger(MyClass.class);`）。
- 不要直接使用具體實作（Logback、Log4j2）或 `System.out.println()`。
- 使用參數化日誌記錄：`logger.info("User {} logged in", userId);`。

### 安全性和輸入處理

- 使用參數化查詢 | 始終使用 Spring Data JPA 或 `NamedParameterJdbcTemplate` 來防止 SQL 注入。
- 使用 JSR-380（`@NotNull`、`@Size` 等）註解和 `BindingResult` 驗證請求主體和參數

## 構建和驗證

- 在新增或修改程式碼後，驗證專案繼續成功構建。
- 如果專案使用 Maven，執行 `mvn clean package`。
- 如果專案使用 Gradle，執行 `./gradlew build`（或在 Windows 上執行 `gradlew.bat build`）。
- 確保所有測試作為構建的一部分通過。

## 實用命令

| Gradle 命令               | Maven 命令                        | 描述                                          |
|:-------------------------|:---------------------------------|:---------------------------------------------|
| `./gradlew bootRun`      |`./mvnw spring-boot:run`          | 執行應用程式。                                  |
| `./gradlew build`        |`./mvnw package`                  | 構建應用程式。                                  |
| `./gradlew test`         |`./mvnw test`                     | 執行測試。                                     |
| `./gradlew bootJar`      |`./mvnw spring-boot:repackage`    | 將應用程式打包為 JAR。                          |
| `./gradlew bootBuildImage`|`./mvnw spring-boot:build-image` | 將應用程式打包為容器映像。                       |
