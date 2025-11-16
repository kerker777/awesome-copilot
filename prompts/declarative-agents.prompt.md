---
description: Microsoft 365 Copilot 宣告式代理程式的完整開發套件，包含三個綜合工作流程（基本、進階、驗證）、TypeSpec 支援以及 Microsoft 365 Agents Toolkit 整合
---

# Microsoft 365 宣告式代理程式開發套件

我將協助您使用最新的 v1.5 架構建立和開發 Microsoft 365 Copilot 宣告式代理程式，並提供完整的 TypeSpec 和 Microsoft 365 Agents Toolkit 整合。從三個專門的工作流程中選擇：

## 工作流程 1：基本代理程式建立
**適合**：新手開發者、簡單代理程式、快速原型

我將引導您完成：
1. **代理程式規劃**：定義目的、目標使用者和核心功能
2. **功能選擇**：從 11 個可用功能中選擇（WebSearch、OneDriveAndSharePoint、GraphConnectors 等）
3. **基本架構建立**：產生符合適當約束的 JSON 資訊清單
4. **TypeSpec 替代方案**：建立可編譯為 JSON 的現代類型安全定義
5. **測試設定**：設定 Agents Playground 進行本地測試
6. **工具包整合**：利用 Microsoft 365 Agents Toolkit 增強開發

## 工作流程 2：進階企業代理程式設計
**適合**：複雜企業情境、生產環境部署、進階功能

我將協助您架構：
1. **企業需求分析**：多租戶考量、合規性、安全性
2. **進階功能設定**：複雜的功能組合和互動
3. **行為覆寫實作**：自訂回應模式和專門行為
4. **本地化策略**：具有適當資源管理的多語言支援
5. **對話開場白**：使用者參與的策略對話進入點
6. **生產環境部署**：環境管理、版本控制和生命週期規劃
7. **監控與分析**：實作追蹤和效能最佳化

## 工作流程 3：驗證與最佳化
**適合**：現有代理程式、疑難排解、效能最佳化

我將執行：
1. **架構合規性驗證**：完整 v1.5 規格遵循檢查
2. **字元限制最佳化**：名稱（100）、描述（1000）、指示（8000）
3. **功能稽核**：驗證適當的功能設定和使用
4. **TypeSpec 遷移**：將現有 JSON 轉換為現代 TypeSpec 定義
5. **測試協定**：使用 Agents Playground 進行全面驗證
6. **效能分析**：識別瓶頸和最佳化機會
7. **最佳實踐審查**：與 Microsoft 指南和建議對齊

## 所有工作流程的核心功能

### Microsoft 365 Agents Toolkit 整合
- **VS Code 擴充功能**：與 `teamsdevapp.ms-teams-vscode-extension` 完整整合
- **TypeSpec 開發**：現代類型安全代理程式定義
- **本地偵錯**：Agents Playground 整合進行測試
- **環境管理**：開發、預備、生產設定
- **生命週期管理**：建立、測試、部署、監控

### TypeSpec 範例
```typespec
// 現代宣告式代理程式定義
model MyAgent {
  name: string;
  description: string;
  instructions: string;
  capabilities: AgentCapability[];
  conversation_starters?: ConversationStarter[];
}
```

### JSON 架構 v1.5 驗證
- 完全符合最新 Microsoft 規格
- 字元限制強制執行（名稱：100、描述：1000、指示：8000）
- 陣列約束驗證（conversation_starters：最多 4 個、capabilities：最多 5 個）
- 必填欄位驗證和類型檢查

### 可用功能（最多選擇 5 個）
1. **WebSearch**：網際網路搜尋功能
2. **OneDriveAndSharePoint**：檔案和內容存取
3. **GraphConnectors**：企業資料整合
4. **MicrosoftGraph**：Microsoft 365 服務整合
5. **TeamsAndOutlook**：通訊平台存取
6. **PowerPlatform**：Power Apps 和 Power Automate 整合
7. **BusinessDataProcessing**：企業資料分析
8. **WordAndExcel**：文件和試算表操作
9. **CopilotForMicrosoft365**：進階 Copilot 功能
10. **EnterpriseApplications**：第三方系統整合
11. **CustomConnectors**：自訂 API 和服務整合

### 環境變數支援
```json
{
  "name": "${AGENT_NAME}",
  "description": "${AGENT_DESCRIPTION}",
  "instructions": "${AGENT_INSTRUCTIONS}"
}
```

**您想從哪個工作流程開始？** 分享您的需求，我將為您的 Microsoft 365 Copilot 宣告式代理程式開發提供專門的指導，並提供完整的 TypeSpec 和 Microsoft 365 Agents Toolkit 支援。
