---
description: 'Blazor 元件和應用程式模式'
applyTo: '**/*.razor, **/*.razor.cs, **/*.razor.css'
---

## Blazor 程式碼樣式和結構

- 撰寫慣用且高效的 Blazor 和 C# 程式碼。
- 遵循 .NET 和 Blazor 慣例。
- 適當使用 Razor 元件進行基於元件的 UI 開發。
- 對較小的元件偏好使用內聯函數,但將複雜邏輯分離到程式碼後置或服務類別中。
- 應在適用的地方使用 Async/await 以確保非阻塞的 UI 操作。

## 命名慣例

- 對元件名稱、方法名稱和公共成員遵循 PascalCase。
- 對私有欄位和區域變數使用 camelCase。
- 介面名稱以 "I" 為前綴(例如,IUserService)。

## Blazor 和 .NET 特定指南

- 利用 Blazor 的內建功能管理元件生命週期(例如,OnInitializedAsync、OnParametersSetAsync)。
- 使用 @bind 有效地進行資料繫結。
- 在 Blazor 中利用依賴注入提供服務。
- 遵循關注點分離原則組織 Blazor 元件和服務。
- 始終使用最新版本的 C#,目前是 C# 13 功能,如記錄型別、模式匹配和全域 using。

## 錯誤處理和驗證

- 對 Blazor 頁面和 API 呼叫實作適當的錯誤處理。
- 在後端使用記錄進行錯誤追蹤,並考慮使用 ErrorBoundary 等工具在 Blazor 中擷取 UI 層級錯誤。
- 在表單中使用 FluentValidation 或 DataAnnotations 實作驗證。

## Blazor API 和效能最佳化

- 根據專案需求最佳地使用 Blazor 伺服器端或 WebAssembly。
- 對可能阻塞主執行緒的 API 呼叫或 UI 動作使用非同步方法(async/await)。
- 透過減少不必要的渲染並有效使用 StateHasChanged() 來最佳化 Razor 元件。
- 透過避免不必要的重新渲染來最小化元件渲染樹,適時使用 ShouldRender()。
- 使用 EventCallbacks 有效處理使用者互動,在觸發事件時僅傳遞最少的資料。

## 快取策略

- 對經常使用的資料實作記憶體內快取,特別是對於 Blazor Server 應用程式。使用 IMemoryCache 進行輕量級快取解決方案。
- 對於 Blazor WebAssembly,利用 localStorage 或 sessionStorage 在使用者會話之間快取應用程式狀態。
- 對需要在多個使用者或客戶端之間共享狀態的大型應用程式,考慮分散式快取策略(如 Redis 或 SQL Server Cache)。
- 透過儲存回應來快取 API 呼叫,以避免在資料不太可能變更時進行冗餘呼叫,從而改善使用者體驗。

## 狀態管理函式庫

- 使用 Blazor 的內建級聯參數和 EventCallbacks 在元件之間進行基本狀態共享。
- 當應用程式複雜度增加時,使用 Fluxor 或 BlazorState 等函式庫實作進階狀態管理解決方案。
- 對於 Blazor WebAssembly 中的客戶端狀態持久性,考慮使用 Blazored.LocalStorage 或 Blazored.SessionStorage 在頁面重新載入之間維護狀態。
- 對於伺服器端 Blazor,使用範疇服務和 StateContainer 模式在使用者會話內管理狀態,同時最小化重新渲染。

## API 設計和整合

- 使用 HttpClient 或其他適當的服務與外部 API 或您自己的後端進行通訊。
- 使用 try-catch 為 API 呼叫實作錯誤處理,並在 UI 中提供適當的使用者回饋。

## Visual Studio 中的測試和除錯

- 所有單元測試和整合測試都應在 Visual Studio Enterprise 中完成。
- 使用 xUnit、NUnit 或 MSTest 測試 Blazor 元件和服務。
- 在測試期間使用 Moq 或 NSubstitute 模擬依賴項。
- 使用瀏覽器開發人員工具除錯 Blazor UI 問題,使用 Visual Studio 的除錯工具處理後端和伺服器端問題。
- 對於效能分析和最佳化,依賴 Visual Studio 的診斷工具。

## 安全性和驗證

- 必要時在 Blazor 應用程式中實作驗證和授權,使用 ASP.NET Identity 或 JWT 權杖進行 API 驗證。
- 對所有 Web 通訊使用 HTTPS,並確保實作適當的 CORS 政策。

## API 文件和 Swagger

- 對後端 API 服務使用 Swagger/OpenAPI 進行 API 文件。
- 確保模型和 API 方法的 XML 文件以增強 Swagger 文件。
