---
mode: 'agent'
tools: ['changes', 'search/codebase', 'edit/editFiles', 'problems', 'search']
description: '取得 JUnit 5 單元測試的最佳實踐,包括資料驅動測試'
---

# JUnit 5+ 最佳實踐

您的目標是協助我使用 JUnit 5 撰寫有效的單元測試,涵蓋標準和資料驅動測試方法。

## 專案設定

- 使用標準的 Maven 或 Gradle 專案結構。
- 將測試原始碼放在 `src/test/java` 中。
- 包含 `junit-jupiter-api`、`junit-jupiter-engine` 和 `junit-jupiter-params`（用於參數化測試）的相依性。
- 使用建置工具命令執行測試：`mvn test` 或 `gradle test`。

## 測試結構

- 測試類別應該有 `Test` 後綴,例如 `Calculator` 類別對應 `CalculatorTest`。
- 使用 `@Test` 標記測試方法。
- 遵循 Arrange-Act-Assert (AAA) 模式。
- 使用描述性慣例命名測試,如 `methodName_should_expectedBehavior_when_scenario`。
- 使用 `@BeforeEach` 和 `@AfterEach` 進行每個測試的設定和清理。
- 使用 `@BeforeAll` 和 `@AfterAll` 進行每個類別的設定和清理（必須是靜態方法）。
- 使用 `@DisplayName` 為測試類別和方法提供易於閱讀的名稱。

## 標準測試

- 保持測試專注於單一行為。
- 避免在一個測試方法中測試多個條件。
- 使測試獨立且具有冪等性（可以任意順序執行）。
- 避免測試相互依賴。

## 資料驅動（參數化）測試

- 使用 `@ParameterizedTest` 標記方法為參數化測試。
- 使用 `@ValueSource` 提供簡單的字面值（字串、整數等）。
- 使用 `@MethodSource` 參考提供測試參數的工廠方法（以 `Stream`、`Collection` 等形式）。
- 使用 `@CsvSource` 提供內聯逗號分隔值。
- 使用 `@CsvFileSource` 使用類別路徑中的 CSV 檔案。
- 使用 `@EnumSource` 使用列舉常數。

## 斷言

- 使用 `org.junit.jupiter.api.Assertions` 的靜態方法（例如 `assertEquals`、`assertTrue`、`assertNotNull`）。
- 為了更流暢和易讀的斷言,考慮使用 AssertJ 之類的程式庫（`assertThat(...).is...`）。
- 使用 `assertThrows` 或 `assertDoesNotThrow` 測試例外。
- 使用 `assertAll` 將相關斷言分組,以確保在測試失敗前檢查所有斷言。
- 在斷言中使用描述性訊息,以便在失敗時提供清晰的說明。

## 模擬和隔離

- 使用 Mockito 等模擬框架為相依性建立模擬物件。
- 使用 Mockito 的 `@Mock` 和 `@InjectMocks` 註解簡化模擬建立和注入。
- 使用介面來促進模擬。

## 測試組織

- 使用套件按功能或元件分組測試。
- 使用 `@Tag` 對測試進行分類（例如 `@Tag("fast")`、`@Tag("integration")`）。
- 在嚴格必要時,使用 `@TestMethodOrder(MethodOrderer.OrderAnnotation.class)` 和 `@Order` 控制測試執行順序。
- 使用 `@Disabled` 暫時跳過測試方法或類別,並提供原因。
- 使用 `@Nested` 在巢狀內部類別中分組測試,以獲得更好的組織和結構。
