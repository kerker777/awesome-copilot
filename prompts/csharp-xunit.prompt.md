---
mode: 'agent'
tools: ['changes', 'search/codebase', 'edit/editFiles', 'problems', 'search']
description: '取得 XUnit 單元測試的最佳實踐，包括資料驅動測試'
---

# XUnit 最佳實踐

您的目標是協助我使用 XUnit 撰寫有效的單元測試，涵蓋標準和資料驅動測試方法。

## 專案設定

- 使用單獨的測試專案，命名慣例為 `[專案名稱].Tests`
- 參考 Microsoft.NET.Test.Sdk、xunit 和 xunit.runner.visualstudio 套件
- 建立與被測試類別相符的測試類別（例如，`Calculator` 對應 `CalculatorTests`）
- 使用 .NET SDK 測試命令：`dotnet test` 執行測試

## 測試結構

- 不需要測試類別屬性（與 MSTest/NUnit 不同）
- 對簡單測試使用 `[Fact]` 屬性進行基於事實的測試
- 遵循 Arrange-Act-Assert (AAA) 模式
- 使用 `方法名稱_情境_預期行為` 模式命名測試
- 使用建構函式進行設定，使用 `IDisposable.Dispose()` 進行清理
- 使用 `IClassFixture<T>` 在類別中的測試之間共享內容
- 使用 `ICollectionFixture<T>` 在多個測試類別之間共享內容

## 標準測試

- 保持測試專注於單一行為
- 避免在一個測試方法中測試多個行為
- 使用表達意圖的清晰斷言
- 僅包含驗證測試案例所需的斷言
- 使測試獨立且具有冪等性（可以任意順序執行）
- 避免測試相依性

## 資料驅動測試

- 將 `[Theory]` 與資料來源屬性結合使用
- 使用 `[InlineData]` 提供行內測試資料
- 使用 `[MemberData]` 提供基於方法的測試資料
- 使用 `[ClassData]` 提供基於類別的測試資料
- 透過實作 `DataAttribute` 建立自訂資料屬性
- 在資料驅動測試中使用有意義的參數名稱

## 斷言

- 使用 `Assert.Equal` 進行值相等性比較
- 使用 `Assert.Same` 進行參考相等性比較
- 使用 `Assert.True`/`Assert.False` 進行布林條件判斷
- 使用 `Assert.Contains`/`Assert.DoesNotContain` 處理集合
- 使用 `Assert.Matches`/`Assert.DoesNotMatch` 進行正規表示式模式比對
- 使用 `Assert.Throws<T>` 或 `await Assert.ThrowsAsync<T>` 測試例外
- 使用流暢斷言程式庫以獲得更易讀的斷言

## 模擬和隔離

- 考慮搭配 XUnit 使用 Moq 或 NSubstitute
- 模擬相依性以隔離待測單元
- 使用介面來促進模擬
- 對於複雜的測試設定，考慮使用 DI 容器

## 測試組織

- 按功能或元件分組測試
- 使用 `[Trait("Category", "分類名稱")]` 進行分類
- 使用集合固定裝置將具有共享相依性的測試分組
- 考慮使用輸出協助程式（`ITestOutputHelper`）進行測試診斷
- 在 fact/theory 屬性中使用 `Skip = "原因"` 有條件地跳過測試
