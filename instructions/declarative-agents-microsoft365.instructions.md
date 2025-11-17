---
description: Microsoft 365 Copilot 宣告式代理程式的完整開發指南，包含 schema v1.5、TypeSpec 整合及 Microsoft 365 Agents Toolkit 工作流程
applyTo: "**.json, **.ts, **.tsp, **manifest.json, **agent.json, **declarative-agent.json"
---

# Microsoft 365 宣告式代理程式開發指南

## 概述

Microsoft 365 Copilot 宣告式代理程式是功能強大的自訂 AI 助理，可以透過專門的功能、企業資料存取和自訂行為來擴充 Microsoft 365 Copilot。本指南提供完整的開發實務，協助建立可用於生產環境的代理程式，使用最新的 v1.5 JSON schema 規格並完整整合 Microsoft 365 Agents Toolkit。

## Schema 規格 v1.5

### 核心屬性

```json
{
  "$schema": "https://developer.microsoft.com/json-schemas/copilot/declarative-agent/v1.5/schema.json",
  "version": "v1.5",
  "name": "string (最多 100 字元)",
  "description": "string (最多 1000 字元)",
  "instructions": "string (最多 8000 字元)",
  "capabilities": ["array (最多 5 個項目)"],
  "conversation_starters": ["array (最多 4 個項目，選填)"]
}
```

### 字元限制與約束
- **Name**：最多 100 字元，必填
- **Description**：最多 1000 字元，必填
- **Instructions**：最多 8000 字元，必填
- **Capabilities**：最多 5 個項目，至少 1 個項目
- **Conversation Starters**：最多 4 個項目，選填

## 可用功能

### 核心功能
1. **WebSearch**：網際網路搜尋和即時資訊存取
2. **OneDriveAndSharePoint**：檔案存取、文件搜尋、內容管理
3. **GraphConnectors**：從第三方系統整合企業資料
4. **MicrosoftGraph**：存取 Microsoft 365 服務和資料

### 溝通與協作
5. **TeamsAndOutlook**：Teams 聊天、會議、電子郵件整合
6. **CopilotForMicrosoft365**：進階 Copilot 功能和工作流程

### 商務應用程式
7. **PowerPlatform**：Power Apps、Power Automate、Power BI 整合
8. **BusinessDataProcessing**：進階資料分析和處理
9. **WordAndExcel**：文件建立、編輯、分析
10. **EnterpriseApplications**：第三方商務系統整合
11. **CustomConnectors**：自訂 API 和服務整合

## Microsoft 365 Agents Toolkit 整合

### VS Code 擴充功能設定
```bash
# 安裝 Microsoft 365 Agents Toolkit
# 擴充功能 ID: teamsdevapp.ms-teams-vscode-extension
```

### TypeSpec 開發工作流程

#### 1. 現代化代理程式定義
```typespec
import "@typespec/json-schema";

using TypeSpec.JsonSchema;

@jsonSchema("/schemas/declarative-agent/v1.5/schema.json")
namespace DeclarativeAgent;

/** Microsoft 365 宣告式代理程式 */
model Agent {
  /** Schema 版本 */
  @minLength(1)
  $schema: "https://developer.microsoft.com/json-schemas/copilot/declarative-agent/v1.5/schema.json";

  /** 代理程式版本 */
  version: "v1.5";

  /** 代理程式名稱 (最多 100 字元) */
  @maxLength(100)
  @minLength(1)
  name: string;

  /** 代理程式描述 (最多 1000 字元) */
  @maxLength(1000)
  @minLength(1)
  description: string;

  /** 代理程式指示 (最多 8000 字元) */
  @maxLength(8000)
  @minLength(1)
  instructions: string;

  /** 代理程式功能 (1-5 個項目) */
  @minItems(1)
  @maxItems(5)
  capabilities: AgentCapability[];

  /** 對話起始語 (最多 4 個項目) */
  @maxItems(4)
  conversation_starters?: ConversationStarter[];
}

/** 可用的代理程式功能 */
union AgentCapability {
  "WebSearch",
  "OneDriveAndSharePoint",
  "GraphConnectors",
  "MicrosoftGraph",
  "TeamsAndOutlook",
  "PowerPlatform",
  "BusinessDataProcessing",
  "WordAndExcel",
  "CopilotForMicrosoft365",
  "EnterpriseApplications",
  "CustomConnectors"
}

/** 對話起始語定義 */
model ConversationStarter {
  /** 起始語文字 (最多 100 字元) */
  @maxLength(100)
  @minLength(1)
  text: string;
}
```

#### 2. 編譯為 JSON
```bash
# 將 TypeSpec 編譯為 JSON manifest
tsp compile agent.tsp --emit=@typespec/json-schema
```

### 環境設定

#### 開發環境
```json
{
  "name": "${DEV_AGENT_NAME}",
  "description": "開發版本：${AGENT_DESCRIPTION}",
  "instructions": "${AGENT_INSTRUCTIONS}",
  "capabilities": ["${REQUIRED_CAPABILITIES}"]
}
```

#### 生產環境
```json
{
  "name": "${PROD_AGENT_NAME}",
  "description": "${AGENT_DESCRIPTION}",
  "instructions": "${AGENT_INSTRUCTIONS}",
  "capabilities": ["${PRODUCTION_CAPABILITIES}"]
}
```

## 開發最佳實務

### 1. Schema 驗證
```typescript
// 針對 v1.5 schema 進行驗證
const schema = await fetch('https://developer.microsoft.com/json-schemas/copilot/declarative-agent/v1.5/schema.json');
const validator = new JSONSchema(schema);
const isValid = validator.validate(agentManifest);
```

### 2. 字元限制管理
```typescript
// 驗證輔助函式
function validateName(name: string): boolean {
  return name.length > 0 && name.length <= 100;
}

function validateDescription(description: string): boolean {
  return description.length > 0 && description.length <= 1000;
}

function validateInstructions(instructions: string): boolean {
  return instructions.length > 0 && instructions.length <= 8000;
}
```

### 3. 功能選擇策略
- **從簡單開始**：從 1-2 個核心功能開始
- **逐步增加**：根據使用者回饋增加功能
- **效能測試**：徹底測試每個功能組合
- **企業就緒**：考慮合規性和安全性影響

## Agents Playground 測試

### 本地測試設定
```bash
# 啟動 Agents Playground
npm install -g @microsoft/agents-playground
agents-playground start --manifest=./agent.json
```

### 測試情境
1. **功能驗證**：測試每個已宣告的功能
2. **對話流程**：驗證對話起始語
3. **錯誤處理**：測試無效輸入和邊緣情況
4. **效能**：測量回應時間和可靠性

## 部署與生命週期管理

### 1. 開發生命週期
```mermaid
graph LR
    A[TypeSpec 定義] --> B[JSON 編譯]
    B --> C[本地測試]
    C --> D[驗證]
    D --> E[預備環境部署]
    E --> F[正式環境發布]
```

### 2. 版本管理
```json
{
  "name": "MyAgent v1.2.0",
  "description": "具備增強功能的生產代理程式",
  "version": "v1.5",
  "metadata": {
    "version": "1.2.0",
    "build": "20241208.1",
    "environment": "production"
  }
}
```

### 3. 環境升級
- **開發環境**：完整除錯、詳細記錄
- **預備環境**：類生產環境測試、效能監控
- **正式環境**：最佳化效能、最少記錄

## 進階功能

### 行為覆寫
```json
{
  "instructions": "您是專業的財務分析代理程式。請務必為財務建議提供免責聲明。",
  "behavior_overrides": {
    "response_tone": "professional",
    "max_response_length": 2000,
    "citation_requirements": true
  }
}
```

### 在地化支援
```json
{
  "name": {
    "en-US": "Financial Assistant",
    "es-ES": "Asistente Financiero",
    "fr-FR": "Assistant Financier"
  },
  "description": {
    "en-US": "Provides financial analysis and insights",
    "es-ES": "Proporciona análisis e insights financieros",
    "fr-FR": "Fournit des analyses et insights financiers"
  }
}
```

## 監控與分析

### 效能指標
- 每個功能的回應時間
- 使用者對對話起始語的互動程度
- 錯誤率和失敗模式
- 功能使用統計

### 記錄策略
```typescript
// 代理程式互動的結構化記錄
const log = {
  timestamp: new Date().toISOString(),
  agentName: "MyAgent",
  version: "1.2.0",
  userId: "user123",
  capability: "WebSearch",
  responseTime: 1250,
  success: true
};
```

## 安全性與合規性

### 資料隱私
- 為敏感資訊實作適當的資料處理
- 確保符合 GDPR、CCPA 和組織政策
- 為企業功能使用適當的存取控制

### 安全性考量
- 驗證所有輸入和輸出
- 實作速率限制和濫用防護
- 監控可疑活動模式
- 定期進行安全性稽核和更新

## 疑難排解

### 常見問題
1. **Schema 驗證錯誤**：檢查字元限制和必填欄位
2. **功能衝突**：確認功能組合是否受支援
3. **效能問題**：監控回應時間並最佳化指示
4. **部署失敗**：驗證環境設定和權限

### 除錯工具
- TypeSpec 編譯器診斷
- Agents Playground 除錯
- Microsoft 365 Agents Toolkit 記錄
- Schema 驗證工具

本完整指南確保能建立穩健、可擴充且易於維護的 Microsoft 365 Copilot 宣告式代理程式，並完整整合 TypeSpec 和 Microsoft 365 Agents Toolkit。