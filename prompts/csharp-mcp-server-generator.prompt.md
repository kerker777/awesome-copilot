---
mode: 'agent'
description: '產生完整的 C# MCP 伺服器專案，包含工具、提示詞和適當的設定'
---

# 產生 C# MCP 伺服器

建立具有以下規格的完整 Model Context Protocol (MCP) 伺服器（使用 C#）：

## 需求

1. **專案結構**：建立具有適當目錄結構的新 C# 主控台應用程式
2. **NuGet 套件**：包含 ModelContextProtocol（預發行版本）和 Microsoft.Extensions.Hosting
3. **日誌設定**：將所有日誌設定為輸出到 stderr，以避免干擾 stdio 傳輸
4. **伺服器設定**：使用 Host 建構器模式搭配適當的 DI 設定
5. **工具**：建立至少一個有用的工具，並加上適當的屬性和描述
6. **錯誤處理**：包含適當的錯誤處理和驗證

## 實作細節

### 基本專案設定
- 使用 .NET 8.0 或更新版本
- 建立主控台應用程式
- 使用 --prerelease 旗標加入必要的 NuGet 套件
- 將日誌設定為輸出到 stderr

### 伺服器設定
- 使用 `Host.CreateApplicationBuilder` 進行 DI 和生命週期管理
- 使用 stdio 傳輸設定 `AddMcpServer()`
- 使用 `WithToolsFromAssembly()` 進行自動工具探索
- 確保伺服器使用 `RunAsync()` 執行

### 工具實作
- 在工具類別上使用 `[McpServerToolType]` 屬性
- 在工具方法上使用 `[McpServerTool]` 屬性
- 將 `[Description]` 屬性加入工具和參數
- 適當的地方支援非同步操作
- 包含適當的參數驗證

### 程式碼品質
- 遵循 C# 命名慣例
- 包含 XML 文件註解
- 使用可為 null 的參考類型
- 使用 McpProtocolException 實作適當的錯誤處理
- 使用結構化日誌進行偵錯

## 考慮的工具類型範例
- 檔案操作（讀取、寫入、搜尋）
- 資料處理（轉換、驗證、分析）
- 外部 API 整合（HTTP 請求）
- 系統操作（執行命令、檢查狀態）
- 資料庫操作（查詢、更新）

## 測試指南
- 說明如何執行伺服器
- 提供使用 MCP 用戶端測試的範例命令
- 包含疑難排解提示

產生具有完整文件和錯誤處理的完整、可用於生產環境的 MCP 伺服器。
