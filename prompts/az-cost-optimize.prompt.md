---
mode: 'agent'
description: '分析應用程式中使用的 Azure 資源（IaC 檔案和/或目標資源群組中的資源），並優化成本 - 為識別的優化創建 GitHub 議題。'
---

# Azure 成本優化

此工作流程分析基礎設施即程式碼 (IaC) 檔案和 Azure 資源，以生成成本優化建議。它為每個優化機會創建個別的 GitHub 議題，並創建一個 EPIC 議題來協調實作，從而實現成本節省計劃的高效追蹤和執行。

## 先決條件
- 已配置並驗證 Azure MCP 伺服器
- 已配置並驗證 GitHub MCP 伺服器
- 已識別目標 GitHub 儲存庫
- 已部署 Azure 資源（IaC 檔案可選但有幫助）
- 當可用時，優先使用 Azure MCP 工具（`azmcp-*`）而非直接使用 Azure CLI

## 工作流程步驟

### 步驟 1：獲取 Azure 最佳實踐
**行動**：在分析前檢索成本優化最佳實踐
**工具**：Azure MCP 最佳實踐工具
**流程**：
1. **載入最佳實踐**：
   - 執行 `azmcp-bestpractices-get` 以獲取一些最新的 Azure 優化指南。這可能無法涵蓋所有場景，但提供了基礎。
   - 使用這些實踐來指導後續分析和建議
   - 在優化建議中引用最佳實踐，無論是來自 MCP 工具輸出還是一般 Azure 文件

### 步驟 2：探索 Azure 基礎設施
**行動**：動態探索和分析 Azure 資源和配置
**工具**：Azure MCP 工具 + Azure CLI 後備 + 本地檔案系統存取
**流程**：
1. **資源探索**：
   - 執行 `azmcp-subscription-list` 以尋找可用的訂閱
   - 執行 `azmcp-group-list --subscription <subscription-id>` 以尋找資源群組
   - 獲取相關群組中所有資源的清單：
     - 使用 `az resource list --subscription <id> --resource-group <name>`
   - 對於每種資源類型，如果可能優先使用 MCP 工具，然後使用 CLI 後備：
     - `azmcp-cosmos-account-list --subscription <id>` - Cosmos DB 帳戶
     - `azmcp-storage-account-list --subscription <id>` - 儲存體帳戶
     - `azmcp-monitor-workspace-list --subscription <id>` - Log Analytics 工作區
     - `azmcp-keyvault-key-list` - Key Vaults
     - `az webapp list` - Web 應用程式（後備 - 沒有可用的 MCP 工具）
     - `az appservice plan list` - App Service 方案（後備）
     - `az functionapp list` - Function 應用程式（後備）
     - `az sql server list` - SQL 伺服器（後備）
     - `az redis list` - Redis 快取（後備）
     - ... 以及其他資源類型

2. **IaC 偵測**：
   - 使用 `file_search` 掃描 IaC 檔案："**/*.bicep"、"**/*.tf"、"**/main.json"、"**/*template*.json"
   - 解析資源定義以了解預期配置
   - 與發現的資源進行比較以識別差異
   - 注意 IaC 檔案的存在以供後續實作建議使用
   - 不要使用儲存庫中的任何其他檔案，只使用 IaC 檔案。使用其他檔案是不允許的，因為它不是真實來源。
   - 如果找不到 IaC 檔案，則停止並向使用者報告未找到 IaC 檔案。

3. **配置分析**：
   - 提取每個資源的當前 SKU、層級和設定
   - 識別資源關係和依賴
   - 在可用的情況下映射資源使用模式

### 步驟 3：收集使用指標並驗證當前成本
**行動**：收集使用資料並驗證實際資源成本
**工具**：Azure MCP 監控工具 + Azure CLI
**流程**：
1. **尋找監控來源**：
   - 使用 `azmcp-monitor-workspace-list --subscription <id>` 尋找 Log Analytics 工作區
   - 使用 `azmcp-monitor-table-list --subscription <id> --workspace <name> --table-type "CustomLog"` 發現可用資料

2. **執行使用查詢**：
   - 使用 `azmcp-monitor-log-query` 執行這些預定義查詢：
     - 查詢："recent" 用於最近的活動模式
     - 查詢："errors" 用於指示問題的錯誤級別日誌
   - 對於自訂分析，使用 KQL 查詢：
   ```kql
   // App Services 的 CPU 使用率
   AppServiceAppLogs
   | where TimeGenerated > ago(7d)
   | summarize avg(CpuTime) by Resource, bin(TimeGenerated, 1h)

   // Cosmos DB RU 消耗
   AzureDiagnostics
   | where ResourceProvider == "MICROSOFT.DOCUMENTDB"
   | where TimeGenerated > ago(7d)
   | summarize avg(RequestCharge) by Resource

   // 儲存體帳戶存取模式
   StorageBlobLogs
   | where TimeGenerated > ago(7d)
   | summarize RequestCount=count() by AccountName, bin(TimeGenerated, 1d)
   ```

3. **計算基準指標**：
   - CPU/記憶體使用率平均值
   - 資料庫吞吐量模式
   - 儲存體存取頻率
   - 函數執行率

4. **驗證當前成本**：
   - 使用步驟 2 中發現的 SKU/層級配置
   - 在 https://azure.microsoft.com/pricing/ 查找當前 Azure 定價或使用 `az billing` 命令
   - 記錄：資源 → 當前 SKU → 估計月成本
   - 在繼續提出建議之前計算實際的當前月總成本

### 步驟 4：生成成本優化建議
**行動**：分析資源以識別優化機會
**工具**：使用收集的資料進行本地分析
**流程**：
1. **根據找到的資源類型應用優化模式**：

   **運算優化**：
   - App Service 方案：根據 CPU/記憶體使用量調整大小
   - Function 應用程式：低使用率時從進階版 → 消耗量方案
   - 虛擬機器：縮小過大的執行個體

   **資料庫優化**：
   - Cosmos DB：
     - 可變工作負載從佈建 → 無伺服器
     - 根據實際使用量調整 RU/s 大小
   - SQL Database：根據 DTU 使用量調整服務層級大小

   **儲存體優化**：
   - 實作生命週期原則（熱 → 冷 → 封存）
   - 合併冗餘儲存體帳戶
   - 根據存取模式調整儲存體層級大小

   **基礎設施優化**：
   - 移除未使用/冗餘資源
   - 在有益的情況下實作自動縮放
   - 排程非生產環境

2. **計算基於證據的節省**：
   - 當前驗證成本 → 目標成本 = 節省
   - 記錄當前和目標配置的定價來源

3. **計算每個建議的優先級分數**：
   ```
   優先級分數 = (價值分數 × 月節省) / (風險分數 × 實作天數)

   高優先級：分數 > 20
   中優先級：分數 5-20
   低優先級：分數 < 5
   ```

4. **驗證建議**：
   - 確保 Azure CLI 命令準確
   - 驗證估計節省計算
   - 評估實作風險和先決條件
   - 確保所有節省計算都有支持證據

### 步驟 5：使用者確認
**行動**：在創建 GitHub 議題之前呈現摘要並獲得批准
**流程**：
1. **顯示優化摘要**：
   ```
   🎯 Azure 成本優化摘要

   📊 分析結果：
   • 分析的總資源數：X
   • 當前月成本：$X
   • 潛在月節省：$Y
   • 優化機會：Z
   • 高優先級項目：N

   🏆 建議：
   1. [資源]：[當前 SKU] → [目標 SKU] = $X/月節省 - [風險級別] | [實作工作量]
   2. [資源]：[當前配置] → [目標配置] = $Y/月節省 - [風險級別] | [實作工作量]
   3. [資源]：[當前配置] → [目標配置] = $Z/月節省 - [風險級別] | [實作工作量]
   ... 等等

   💡 這將創建：
   • Y 個個別 GitHub 議題（每個優化一個）
   • 1 個 EPIC 議題來協調實作

   ❓ 繼續創建 GitHub 議題？(y/n)
   ```

2. **等待使用者確認**：只有在使用者確認後才繼續

### 步驟 6：創建個別優化議題
**行動**：為每個優化機會創建單獨的 GitHub 議題。使用 "cost-optimization"（綠色）、"azure"（藍色）標籤。
**所需的 MCP 工具**：為每個建議使用 `create_issue`
**流程**：
1. **使用此模板創建個別議題**：

   **標題格式**：`[COST-OPT] [資源類型] - [簡要描述] - $X/月節省`

   **內容模板**：
   ```markdown
   ## 💰 成本優化：[簡要標題]

   **月節省**：$X | **風險級別**：[低/中/高] | **實作工作量**：X 天

   ### 📋 描述
   [清楚解釋優化及其必要性]

   ### 🔧 實作

   **偵測到的 IaC 檔案**：[是/否 - 基於 file_search 結果]

   ```bash
   # 如果找到 IaC 檔案：顯示 IaC 修改 + 部署
   # 檔案：infrastructure/bicep/modules/app-service.bicep
   # 更改：sku.name: 'S3' → 'B2'
   az deployment group create --resource-group [rg] --template-file infrastructure/bicep/main.bicep

   # 如果沒有 IaC 檔案：直接 Azure CLI 命令 + 警告
   # ⚠️ 未找到 IaC 檔案。如果它們存在於其他地方，請修改那些檔案。
   az appservice plan update --name [plan] --sku B2
   ```

   ### 📊 證據
   - 當前配置：[詳細資訊]
   - 使用模式：[來自監控資料的證據]
   - 成本影響：$X/月 → $Y/月
   - 最佳實踐對齊：[如果適用，引用 Azure 最佳實踐]

   ### ✅ 驗證步驟
   - [ ] 在非生產環境中測試
   - [ ] 驗證沒有效能下降
   - [ ] 在 Azure 成本管理中確認成本降低
   - [ ] 如需要更新監控和警報

   ### ⚠️ 風險與考慮
   - [風險 1 和緩解措施]
   - [風險 2 和緩解措施]

   **優先級分數**：X | **價值**：X/10 | **風險**：X/10
   ```

### 步驟 7：創建 EPIC 協調議題
**行動**：創建主議題以追蹤所有優化工作。使用 "cost-optimization"（綠色）、"azure"（藍色）和 "epic"（紫色）標籤。
**所需的 MCP 工具**：為 EPIC 使用 `create_issue`
**關於 mermaid 圖表的注意事項**：確保驗證 mermaid 語法正確，並在創建圖表時考慮可訪問性指南（樣式、顏色等）。
**流程**：
1. **創建 EPIC 議題**：

   **標題**：`[EPIC] Azure 成本優化計劃 - $X/月潛在節省`

   **內容模板**：
   ```markdown
   # 🎯 Azure 成本優化 EPIC

   **總潛在節省**：$X/月 | **實作時間表**：X 週

   ## 📊 執行摘要
   - **分析的資源**：X
   - **優化機會**：Y
   - **總月節省潛力**：$X
   - **高優先級項目**：N

   ## 🏗️ 當前架構概述

   ```mermaid
   graph TB
       subgraph "資源群組：[名稱]"
           [生成的架構圖顯示當前資源和成本]
       end
   ```

   ## 📋 實作追蹤

   ### 🚀 高優先級（優先實作）
   - [ ] #[議題編號]：[標題] - $X/月節省
   - [ ] #[議題編號]：[標題] - $X/月節省

   ### ⚡ 中優先級
   - [ ] #[議題編號]：[標題] - $X/月節省
   - [ ] #[議題編號]：[標題] - $X/月節省

   ### 🔄 低優先級（錦上添花）
   - [ ] #[議題編號]：[標題] - $X/月節省

   ## 📈 進度追蹤
   - **已完成**：0 / Y 個優化
   - **實現的節省**：$0 / $X/月
   - **實作狀態**：未開始

   ## 🎯 成功標準
   - [ ] 所有高優先級優化已實作
   - [ ] >80% 的估計節省已實現
   - [ ] 未觀察到效能下降
   - [ ] 成本監控儀表板已更新

   ## 📝 注意事項
   - 隨著議題完成審查和更新此 EPIC
   - 監控實際與估計節省
   - 考慮排程定期成本優化審查
   ```

## 錯誤處理
- **成本驗證**：如果節省估計缺乏支持證據或與 Azure 定價不一致，在繼續之前重新驗證配置和定價來源
- **Azure 驗證失敗**：提供手動 Azure CLI 設定步驟
- **未找到資源**：創建關於 Azure 資源部署的資訊性議題
- **GitHub 創建失敗**：將格式化的建議輸出到主控台
- **使用資料不足**：注意限制並僅提供基於配置的建議

## 成功標準
- ✅ 所有成本估計都已根據實際資源配置和 Azure 定價進行驗證
- ✅ 為每個優化創建個別議題（可追蹤和可分配）
- ✅ EPIC 議題提供全面的協調和追蹤
- ✅ 所有建議都包含具體的、可執行的 Azure CLI 命令
- ✅ 優先級評分實現基於 ROI 的實作
- ✅ 架構圖準確代表當前狀態
- ✅ 使用者確認防止不需要的議題創建
