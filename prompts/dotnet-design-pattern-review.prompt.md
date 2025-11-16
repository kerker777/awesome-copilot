---
mode: 'agent'
description: '審查 C#/.NET 程式碼的設計模式實作並建議改進。'
---
# .NET/C# 設計模式審查

審查 ${selection} 中的 C#/.NET 程式碼的設計模式實作，並為解決方案/專案建議改進。不要對程式碼進行任何更改，只提供審查意見。

## 必要的設計模式

- **Command 模式**：泛型基底類別（`CommandHandler<TOptions>`）、`ICommandHandler<TOptions>` 介面、`CommandHandlerOptions` 繼承、靜態 `SetupCommand(IHost host)` 方法
- **Factory 模式**：複雜物件建立服務提供者整合
- **Dependency Injection**：主要建構函式語法、`ArgumentNullException` null 檢查、介面抽象、適當的服務生命週期
- **Repository 模式**：非同步資料存取介面提供者連接抽象
- **Provider 模式**：外部服務抽象（資料庫、AI）、清晰的合約、設定處理
- **Resource 模式**：ResourceManager 用於本地化訊息、分離的 .resx 檔案（LogMessages、ErrorMessages）

## 審查檢查清單

- **設計模式**：識別使用的模式。Command Handler、Factory、Provider 和 Repository 模式是否正確實作？是否缺少有益的模式？
- **架構**：是否遵循命名空間慣例（`{Core|Console|App|Service}.{Feature}`）？Core/Console 專案之間是否適當分離？是否模組化且可讀？
- **.NET 最佳實踐**：主要建構函式、具有 Task 返回的 async/await、ResourceManager 使用、結構化日誌記錄、強類型設定？
- **GoF 模式**：Command、Factory、Template Method、Strategy 模式是否正確實作？
- **SOLID 原則**：單一職責、開放/封閉、里氏替換、介面隔離、相依性反轉是否有違反？
- **效能**：適當的 async/await、資源處置、ConfigureAwait(false)、平行處理機會？
- **可維護性**：關注點清晰分離、一致的錯誤處理、適當的設定使用？
- **可測試性**：相依性是否透過介面抽象、元件是否可模擬、非同步可測試性、AAA 模式相容性？
- **安全性**：輸入驗證、安全憑證處理、參數化查詢、安全的例外處理？
- **文件**：公開 API 的 XML 文件、參數/返回描述、資源檔案組織？
- **程式碼清晰度**：反映領域概念的有意義名稱、透過模式表達清晰意圖、自我說明的結構？
- **整潔程式碼**：一致的風格、適當的方法/類別大小、最小複雜度、消除重複？

## 改進重點領域

- **Command Handlers**：基底類別中的驗證、一致的錯誤處理、適當的資源管理
- **Factories**：相依性設定、服務提供者整合、處置模式
- **Providers**：連接管理、非同步模式、例外處理和日誌記錄
- **Configuration**：資料註解、驗證屬性、安全敏感值處理
- **AI/ML Integration**：Semantic Kernel 模式、結構化輸出處理、模型設定

提供與專案架構和 .NET 最佳實踐對齊的具體、可操作的改進建議。
