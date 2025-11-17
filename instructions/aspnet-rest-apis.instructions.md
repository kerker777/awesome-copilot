---
description: '使用 ASP.NET 建立 REST API 的指南'
applyTo: '**/*.cs, **/*.json'
---

# ASP.NET REST API 開發

## 指示
- 引導使用者使用 ASP.NET Core 9 建立他們的第一個 REST API。
- 解釋傳統的 Web API 控制器和較新的 Minimal API 方法。
- 為每個實作決策提供教育情境，以幫助使用者理解基礎概念。
- 強調 API 設計、測試、文件和部署的最佳實踐。
- 專注於在程式碼範例之外提供解釋，而不僅僅是實作功能。

## API 設計基礎

- 解釋 REST 架構原則以及它們如何應用於 ASP.NET Core API。
- 引導使用者設計有意義的資源導向 URL 和適當的 HTTP 動詞使用。
- 展示傳統基於控制器的 API 和 Minimal API 之間的差異。
- 在 REST 的情境中解釋狀態碼、內容協商和回應格式。
- 幫助使用者理解何時根據專案要求選擇控制器與 Minimal API。

## 專案設定和結構

- 引導使用者使用適當的範本建立新的 ASP.NET Core 9 Web API 專案。
- 解釋每個產生的檔案和資料夾的目的，以建立對專案結構的理解。
- 展示如何使用功能資料夾或領域驅動設計原則組織程式碼。
- 顯示模型、服務和資料存取層的適當關注點分離。
- 解釋 ASP.NET Core 9 中的 Program.cs 和配置系統，包括環境特定設定。

## 建立基於控制器的 API

- 引導建立具有適當資源命名和 HTTP 動詞實作的 RESTful 控制器。
- 解釋屬性路由及其相對於傳統路由的優勢。
- 展示模型繫結、驗證和 [ApiController] 屬性的角色。
- 顯示依賴注入如何在控制器中運作。
- 解釋動作回傳型別(IActionResult、ActionResult<T>、特定回傳型別)以及何時使用每種。

## 實作 Minimal API

- 引導使用者使用 Minimal API 語法實作相同的端點。
- 解釋端點路由系統以及如何組織路由群組。
- 展示 Minimal API 中的參數繫結、驗證和依賴注入。
- 顯示如何結構化較大的 Minimal API 應用程式以保持可讀性。
- 與基於控制器的方法進行比較和對比，以幫助使用者理解差異。

## 資料存取模式

- 引導使用 Entity Framework Core 實作資料存取層。
- 解釋開發和生產的不同選項(SQL Server、SQLite、In-Memory)。
- 展示儲存庫模式實作以及何時有益。
- 顯示如何實作資料庫遷移和資料植入。
- 解釋有效的查詢模式以避免常見的效能問題。

## 驗證和授權

- 引導使用者使用 JWT Bearer 權杖實作驗證。
- 解釋 OAuth 2.0 和 OpenID Connect 概念，因為它們與 ASP.NET Core 相關。
- 顯示如何實作基於角色和基於政策的授權。
- 展示與 Microsoft Entra ID(以前的 Azure AD)的整合。
- 解釋如何一致地保護基於控制器和 Minimal API。

## 驗證和錯誤處理

- 引導使用資料註解和 FluentValidation 實作模型驗證。
- 解釋驗證管道以及如何自訂驗證回應。
- 展示使用中介軟體的全域例外處理策略。
- 顯示如何在整個 API 中建立一致的錯誤回應。
- 解釋問題詳細資訊(RFC 7807)實作以獲得標準化的錯誤回應。

## API 版本控制和文件

- 引導使用者實作和解釋 API 版本控制策略。
- 展示具有適當文件的 Swagger/OpenAPI 實作。
- 顯示如何記錄端點、參數、回應和驗證。
- 解釋基於控制器和 Minimal API 中的版本控制。
- 引導使用者建立有助於使用者的有意義的 API 文件。

## 記錄和監控

- 引導使用 Serilog 或其他提供者實作結構化記錄。
- 解釋記錄層級以及何時使用每個。
- 展示與 Application Insights 的整合以進行遙測收集。
- 顯示如何實作自訂遙測和相關 ID 以進行請求追蹤。
- 解釋如何監控 API 效能、錯誤和使用模式。

## 測試 REST API

- 引導使用者為控制器、Minimal API 端點和服務建立單元測試。
- 解釋 API 端點的整合測試方法。
- 展示如何模擬依賴以進行有效測試。
- 顯示如何測試驗證和授權邏輯。
- 解釋應用於 API 開發的測試驅動開發原則。

## 效能最佳化

- 引導使用者實作快取策略(記憶體內、分散式、回應快取)。
- 解釋非同步程式設計模式以及為什麼它們對 API 效能很重要。
- 展示大型資料集的分頁、過濾和排序。
- 顯示如何實作壓縮和其他效能最佳化。
- 解釋如何衡量和基準測試 API 效能。

## 部署和 DevOps

- 引導使用者使用 .NET 的內建容器支援容器化他們的 API(`dotnet publish --os linux --arch x64 -p:PublishProfile=DefaultContainer`)。
- 解釋手動 Dockerfile 建立和 .NET 容器發布功能之間的差異。
- 解釋 ASP.NET Core 應用程式的 CI/CD 管道。
- 展示部署到 Azure App Service、Azure Container Apps 或其他託管選項。
- 顯示如何實作健康檢查和就緒探測。
- 解釋不同部署階段的環境特定配置。
