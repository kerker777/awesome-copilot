---
description: '使用 .NET Framework 專案的指南。包括專案結構、C# 語言版本、NuGet 管理和最佳實務。'
applyTo: '**/*.csproj, **/*.cs'
---

# .NET Framework 開發

## 建置與編譯需求
- 永遠使用 `msbuild /t:rebuild` 來建置解決方案或專案，而不是 `dotnet build`

## 專案檔案管理

### 非 SDK 樣式專案結構
.NET Framework 專案使用舊式專案格式，與現代 SDK 樣式專案有顯著差異：

- **明確檔案包含**：所有新的原始碼檔案**必須**明確地使用 `<Compile>` 元素加入到專案檔（`.csproj`）中
  - .NET Framework 專案不會像 SDK 樣式專案那樣自動包含目錄中的檔案
  - 範例：`<Compile Include="Path\To\NewFile.cs" />`

- **無隱含匯入**：與 SDK 樣式專案不同，.NET Framework 專案不會自動匯入常見的命名空間或組件

- **建置配置**：包含明確的 Debug/Release 配置的 `<PropertyGroup>` 區段

- **輸出路徑**：明確定義 `<OutputPath>` 和 `<IntermediateOutputPath>`

- **目標框架**：使用 `<TargetFrameworkVersion>` 而不是 `<TargetFramework>`
  - 範例：`<TargetFrameworkVersion>v4.7.2</TargetFrameworkVersion>`

## NuGet 套件管理
- 在 .NET Framework 專案中安裝和更新 NuGet 套件是一項複雜的任務，需要協調多個檔案的變更。因此，**不要嘗試安裝或更新 NuGet 套件**。
- 相反地，如果需要變更 NuGet 參考，請要求使用者使用 Visual Studio NuGet 套件管理員或 Visual Studio 套件管理員主控台來安裝或更新 NuGet 套件。
- 在推薦 NuGet 套件時，請確保它們與 .NET Framework 或 .NET Standard 2.0 相容（不只是 .NET Core 或 .NET 5+）。

## C# 語言版本為 7.3
- 此專案僅限於使用 C# 7.3 功能。請避免使用：

### C# 8.0+ 功能（不支援）：
  - Using 宣告（`using var stream = ...`）
  - Await using 陳述式（`await using var resource = ...`）
  - Switch 運算式（`variable switch { ... }`）
  - Null 聯合指派（`??=`）
  - 範圍和索引運算子（`array[1..^1]`、`array[^1]`）
  - 預設介面方法
  - 結構中的唯讀成員
  - 靜態區域函式
  - 可為 Null 的參考型別（`string?`、`#nullable enable`）

### C# 9.0+ 功能（不支援）：
  - 記錄（`public record Person(string Name)`）
  - Init-only 屬性（`{ get; init; }`）
  - 最上層程式（沒有 Main 方法的程式）
  - 模式比對增強功能
  - 目標型別的 new 運算式（`List<string> list = new()`）

### C# 10+ 功能（不支援）：
  - 全域 using 陳述式
  - 檔案範圍的命名空間
  - 記錄結構
  - 必要成員

### 替代方案（C# 7.3 相容）：
  - 使用傳統的帶大括號的 using 陳述式
  - 使用 switch 陳述式而不是 switch 運算式
  - 使用明確的 null 檢查而不是 null 聯合指派
  - 使用手動索引進行陣列切片
  - 使用抽象類別或介面而不是預設介面方法

## 環境考量（Windows 環境）
- 使用反斜線的 Windows 樣式路徑（例如，`C:\path\to\file.cs`）
- 在建議終端機操作時使用適合 Windows 的命令
- 在處理檔案系統操作時考慮 Windows 特定的行為

## 常見的 .NET Framework 陷阱和最佳實務

### Async/Await 模式
- **ConfigureAwait(false)**：在程式庫程式碼中永遠使用 `ConfigureAwait(false)` 以避免死鎖：
  ```csharp
  var result = await SomeAsyncMethod().ConfigureAwait(false);
  ```
- **避免同步覆蓋非同步**：不要使用 `.Result` 或 `.Wait()` 或 `.GetAwaiter().GetResult()`。這些同步覆蓋非同步模式可能導致死鎖和效能不佳。永遠對非同步呼叫使用 `await`。

### DateTime 處理
- **使用 DateTimeOffset 處理時間戳記**：優先使用 `DateTimeOffset` 而不是 `DateTime` 來處理絕對時間點
- **指定 DateTimeKind**：使用 `DateTime` 時，永遠指定 `DateTimeKind.Utc` 或 `DateTimeKind.Local`
- **文化感知格式化**：使用 `CultureInfo.InvariantCulture` 進行序列化/解析

### 字串操作
- **StringBuilder 用於串接**：使用 `StringBuilder` 進行多個字串串接
- **StringComparison**：永遠為字串操作指定 `StringComparison`：
  ```csharp
  string.Equals(other, StringComparison.OrdinalIgnoreCase)
  ```

### 記憶體管理
- **Dispose 模式**：正確實作 `IDisposable` 以處理非受控資源
- **Using 陳述式**：永遠將 `IDisposable` 物件包裝在 using 陳述式中
- **避免大型物件堆積**：保持物件小於 85KB 以避免 LOH 配置

### 配置
- **使用 ConfigurationManager**：透過 `ConfigurationManager.AppSettings` 存取應用程式設定
- **連接字串**：儲存在 `<connectionStrings>` 區段中，而不是 `<appSettings>`
- **轉換**：使用 web.config/app.config 轉換來處理特定環境的設定

### 例外處理
- **特定例外**：捕捉特定的例外型別，而不是一般的 `Exception`
- **不要吞沒例外**：永遠適當地記錄或重新擲回例外
- **對可處置資源使用 using**：確保即使發生例外時也能正確清理

### 效能考量
- **避免裝箱**：注意值型別和泛型的裝箱/拆箱
- **字串駐留**：審慎使用 `string.Intern()` 處理經常使用的字串
- **延遲初始化**：使用 `Lazy<T>` 進行昂貴的物件建立
- **避免在熱路徑中使用反射**：盡可能快取 `MethodInfo`、`PropertyInfo` 物件
