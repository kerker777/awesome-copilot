---
mode: 'agent'
tools: ['changes', 'search/codebase', 'edit/editFiles', 'problems']
description: '建立具有適當 OpenAPI 文件的 ASP.NET Minimal API 端點'
---

# ASP.NET Minimal API 與 OpenAPI

您的目標是協助我建立結構良好的 ASP.NET Minimal API 端點，具有正確的類型和全面的 OpenAPI/Swagger 文件。

## API 組織

- 使用 `MapGroup()` 擴充方法對相關端點進行分組
- 使用端點篩選器處理跨領域關注點
- 使用單獨的端點類別結構化較大的 API
- 對於複雜的 API，考慮使用基於功能的資料夾結構

## 請求和回應類型

- 定義明確的請求和回應 DTO/模型
- 建立具有適當驗證屬性的清晰模型類別
- 為不可變的請求/回應物件使用 record 類型
- 使用符合 API 設計標準的有意義的屬性名稱
- 套用 `[Required]` 和其他驗證屬性以強制執行約束
- 使用 ProblemDetailsService 和 StatusCodePages 獲得標準錯誤回應

## 類型處理

- 使用具有明確類型綁定的強類型路由參數
- 使用 `Results<T1, T2>` 表示多個回應類型
- 返回 `TypedResults` 而非 `Results` 以獲得強類型回應
- 利用 C# 10+ 功能，如可為 null 的註解和僅初始化屬性

## OpenAPI 文件

- 使用 .NET 9 中加入的內建 OpenAPI 文件支援
- 定義操作摘要和描述
- 使用 `WithName` 擴充方法加入 operationIds
- 使用 `[Description()]` 為屬性和參數加入描述
- 為請求和回應設定適當的內容類型
- 使用文件轉換器加入伺服器、標籤和安全性配置等元素
- 使用架構轉換器對 OpenAPI 架構套用自訂
