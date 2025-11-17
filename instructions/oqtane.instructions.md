---
description: 'Oqtane Module patterns'
applyTo: '**/*.razor, **/*.razor.cs, **/*.razor.css'
---

## Blazor 程式碼風格和結構

- 撰寫慣用且高效的 Blazor 和 C# 程式碼。
- 遵循 .NET 和 Blazor 慣例。
- 適當使用 Razor Components 進行基於元件的 UI 開發。
- 適當使用 Blazor Components 進行基於元件的 UI 開發。
- 對於較小的元件優先使用內聯函式，但將複雜邏輯分離到 code-behind 或服務類別中。
- 在適用的情況下應使用 Async/await 以確保非阻塞 UI 操作。


## 命名慣例

- 元件名稱、方法名稱和公開成員遵循 PascalCase。
- 私有欄位和區域變數使用 camelCase。
- 介面名稱以 "I" 為前綴（例如 IUserService）。

## Blazor 和 .NET 特定指南

- 利用 Blazor 的內建功能進行元件生命週期（例如 OnInitializedAsync、OnParametersSetAsync）。
- 使用 @bind 有效進行資料繫結。
- 在 Blazor 中利用依賴注入來處理服務。
- 遵循關注點分離來結構化 Blazor 元件和服務。
- 始終使用最新版本的 C#，目前為 C# 13 功能，如記錄型別、模式匹配和全域 using。

## Oqtane 特定指南
- 參見 [Main Oqtane repo](https://github.com/oqtane/oqtane.framework) 中的基礎類別和模式
- 遵循客戶端伺服器模式進行模組開發。
- Client 專案在 modules 資料夾中有各種模組。
- 客戶端模組中的每個動作都是一個繼承自 ModuleBase 的獨立 razor 檔案，index.razor 是預設動作。
- 對於複雜的客戶端處理（如取得資料），建立繼承自 ServiceBase 的服務類別並位於 services 資料夾中。每個模組一個服務類別。
- 客戶端服務應使用 ServiceBase 方法呼叫伺服器端點
- Server 專案包含 MVC Controllers，每個模組一個 Controller，對應客戶端服務呼叫。每個 controller 將呼叫由 DI 管理的伺服器端服務或儲存庫
- Server 專案對模組使用儲存庫模式，每個模組一個儲存庫類別以對應 controllers。

## 錯誤處理和驗證

- 為 Blazor 頁面和 API 呼叫實作適當的錯誤處理。
- 使用基礎類別中的內建 Oqtane 記錄方法。
- 在後端使用記錄來追蹤錯誤，並考慮使用 ErrorBoundary 等工具在 Blazor 中捕獲 UI 層級的錯誤。
- 在表單中使用 FluentValidation 或 DataAnnotations 實作驗證。

## Blazor API 和效能最佳化

- 根據專案需求最佳地使用 Blazor 伺服器端或 WebAssembly。
- 對於可能阻塞主執行緒的 API 呼叫或 UI 動作使用非同步方法（async/await）。
- 透過減少不必要的渲染並有效使用 StateHasChanged() 來最佳化 Razor 元件。
- 透過避免不必要的重新渲染來最小化元件渲染樹，在適當的地方使用 ShouldRender()。
- 使用 EventCallbacks 有效處理使用者互動，在觸發事件時僅傳遞最少的資料。

## 快取策略

- 為經常使用的資料實作記憶體內快取，特別是對於 Blazor Server 應用程式。使用 IMemoryCache 進行輕量級快取解決方案。
- 對於 Blazor WebAssembly，利用 localStorage 或 sessionStorage 在使用者會話之間快取應用程式狀態。
- 對於需要在多個使用者或客戶端之間共享狀態的較大應用程式，考慮分散式快取策略（如 Redis 或 SQL Server Cache）。
- 透過儲存回應來快取 API 呼叫，以避免在資料不太可能變更時進行冗餘呼叫，從而改善使用者體驗。

## 狀態管理函式庫

- 使用 Blazor 的內建 Cascading Parameters 和 EventCallbacks 在元件之間進行基本狀態共享。
- 在適當時使用基礎類別中內建的 Oqtane 狀態管理，如 PageState 和 SiteState。
- 當應用程式在複雜性上增長時，避免新增像 Fluxor 或 BlazorState 這樣的額外依賴項。
- 對於 Blazor WebAssembly 中的客戶端狀態持久化，考慮使用 Blazored.LocalStorage 或 Blazored.SessionStorage 來維護頁面重新載入之間的狀態。
- 對於伺服器端 Blazor，使用 Scoped Services 和 StateContainer 模式在使用者會話中管理狀態，同時最小化重新渲染。

## API 設計和整合

- 使用 service base 方法與外部 API 或伺服器專案後端通訊。
- 使用 try-catch 為 API 呼叫實作錯誤處理，並在 UI 中提供適當的使用者回饋。

## 在 Visual Studio 中進行測試和偵錯

- 所有單元測試和整合測試應在 Visual Studio Enterprise 中完成。
- 使用 xUnit、NUnit 或 MSTest 測試 Blazor 元件和服務。
- 在測試期間使用 Moq 或 NSubstitute 來模擬依賴項。
- 使用瀏覽器開發者工具偵錯 Blazor UI 問題，使用 Visual Studio 的偵錯工具處理後端和伺服器端問題。
- 對於效能分析和最佳化，依賴 Visual Studio 的診斷工具。

## 安全性和身份驗證

- 使用內建的 Oqtane 基礎類別成員（如 User.Roles）實作身份驗證和授權。
- 對所有 Web 通訊使用 HTTPS，並確保實作適當的 CORS 政策。
