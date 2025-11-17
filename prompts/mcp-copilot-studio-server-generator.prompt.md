---
description: Generate a complete MCP server implementation optimized for Copilot Studio integration with proper schema constraints and streamable HTTP support
mode: agent
---

# Power Platform MCP 連接器產生器

產生完整的 Power Platform 自訂連接器，整合 Model Context Protocol (MCP) 以供 Microsoft Copilot Studio 使用。此提示會遵循 Power Platform 連接器標準建立所有必要文件，並提供 MCP 串流 HTTP 支援。

## 指示說明

建立完整的 MCP 伺服器實作，滿足以下要求：

1. **使用 Copilot Studio MCP 模式：**
   - 實作 `x-ms-agentic-protocol: mcp-streamable-1.0`
   - 支援 JSON-RPC 2.0 通訊協定
   - 在 `/mcp` 提供可串流的 HTTP 端點
   - 遵循 Power Platform 連接器結構

2. **架構相容要求：**
   - **工具輸入/輸出中不使用參考型別**（由 Copilot Studio 篩選）
   - **只使用單一型別值**（不使用多重型別陣列）
   - **避免使用列舉輸入**（會被解釋為字串，而不是列舉）
   - 使用原始型別：string、number、integer、boolean、array、object
   - 確保所有端點都回傳完整的 URI

3. **要包含的 MCP 元件：**
   - **工具**：供語言模型呼叫的函式（✅ Copilot Studio 支援）
   - **資源**：來自工具的類檔案資料輸出（✅ Copilot Studio 支援 - 必須為工具輸出才能存取）
   - **提示**：特定工作的預定義範本（❌ Copilot Studio 尚未支援）

4. **實作結構：**
   ```
   /apiDefinition.swagger.json  (Power Platform connector schema)
   /apiProperties.json         (Connector metadata and configuration)
   /script.csx                 (Custom code transformations and logic)
   /server/                    (MCP server implementation)
   /tools/                     (Individual MCP tools)
   /resources/                 (MCP resource handlers)
   ```

## 內容變數

- **伺服器用途**：[描述 MCP 伺服器應該完成的工作]
- **需要的工具**：[要實作的特定工具列表]
- **資源**：[要提供的資源型別]
- **身份驗證**：[驗證方式：none、api-key、oauth2]
- **主機環境**：[Azure Function、Express.js、FastAPI 等]
- **目標 API**：[要整合的外部 API]

## 預期輸出

產生下列項目：

1. **apiDefinition.swagger.json** 包含：
   - 正確的 `x-ms-agentic-protocol: mcp-streamable-1.0`
   - POST `/mcp` 的 MCP 端點
   - 相容的架構定義（無參考型別）
   - McpResponse 和 McpErrorResponse 定義

2. **apiProperties.json** 包含：
   - 連接器中繼資料和品牌
   - 身份驗證組態
   - 必要時的原則範本

3. **script.csx** 包含：
   - 用於請求/回應轉換的自訂 C# 程式碼
   - MCP JSON-RPC 訊息處理邏輯
   - 資料驗證和處理函式
   - 錯誤處理和記錄功能

4. **MCP 伺服器程式碼** 包含：
   - JSON-RPC 2.0 請求處理程式
   - 工具註冊和執行
   - 資源管理（作為工具輸出）
   - 正確的錯誤處理
   - Copilot Studio 相容性檢查

5. **個別工具** 必須：
   - 只接受原始型別輸入
   - 回傳結構化輸出
   - 必要時在輸出中包含資源
   - 為 Copilot Studio 提供清晰的描述

6. **部署組態** 包含：
   - Power Platform 環境
   - Copilot Studio 代理程式整合
   - 測試和驗證

## 驗證檢查清單

確保產生的程式碼：
- [ ] 架構中沒有參考型別
- [ ] 所有型別欄位都是單一型別
- [ ] 透過字串進行列舉處理並驗證
- [ ] 資源可透過工具輸出取得
- [ ] 完整的 URI 端點
- [ ] 符合 JSON-RPC 2.0
- [ ] 正確的 x-ms-agentic-protocol 標頭
- [ ] McpResponse/McpErrorResponse 架構
- [ ] 為 Copilot Studio 提供清晰的工具描述
- [ ] 相容於生成式協調

## 使用範例

```yaml
Server Purpose: Customer data management and analysis
Tools Needed:
  - searchCustomers
  - getCustomerDetails
  - analyzeCustomerTrends
Resources:
  - Customer profiles
  - Analysis reports
Authentication: oauth2
Host Environment: Azure Function
Target APIs: CRM System REST API
```