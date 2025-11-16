---
mode: 'agent'
description: '產生具有工具、資源和適當配置的完整 TypeScript MCP 伺服器專案'
---

# 產生 TypeScript MCP 伺服器

使用以下規格建立完整的 Model Context Protocol (MCP) 伺服器（TypeScript 版）：

## 需求

1. **專案結構**：建立具有適當目錄結構的新 TypeScript/Node.js 專案
2. **NPM 套件**：包含 @modelcontextprotocol/sdk、zod@3，以及 express（用於 HTTP）或 stdio 支援
3. **TypeScript 配置**：具有 ES 模組支援的適當 tsconfig.json
4. **伺服器類型**：在 HTTP（使用 Streamable HTTP 傳輸）或基於 stdio 的伺服器之間選擇
5. **工具**：建立至少一個具有適當結構描述驗證的有用工具
6. **錯誤處理**：包含全面的錯誤處理和驗證

## 實作詳細資訊

### 專案設定
- 使用 `npm init` 初始化並建立 package.json
- 安裝相依性：`@modelcontextprotocol/sdk`、`zod@3` 和特定於傳輸的套件
- 使用 ES 模組配置 TypeScript：在 package.json 中加入 `"type": "module"`
- 新增開發相依性：`tsx` 或 `ts-node` 用於開發
- 建立適當的 .gitignore 檔案

### 伺服器配置
- 使用 `McpServer` 類別進行高階實作
- 設定伺服器名稱和版本
- 選擇適當的傳輸（StreamableHTTPServerTransport 或 StdioServerTransport）
- 針對 HTTP：使用適當的中介軟體和錯誤處理設定 Express
- 針對 stdio：直接使用 StdioServerTransport

### 工具實作
- 使用具有描述性名稱的 `registerTool()` 方法
- 使用 zod 定義結構描述以進行輸入和輸出驗證
- 提供清晰的 `title` 和 `description` 欄位
- 在結果中傳回 `content` 和 `structuredContent`
- 使用 try-catch 區塊實作適當的錯誤處理
- 適當時支援非同步操作

### 資源/提示設定（選用）
- 使用具有 ResourceTemplate 的 `registerResource()` 新增資源，用於動態 URI
- 使用具有引數結構描述的 `registerPrompt()` 新增提示
- 考慮新增完成支援以獲得更好的 UX

### 程式碼品質
- 使用 TypeScript 以獲得類型安全
- 始終遵循 async/await 模式
- 在傳輸關閉事件上實作適當的清理
- 使用環境變數進行配置
- 為複雜邏輯新增內嵌註解
- 使用清晰的關注點分離來組織程式碼

## 要考慮的範例工具類型
- 資料處理和轉換
- 外部 API 整合
- 檔案系統操作（讀取、搜尋、分析）
- 資料庫查詢
- 文字分析或摘要（含抽樣）
- 系統資訊擷取

## 配置選項
- **針對 HTTP 伺服器**：
  - 透過環境變數配置連接埠
  - 瀏覽器用戶端的 CORS 設定
  - 工作階段管理（無狀態對有狀態）
  - 本機伺服器的 DNS 重新繫結保護

- **針對 stdio 伺服器**：
  - 適當的 stdin/stdout 處理
  - 基於環境的配置
  - 程序生命週期管理

## 測試指導
- 說明如何執行伺服器（`npm start` 或 `npx tsx server.ts`）
- 提供 MCP Inspector 指令：`npx @modelcontextprotocol/inspector`
- 針對 HTTP 伺服器，包含連線 URL：`http://localhost:PORT/mcp`
- 包含範例工具呼叫
- 新增常見問題的疑難排解提示

## 要考慮的其他功能
- 用於 LLM 驅動工具的抽樣支援
- 互動式工作流程的使用者輸入引出
- 具有啟用/停用功能的動態工具註冊
- 大量更新的通知去抖動
- 用於高效資料參考的資源連結

產生具有全面文件、類型安全和錯誤處理的完整、可供生產使用的 MCP 伺服器。
