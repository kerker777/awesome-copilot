---
description: '.NET MAUI 元件和應用程式模式'
applyTo: '**/*.xaml, **/*.cs'
---

# .NET MAUI

## .NET MAUI 程式碼風格和結構

- 撰寫慣用且高效的 .NET MAUI 和 C# 程式碼。
- 遵循 .NET 和 .NET MAUI 慣例。
- 對於較小的元件優先使用內聯函式，但將複雜的邏輯分離到程式碼後置或服務類別中。
- 應該適當使用 Async/await 以確保非阻塞的 UI 操作。

## 命名慣例

- 對元件名稱、方法名稱和公開成員使用 PascalCase。
- 對私有欄位和區域變數使用 camelCase。
- 介面名稱前綴使用 "I"（例如 IUserService）。

## .NET MAUI 和 .NET 特定指南

- 利用 .NET MAUI 的內建功能處理元件生命週期（例如 OnAppearing、OnDisappearing）。
- 有效使用 {Binding} 進行資料繫結。
- 遵循關注點分離原則來結構化 .NET MAUI 元件和服務。
- 始終使用最新版本的 C#，目前為 C# 13，包括記錄型別、模式比對和全域 using 等功能。

## 錯誤處理和驗證

- 為 .NET MAUI 頁面和 API 呼叫實作適當的錯誤處理。
- 使用日誌記錄來追蹤後端錯誤，並考慮使用 MAUI Community Toolkit 的 Logger 等工具在 MAUI 中捕捉 UI 層級的錯誤。
- 在表單中使用 FluentValidation 或 DataAnnotations 實作驗證。

## MAUI API 和效能最佳化

- 利用 MAUI 的內建功能處理元件生命週期（例如 OnAppearing、OnDisappearing）。
- 對可能阻塞主執行緒的 API 呼叫或 UI 動作使用非同步方法（async/await）。
- 透過減少不必要的渲染並有效使用 OnPropertyChanged() 來最佳化 MAUI 元件。
- 透過避免不必要的重新渲染來最小化元件渲染樹，適當時使用 BatchBegin() 和 BatchCommit()。

## 快取策略

- 為經常使用的資料實作記憶體內快取，特別是對於 MAUI 應用程式。使用 IMemoryCache 作為輕量級快取解決方案。
- 對於需要在多個使用者或客戶端之間共享狀態的大型應用程式，考慮分散式快取策略（如 Redis 或 SQL Server Cache）。
- 透過儲存回應來快取 API 呼叫，避免在資料不太可能變更時進行冗餘呼叫，從而改善使用者體驗。

## 狀態管理函式庫

- 使用依賴注入和 .NET MAUI Community Toolkit 在元件之間共享狀態。

## API 設計和整合

- 使用 HttpClient 或其他適當的服務與外部 API 或您自己的後端通訊。
- 使用 try-catch 為 API 呼叫實作錯誤處理，並在 UI 中提供適當的使用者回饋。

## 測試和除錯

- 使用 xUnit、NUnit 或 MSTest 測試元件和服務。
- 在測試期間使用 Moq 或 NSubstitute 模擬依賴項。

## 安全性和驗證

- 在必要時在 MAUI 應用程式中實作驗證和授權，使用 OAuth 或 JWT 權杖進行 API 驗證。
- 對所有網路通訊使用 HTTPS，並確保實作適當的 CORS 政策。

## API 文件和 Swagger

- 對您的後端 API 服務使用 Swagger/OpenAPI 進行 API 文件。
- 確保為模型和 API 方法提供 XML 文件以增強 Swagger 文件。
