---
mode: 'agent'
tools: ['changes', 'search/codebase', 'edit/editFiles', 'problems', 'search']
description: '取得 MSTest 單元測試的最佳實踐，包括資料驅動測試'
---

# MSTest 最佳實踐

您的目標是協助我使用 MSTest 撰寫有效的單元測試，涵蓋標準和資料驅動測試方法。

## 專案設定

- 使用單獨的測試專案，命名慣例為 `[專案名稱].Tests`
- 參考 MSTest 套件
- 建立與被測試類別相符的測試類別（例如，`Calculator` 對應 `CalculatorTests`）
- 使用 .NET SDK 測試命令：`dotnet test` 執行測試

## 測試結構

- 在測試類別上使用 `[TestClass]` 屬性
- 在測試方法上使用 `[TestMethod]` 屬性
- 遵循 Arrange-Act-Assert (AAA) 模式
- 使用 `方法名稱_情境_預期行為` 模式命名測試
- 使用 `[TestInitialize]` 和 `[TestCleanup]` 進行每個測試的設定和清理
- 使用 `[ClassInitialize]` 和 `[ClassCleanup]` 進行每個類別的設定和清理
- 使用 `[AssemblyInitialize]` 和 `[AssemblyCleanup]` 進行組件層級的設定和清理

## 標準測試

- 保持測試專注於單一行為
- 避免在一個測試方法中測試多個行為
- 使用表達意圖的清晰斷言
- 僅包含驗證測試案例所需的斷言
- 使測試獨立且具有冪等性（可以任意順序執行）
- 避免測試相依性

## 資料驅動測試

- 將 `[TestMethod]` 與資料來源屬性結合使用
- 使用 `[DataRow]` 提供行內測試資料
- 使用 `[DynamicData]` 提供程式化產生的測試資料
- 使用 `[TestProperty]` 為測試新增中繼資料
- 在資料驅動測試中使用有意義的參數名稱

## 斷言

- 使用 `Assert.AreEqual` 進行值相等性比較
- 使用 `Assert.AreSame` 進行參考相等性比較
- 使用 `Assert.IsTrue`/`Assert.IsFalse` 進行布林條件判斷
- 使用 `CollectionAssert` 進行集合比較
- 使用 `StringAssert` 進行字串特定斷言
- 使用 `Assert.Throws<T>` 測試例外
- 確保斷言本質上簡單，並提供訊息以便在失敗時清楚說明

## 模擬和隔離

- 考慮搭配 MSTest 使用 Moq 或 NSubstitute
- 模擬相依性以隔離待測單元
- 使用介面來促進模擬
- 對於複雜的測試設定，考慮使用 DI 容器

## 測試組織

- 按功能或元件分組測試
- 使用 `[TestCategory("分類")]` 進行測試分類
- 使用 `[Priority(1)]` 標記關鍵測試的優先順序
- 使用 `[Owner("開發者名稱")]` 表示擁有者
