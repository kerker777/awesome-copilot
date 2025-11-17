---
mode: 'agent'
description: '確保 .NET/C# 程式碼符合解決方案/專案的最佳實踐。'
---
# .NET/C# 最佳實踐

您的任務是確保 ${selection} 中的 .NET/C# 程式碼符合此解決方案/專案的最佳實踐。這包括：

## 文件與結構

- 為所有公開的類別、介面、方法和屬性建立完整的 XML 文件註解
- 在 XML 註解中包含參數描述和返回值描述
- 遵循既定的命名空間結構：{Core|Console|App|Service}.{Feature}

## 設計模式與架構

- 使用主要建構函式語法進行相依性注入（例如，`public class MyClass(IDependency dependency)`）
- 使用泛型基底類別實作 Command Handler 模式（例如，`CommandHandler<TOptions>`）
- 使用具有清晰命名慣例的介面隔離（介面前綴加 'I'）
- 遵循 Factory 模式進行複雜物件建立

## 相依性注入與服務

- 使用建構函式相依性注入，並透過 ArgumentNullException 進行 null 檢查
- 使用適當的生命週期註冊服務（Singleton、Scoped、Transient）
- 使用 Microsoft.Extensions.DependencyInjection 模式
- 實作服務介面以提高可測試性

## 資源管理與本地化

- 使用 ResourceManager 管理本地化訊息和錯誤字串
- 分離 LogMessages 和 ErrorMessages 資源檔案
- 透過 `_resourceManager.GetString("MessageKey")` 存取資源

## Async/Await 模式

- 對所有 I/O 操作和長時間執行的任務使用 async/await
- 從非同步方法返回 Task 或 Task<T>
- 在適當的地方使用 ConfigureAwait(false)
- 適當地處理非同步例外

## 測試標準

- 使用 MSTest 框架搭配 FluentAssertions 進行斷言
- 遵循 AAA 模式（Arrange、Act、Assert）
- 使用 Moq 模擬相依性
- 測試成功和失敗情境
- 包含 null 參數驗證測試

## 設定與設置

- 使用具有資料註解的強類型設定類別
- 實作驗證屬性（Required、NotEmptyOrWhitespace）
- 使用 IConfiguration 繫結設定
- 支援 appsettings.json 設定檔

## Semantic Kernel 與 AI 整合

- 使用 Microsoft.SemanticKernel 進行 AI 操作
- 實作適當的核心設定和服務註冊
- 處理 AI 模型設定（ChatCompletion、Embedding 等）
- 使用結構化輸出模式以獲得可靠的 AI 回應

## 錯誤處理與日誌記錄

- 使用 Microsoft.Extensions.Logging 進行結構化日誌記錄
- 包含具有有意義內容的範圍日誌記錄
- 拋出具有描述性訊息的特定例外
- 對預期的失敗情境使用 try-catch 區塊

## 效能與安全性

- 在適用的地方使用 C# 12+ 功能和 .NET 8 最佳化
- 實作適當的輸入驗證和清理
- 對資料庫操作使用參數化查詢
- 遵循 AI/ML 操作的安全編碼實踐

## 程式碼品質

- 確保符合 SOLID 原則
- 透過基底類別和工具程式避免程式碼重複
- 使用反映領域概念的有意義名稱
- 保持方法專注且內聚
- 為資源實作適當的處置模式
