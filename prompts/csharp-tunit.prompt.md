---
mode: 'agent'
tools: ['changes', 'search/codebase', 'edit/editFiles', 'problems', 'search']
description: '取得 TUnit 單元測試的最佳實踐，包括資料驅動測試'
---

# TUnit 最佳實踐

您的目標是協助我使用 TUnit 撰寫有效的單元測試，涵蓋標準和資料驅動測試方法。

## 專案設定

- 使用單獨的測試專案，命名慣例為 `[專案名稱].Tests`
- 參考 TUnit 套件和 TUnit.Assertions（用於流暢斷言）
- 建立與被測試類別相符的測試類別（例如，`Calculator` 對應 `CalculatorTests`）
- 使用 .NET SDK 測試命令：`dotnet test` 執行測試
- TUnit 需要 .NET 8.0 或更高版本

## 測試結構

- 不需要測試類別屬性（與 xUnit/NUnit 不同）
- 在測試方法上使用 `[Test]` 屬性（而非 xUnit 的 `[Fact]`）
- 遵循 Arrange-Act-Assert (AAA) 模式
- 使用 `方法名稱_情境_預期行為` 模式命名測試
- 使用生命週期掛鉤：`[Before(Test)]` 用於設定，`[After(Test)]` 用於清理
- 使用 `[Before(Class)]` 和 `[After(Class)]` 在類別中的測試之間共享內容
- 使用 `[Before(Assembly)]` 和 `[After(Assembly)]` 在測試類別之間共享內容
- TUnit 支援進階生命週期掛鉤，如 `[Before(TestSession)]` 和 `[After(TestSession)]`

## 標準測試

- 保持測試專注於單一行為
- 避免在一個測試方法中測試多個行為
- 使用 TUnit 的流暢斷言語法搭配 `await Assert.That()`
- 僅包含驗證測試案例所需的斷言
- 使測試獨立且具有冪等性（可以任意順序執行）
- 避免測試相依性（如有需要可使用 `[DependsOn]` 屬性）

## 資料驅動測試

- 使用 `[Arguments]` 屬性提供行內測試資料（相當於 xUnit 的 `[InlineData]`）
- 使用 `[MethodData]` 提供基於方法的測試資料（相當於 xUnit 的 `[MemberData]`）
- 使用 `[ClassData]` 提供基於類別的測試資料
- 透過實作 `ITestDataSource` 建立自訂資料來源
- 在資料驅動測試中使用有意義的參數名稱
- 可以在同一個測試方法上套用多個 `[Arguments]` 屬性

## 斷言

- 使用 `await Assert.That(value).IsEqualTo(expected)` 進行值相等性比較
- 使用 `await Assert.That(value).IsSameReferenceAs(expected)` 進行參考相等性比較
- 使用 `await Assert.That(value).IsTrue()` 或 `await Assert.That(value).IsFalse()` 進行布林條件判斷
- 使用 `await Assert.That(collection).Contains(item)` 或 `await Assert.That(collection).DoesNotContain(item)` 處理集合
- 使用 `await Assert.That(value).Matches(pattern)` 進行正規表示式模式比對
- 使用 `await Assert.That(action).Throws<TException>()` 或 `await Assert.That(asyncAction).ThrowsAsync<TException>()` 測試例外
- 使用 `.And` 運算子串接斷言：`await Assert.That(value).IsNotNull().And.IsEqualTo(expected)`
- 使用 `.Or` 運算子表示替代條件：`await Assert.That(value).IsEqualTo(1).Or.IsEqualTo(2)`
- 使用 `.Within(tolerance)` 進行具有容差的 DateTime 和數值比較
- 所有斷言都是非同步的，必須使用 await

## 進階功能

- 使用 `[Repeat(n)]` 重複執行測試多次
- 使用 `[Retry(n)]` 在失敗時自動重試
- 使用 `[ParallelLimit<T>]` 控制平行執行限制
- 使用 `[Skip("原因")]` 有條件地跳過測試
- 使用 `[DependsOn(nameof(OtherTest))]` 建立測試相依性
- 使用 `[Timeout(milliseconds)]` 設定測試逾時
- 透過擴充 TUnit 的基礎屬性來建立自訂屬性

## 測試組織

- 按功能或元件分組測試
- 使用 `[Category("分類名稱")]` 進行測試分類
- 使用 `[DisplayName("自訂測試名稱")]` 設定自訂測試名稱
- 考慮使用 `TestContext` 進行測試診斷和資訊
- 對於特定平台的測試，使用條件屬性，如自訂的 `[WindowsOnly]`

## 效能和平行執行

- TUnit 預設以平行方式執行測試（與需要明確設定的 xUnit 不同）
- 使用 `[NotInParallel]` 停用特定測試的平行執行
- 使用 `[ParallelLimit<T>]` 搭配自訂限制類別來控制並行性
- 同一類別內的測試預設按順序執行
- 使用 `[Repeat(n)]` 搭配 `[ParallelLimit<T>]` 進行負載測試情境

## 從 xUnit 遷移

- 將 `[Fact]` 替換為 `[Test]`
- 將 `[Theory]` 替換為 `[Test]` 並使用 `[Arguments]` 提供資料
- 將 `[InlineData]` 替換為 `[Arguments]`
- 將 `[MemberData]` 替換為 `[MethodData]`
- 將 `Assert.Equal` 替換為 `await Assert.That(actual).IsEqualTo(expected)`
- 將 `Assert.True` 替換為 `await Assert.That(condition).IsTrue()`
- 將 `Assert.Throws<T>` 替換為 `await Assert.That(action).Throws<T>()`
- 將建構函式/IDisposable 替換為 `[Before(Test)]`/`[After(Test)]`
- 將 `IClassFixture<T>` 替換為 `[Before(Class)]`/`[After(Class)]`

**為什麼選擇 TUnit 而非 xUnit？**

TUnit 提供現代化、快速且靈活的測試體驗，具有 xUnit 所沒有的進階功能，例如非同步斷言、更精細的生命週期掛鉤以及改進的資料驅動測試能力。TUnit 的流暢斷言提供更清晰、更具表達力的測試驗證，使其特別適合複雜的 .NET 專案。
