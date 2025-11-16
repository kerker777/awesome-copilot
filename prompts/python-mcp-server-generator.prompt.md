---
mode: 'agent'
description: '產生具有工具、資源和適當配置的完整 Python MCP 伺服器專案'
---

# 產生 Python MCP 伺服器

使用以下規格建立完整的 Model Context Protocol (MCP) 伺服器（Python 版）：

## 需求

1. **專案結構**：使用 uv 建立具有適當結構的新 Python 專案
2. **相依性**：使用 uv 包含 mcp[cli] 套件
3. **傳輸類型**：在 stdio（用於本機）或 streamable-http（用於遠端）之間選擇
4. **工具**：建立至少一個具有適當類型提示的有用工具
5. **錯誤處理**：包含全面的錯誤處理和驗證

## 實作詳細資訊

### 專案設定
- 使用 `uv init project-name` 初始化
- 新增 MCP SDK：`uv add "mcp[cli]"`
- 建立主要伺服器檔案（例如，`server.py`）
- 為 Python 專案新增 `.gitignore`
- 配置為使用 `if __name__ == "__main__"` 直接執行

### 伺服器配置
- 從 `mcp.server.fastmcp` 使用 `FastMCP` 類別
- 設定伺服器名稱和選用指示
- 選擇傳輸：stdio（預設）或 streamable-http
- 針對 HTTP：選擇性配置主機、連接埠和無狀態模式

### 工具實作
- 在函式上使用 `@mcp.tool()` 裝飾器
- 始終包含類型提示 - 它們會自動產生結構描述
- 撰寫清晰的文件字串 - 它們會成為工具描述
- 對於結構化輸出使用 Pydantic 模型或 TypedDicts
- 支援 I/O 繫結任務的非同步操作
- 包含適當的錯誤處理

### 資源/提示設定（選用）
- 使用 `@mcp.resource()` 裝飾器新增資源
- 對於動態資源使用 URI 範本：`"resource://{param}"`
- 使用 `@mcp.prompt()` 裝飾器新增提示
- 從提示傳回字串或訊息清單

### 程式碼品質
- 對所有函式參數和傳回值使用類型提示
- 為工具、資源和提示撰寫文件字串
- 遵循 PEP 8 樣式指南
- 對非同步操作使用 async/await
- 實作資源清理的內容管理器
- 為複雜邏輯新增內嵌註解

## 要考慮的範例工具類型
- 資料處理和轉換
- 檔案系統操作（讀取、分析、搜尋）
- 外部 API 整合
- 資料庫查詢
- 文字分析或產生（含抽樣）
- 系統資訊擷取
- 數學或科學計算

## 配置選項
- **針對 stdio 伺服器**：
  - 簡單的直接執行
  - 使用 `uv run mcp dev server.py` 測試
  - 安裝到 Claude：`uv run mcp install server.py`

- **針對 HTTP 伺服器**：
  - 透過環境變數配置連接埠
  - 可擴展性的無狀態模式：`stateless_http=True`
  - JSON 回應模式：`json_response=True`
  - 瀏覽器用戶端的 CORS 配置
  - 掛載到現有的 ASGI 伺服器（Starlette/FastAPI）

## 測試指導
- 說明如何執行伺服器：
  - stdio：`python server.py` 或 `uv run server.py`
  - HTTP：`python server.py` 然後連線到 `http://localhost:PORT/mcp`
- 使用 MCP Inspector 測試：`uv run mcp dev server.py`
- 安裝到 Claude Desktop：`uv run mcp install server.py`
- 包含範例工具呼叫
- 新增疑難排解提示

## 要考慮的其他功能
- 使用 Context 進行記錄、進度和通知
- 用於 AI 驅動工具的 LLM 抽樣
- 互動式工作流程的使用者輸入引出
- 共用資源（資料庫、連線）的生命週期管理
- 使用 Pydantic 模型的結構化輸出
- UI 顯示的圖示
- 使用 Image 類別處理影像
- 更好 UX 的完成支援

## 最佳實踐
- 到處使用類型提示 - 它們不是選用的
- 盡可能傳回結構化資料
- 記錄到 stderr（或使用 Context 記錄）以避免 stdout 污染
- 適當清理資源
- 儘早驗證輸入
- 提供清晰的錯誤訊息
- 在 LLM 整合之前獨立測試工具

產生具有類型安全、適當錯誤處理和全面文件的完整、可供生產使用的 MCP 伺服器。
