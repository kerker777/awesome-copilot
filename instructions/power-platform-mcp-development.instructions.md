---
description: 'Power Platform 自訂連接器開發指導，整合模型上下文協議 (MCP) 用於 Microsoft Copilot Studio'
applyTo: '**/*.{json,csx,md}'
---

# Power Platform MCP 自訂連接器開發

## 指導方針

### MCP 協議整合
- 必須實作 JSON-RPC 2.0 標準用於 MCP 通訊
- 使用 `x-ms-agentic-protocol: mcp-streamable-1.0` 標頭以相容於 Copilot Studio
- 結構化端點以支援標準 REST 操作和 MCP 工具呼叫
- 轉換回應以符合 Copilot Studio 限制（無參考類型、僅單一類型）

### 架構設計最佳實務
- 從 JSON 架構中移除 `$ref` 和其他參考類型，因為 Copilot Studio 無法處理
- 在架構定義中使用單一類型而非類型陣列
- 將 `anyOf`/`oneOf` 結構扁平化為單一架構以相容於 Copilot Studio
- 確保所有工具輸入架構都是自包含的，沒有外部參考

### 身份驗證和安全性
- 在 Power Platform 限制內實作 OAuth 2.0 並遵循 MCP 安全最佳實務
- 使用連線參數集以提供靈活的身份驗證組態
- 驗證權杖對象以防止通道攻擊
- 新增 MCP 特定的安全標頭以進行增強驗證
- 支援多種身份驗證方法（OAuth 標準、OAuth 增強、API 金鑰後援）

### 自訂指令碼實作
- 在自訂指令碼（script.csx）中處理 JSON-RPC 轉換
- 實作適當的錯誤處理，使用 JSON-RPC 錯誤回應格式
- 在身份驗證流程中新增權杖驗證和對象檢查
- 轉換 MCP 伺服器回應以相容於 Copilot Studio
- 使用連線參數進行動態安全組態

### Swagger 定義指導方針
- 使用 Swagger 2.0 規格以相容於 Power Platform
- 為每個端點實作適當的 `operationId` 值
- 定義清晰的參數架構並使用適當的類型和說明
- 為所有成功和錯誤情況新增全面的回應架構
- 包含適當的 HTTP 狀態碼和回應標頭

### 資源和工具管理
- 結構化 MCP 資源以便在 Copilot Studio 中作為工具輸出使用
- 確保資源內容的適當 MIME 類型宣告
- 為更好的 Copilot Studio 整合新增對象和優先權註解
- 實作資源轉換以符合 Copilot Studio 需求

### 連線參數組態
- 使用列舉下拉式清單進行 OAuth 版本和安全層級選擇
- 提供清晰的參數說明和限制條件
- 針對不同的部署案例支援多個身份驗證參數集
- 在適當的地方包含驗證規則和預設值
- 透過連線參數值啟用動態組態

### 錯誤處理和日誌記錄
- 實作遵循 JSON-RPC 2.0 錯誤格式的全面錯誤回應
- 為身份驗證、驗證和轉換步驟新增詳細的日誌記錄
- 提供清晰的錯誤訊息以協助疑難排解
- 包含與錯誤條件相符的適當 HTTP 狀態碼

### 測試和驗證
- 使用實際的 MCP 伺服器實作測試連接器
- 驗證架構轉換能夠正確搭配 Copilot Studio 運作
- 驗證所有支援的參數集的身份驗證流程
- 確保針對各種失敗情況的適當錯誤處理
- 測試連線參數組態和動態行為

## 額外指導方針

### Power Platform 認證需求
- 包含全面的文件（readme.md、CUSTOMIZE.md）
- 提供清晰的設定和組態指導
- 說明所有身份驗證選項和安全性考量
- 包含適當的發行者和堆疊擁有者資訊
- 確保符合 Power Platform 連接器認證標準

### MCP 伺服器相容性
- 設計以相容於標準 MCP 伺服器實作
- 支援常見的 MCP 方法，例如 `tools/list`、`tools/call`、`resources/list`
- 適當處理 `mcp-streamable-1.0` 協議的串流回應
- 實作適當的協議協商和功能偵測

### Copilot Studio 整合
- 確保工具定義在 Copilot Studio 的限制內正確運作
- 從 Copilot Studio 介面測試資源存取和工具呼叫
- 驗證轉換的架構在對話中產生預期行為
- 確認與 Copilot Studio 代理程式架構的適當整合
