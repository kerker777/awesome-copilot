---
description: 產生完整的 Power Platform 自訂連接器，具有 MCP 整合功能用於 Copilot Studio - 包括結構描述產生、疑難排解和驗證
mode: agent
---

# Power Platform MCP 連接器套件

產生具有 Model Context Protocol 整合的全面 Power Platform 自訂連接器實作，用於 Microsoft Copilot Studio。

## Copilot Studio 中的 MCP 功能

**目前支援：**
- ✅ **工具**：LLM 可以呼叫的函式（需要使用者核准）
- ✅ **資源**：代理可以讀取的類檔案資料（必須是工具輸出）

**尚未支援：**
- ❌ **提示**：預先撰寫的範本（準備未來支援）

## 連接器產生

建立具有以下功能的完整 Power Platform 連接器：

**核心檔案：**
- 具有 `x-ms-agentic-protocol: mcp-streamable-1.0` 的 `apiDefinition.swagger.json`
- 具有連接器中繼資料和驗證的 `apiProperties.json`
- 具有用於 MCP JSON-RPC 處理的自訂 C# 轉換的 `script.csx`
- 具有連接器文件的 `readme.md`

**MCP 整合：**
- 用於 JSON-RPC 2.0 通訊的 POST `/mcp` 端點
- McpResponse 和 McpErrorResponse 結構描述定義
- Copilot Studio 限制合規性（無參考類型、單一類型）
- 作為工具輸出的資源整合（支援資源和工具；尚未支援提示）

## 結構描述驗證和疑難排解

**驗證 Copilot Studio 合規性的結構描述：**
- ✅ 工具輸入/輸出中無參考類型（`$ref`）
- ✅ 僅單一類型值（非 `["string", "number"]`）
- ✅ 基本類型：string、number、integer、boolean、array、object
- ✅ 資源作為工具輸出，而非單獨實體
- ✅ 所有端點的完整 URI

**常見問題和修正：**
- 工具已篩選 → 移除參考類型，使用基本類型
- 類型錯誤 → 具有驗證邏輯的單一類型
- 資源不可用 → 包含在工具輸出中
- 連線失敗 → 驗證 `x-ms-agentic-protocol` 標頭

## 背景變數

- **連接器名稱**：[連接器的顯示名稱]
- **伺服器目的**：[MCP 伺服器應完成的任務]
- **所需工具**：[要實作的 MCP 工具清單]
- **資源**：[要提供的資源類型]
- **驗證**：[none、api-key、oauth2、basic]
- **主機環境**：[Azure Function、Express.js 等]
- **目標 API**：[要整合的外部 API]

## 產生模式

### 模式 1：完整的新連接器
從頭開始產生新 Power Platform MCP 連接器的所有檔案，包括 CLI 驗證設定。

### 模式 2：結構描述驗證
使用 paconn 和驗證工具分析並修正 Copilot Studio 合規性的現有結構描述。

### 模式 3：整合疑難排解
使用 CLI 偵錯工具診斷並解決 Copilot Studio 的 MCP 整合問題。

### 模式 4：混合連接器
透過適當的驗證工作流程，將 MCP 功能新增至現有的 Power Platform 連接器。

### 模式 5：認證準備
準備連接器以提交 Microsoft 認證，包含完整的中繼資料和驗證合規性。

### 模式 6：OAuth 安全強化
實作 OAuth 2.0 驗證，並透過 MCP 安全最佳實踐和進階權杖驗證進行增強。

## 預期輸出

**1. apiDefinition.swagger.json**
- 具有 Microsoft 擴充功能的 Swagger 2.0 格式
- MCP 端點：具有適當協定標頭的 `POST /mcp`
- 合規的結構描述定義（僅基本類型）
- McpResponse/McpErrorResponse 定義

**2. apiProperties.json**
- 連接器中繼資料和品牌（需要 `iconBrandColor`）
- 驗證配置
- MCP 轉換的原則範本

**3. script.csx**
- JSON-RPC 2.0 訊息處理
- 要求/回應轉換
- MCP 協定合規性邏輯
- 錯誤處理和驗證

**4. 實作指導**
- 工具註冊和執行模式
- 資源管理策略
- Copilot Studio 整合步驟
- 測試和驗證程序

## 驗證檢查清單

### 技術合規性
- [ ] MCP 端點中的 `x-ms-agentic-protocol: mcp-streamable-1.0`
- [ ] 任何結構描述定義中無參考類型
- [ ] 所有類型欄位都是單一類型（非陣列）
- [ ] 資源包含為工具輸出
- [ ] script.csx 中的 JSON-RPC 2.0 合規性
- [ ] 全面的完整 URI 端點
- [ ] Copilot Studio 代理的清晰說明
- [ ] 驗證已適當配置
- [ ] MCP 轉換的原則範本
- [ ] 生成式協調相容性

### CLI 驗證
- [ ] **paconn validate**：`paconn validate --api-def apiDefinition.swagger.json` 通過，無錯誤
- [ ] **pac CLI 就緒**：可以使用 `pac connector create/update` 建立/更新連接器
- [ ] **指令碼驗證**：script.csx 在 pac CLI 上傳期間通過自動驗證
- [ ] **套件驗證**：`ConnectorPackageValidator.ps1` 成功執行

### OAuth 和安全需求
- [ ] **OAuth 2.0 增強**：標準 OAuth 2.0 與 MCP 安全最佳實踐實作
- [ ] **權杖驗證**：實作權杖受眾驗證以防止穿透攻擊
- [ ] **自訂安全邏輯**：script.csx 中的增強驗證以符合 MCP 合規性
- [ ] **狀態參數保護**：安全的狀態參數以防止 CSRF
- [ ] **HTTPS 強制**：所有生產端點僅使用 HTTPS
- [ ] **MCP 安全實踐**：在 OAuth 2.0 內實作混淆代理攻擊防護

### 認證需求
- [ ] **完整的中繼資料**：具有產品和服務資訊的 settings.json
- [ ] **圖示合規性**：PNG 格式、230x230 或 500x500 尺寸
- [ ] **文件**：具有全面範例的認證就緒 readme
- [ ] **安全合規性**：透過 MCP 安全實踐增強的 OAuth 2.0、隱私政策
- [ ] **驗證流程**：OAuth 2.0 與自訂安全驗證適當配置

## 使用範例

```yaml
模式：完整的新連接器
連接器名稱：Customer Analytics MCP
伺服器目的：客戶資料分析和洞察
所需工具：
  - searchCustomers：依條件尋找客戶
  - getCustomerProfile：擷取詳細的客戶資料
  - analyzeCustomerTrends：產生趨勢分析
資源：
  - 客戶設定檔（JSON 資料）
  - 分析報告（結構化資料）
驗證：oauth2
主機環境：Azure Function
目標 API：CRM REST API
```
