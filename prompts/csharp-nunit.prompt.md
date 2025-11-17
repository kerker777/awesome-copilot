---
mode: 'agent'
tools: ['changes', 'search/codebase', 'edit/editFiles', 'problems', 'search']
description: '取得 NUnit 單元測試的最佳實踐，包括資料驅動測試'
---

# NUnit 最佳實踐

您的目標是協助我使用 NUnit 撰寫有效的單元測試，涵蓋標準和資料驅動測試方法。

## 專案設定

- 使用單獨的測試專案，命名慣例為 `[專案名稱].Tests`
- 參考 Microsoft.NET.Test.Sdk、NUnit 和 NUnit3TestAdapter 套件
- 建立與被測試類別相符的測試類別（例如，`Calculator` 對應 `CalculatorTests`）
- 使用 .NET SDK 測試命令：`dotnet test` 執行測試

## 測試結構

- 在測試類別上套用 `[TestFixture]` 屬性
- 在測試方法上使用 `[Test]` 屬性
- 遵循 Arrange-Act-Assert (AAA) 模式
- 使用 `方法名稱_情境_預期行為` 模式命名測試
- 使用 `[SetUp]` 和 `[TearDown]` 進行每個測試的設定和清理
- 使用 `[OneTimeSetUp]` 和 `[OneTimeTearDown]` 進行每個類別的設定和清理
- 使用 `[SetUpFixture]` 進行組件層級的設定和清理

## 標準測試

- 保持測試專注於單一行為
- 避免在一個測試方法中測試多個行為
- 使用表達意圖的清晰斷言
- 僅包含驗證測試案例所需的斷言
- 使測試獨立且具有冪等性（可以任意順序執行）
- 避免測試相依性

## 資料驅動測試

- 使用 `[TestCase]` 提供行內測試資料
- 使用 `[TestCaseSource]` 提供程式化產生的測試資料
- 使用 `[Values]` 進行簡單參數組合
- 使用 `[ValueSource]` 提供基於屬性或方法的資料來源
- 使用 `[Random]` 提供隨機數值測試值
- 使用 `[Range]` 提供連續數值測試值
- 使用 `[Combinatorial]` 或 `[Pairwise]` 組合多個參數

## 斷言

- 使用 `Assert.That` 搭配約束模型（NUnit 偏好的風格）
- 使用 `Is.EqualTo`、`Is.SameAs`、`Contains.Item` 等約束
- 使用 `Assert.AreEqual` 進行簡單值相等性比較（傳統風格）
- 使用 `CollectionAssert` 進行集合比較
- 使用 `StringAssert` 進行字串特定斷言
- 使用 `Assert.Throws<T>` 或 `Assert.ThrowsAsync<T>` 測試例外
- 在斷言中使用描述性訊息，以便在失敗時清楚說明

## 模擬和隔離

- 考慮搭配 NUnit 使用 Moq 或 NSubstitute
- 模擬相依性以隔離待測單元
- 使用介面來促進模擬
- 對於複雜的測試設定，考慮使用 DI 容器

## 測試組織

- 按功能或元件分組測試
- 使用 `[Category("分類名稱")]` 進行分類
- 使用 `[Order]` 在必要時控制測試執行順序
- 使用 `[Author("開發者名稱")]` 表示擁有者
- 使用 `[Description]` 提供額外的測試資訊
- 對於不應自動執行的測試，考慮使用 `[Explicit]`
- 使用 `[Ignore("原因")]` 暫時跳過測試
