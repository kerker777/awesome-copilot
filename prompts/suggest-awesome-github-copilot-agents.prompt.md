---
agent: "agent"
description: "Suggest relevant GitHub Copilot Custom Agents files from the awesome-copilot repository based on current repository context and chat history, avoiding duplicates with existing custom agents in this repository."
tools: ["edit", "search", "runCommands", "runTasks", "changes", "testFailure", "openSimpleBrowser", "fetch", "githubRepo", "todos"]
---

# 建議優秀的 GitHub Copilot 自訂代理

分析目前的儲存庫內容，並建議來自 [GitHub awesome-copilot 儲存庫](https://github.com/github/awesome-copilot/blob/main/docs/README.agents.md) 中的相關自訂代理檔案，這些檔案在本儲存庫中還未存在。自訂代理檔案位於 awesome-copilot 儲存庫的 [agents](https://github.com/github/awesome-copilot/tree/main/agents) 資料夾中。

## 流程

1. **取得可用的自訂代理**：從 [awesome-copilot README.agents.md](https://github.com/github/awesome-copilot/blob/main/docs/README.agents.md) 中提取自訂代理清單與描述。必須使用 `fetch` 工具。
2. **掃描本地自訂代理**：探索 `.github/agents/` 資料夾中現有的自訂代理檔案
3. **提取描述**：讀取本地自訂代理檔案的前言資訊以取得描述
4. **分析內容**：檢視聊天歷史、儲存庫檔案與目前的專案需求
5. **比較現有項目**：檢查本儲存庫中已有的自訂代理
6. **符合相關性**：比較可用的自訂代理與識別的模式與需求
7. **呈現選項**：顯示相關的自訂代理，包含描述、原因與可用狀態
8. **驗證**：確保所建議的代理能提供現有代理尚未涵蓋的價值
9. **輸出**：提供結構化表格，包含建議、描述與 awesome-copilot 自訂代理和類似本地自訂代理的連結
   **等待**使用者要求以繼續安裝特定的自訂代理。除非有明確指示，否則不要安裝。
10. **下載資產**：針對要求的代理，自動下載並安裝個別代理至 `.github/agents/` 資料夾。不要調整檔案內容。使用 `#todos` 工具追蹤進度。優先使用 `#fetch` 工具下載資產，但可能使用 `curl` 透過 `#runInTerminal` 工具以確保所有內容都被取得。

## 內容分析標準

🔍 **儲存庫模式**:

- 使用的程式語言 (.cs, .js, .py 等)
- 框架指標 (ASP.NET、React、Azure 等)
- 專案類型 (網路應用、API、程式庫、工具)
- 文件需求 (README、規格、ADR)

🗨️ **聊天歷史內容**:

- 最近的討論與痛點
- 功能要求或實現需求
- 程式碼審查模式
- 開發工作流程需求

## 輸出格式

在結構化表格中顯示分析結果，比較 awesome-copilot 自訂代理與現有儲存庫自訂代理：

| Awesome-Copilot 自訂代理                                                                                                                            | 描述                                                                                                                                                                | 已安裝 | 類似的本地自訂代理         | 建議原因                                          |
| ------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------- | ---------------------------------- | ------------------------------------------------------------- |
| [amplitude-experiment-implementation.agent.md](https://github.com/github/awesome-copilot/blob/main/agents/amplitude-experiment-implementation.agent.md) | 此自訂代理使用 Amplitude 的 MCP 工具以在 Amplitude 內部署新實驗，提供無縫的變體測試功能與產品功能推出能力 | ❌ 否   | 無                               | 將增強產品內的實驗功能 |
| [launchdarkly-flag-cleanup.agent.md](https://github.com/github/awesome-copilot/blob/main/agents/launchdarkly-flag-cleanup.agent.md)                     | LaunchDarkly 的功能旗標清理代理                                                                                                                                | ✅ 是   | launchdarkly-flag-cleanup.agent.md | 現有 LaunchDarkly 自訂代理已涵蓋 |

## 本地代理探索流程

1. 列出 `.github/agents/` 目錄中所有 `*.agent.md` 檔案
2. 對於每個發現的檔案，讀取前言資訊以提取 `description`
3. 建立現有代理的全面清單
4. 使用此清單以避免建議重複項目

## 需求

- 使用 `githubRepo` 工具以從 awesome-copilot 儲存庫代理資料夾取得內容
- 掃描本地檔案系統以尋找 `.github/agents/` 目錄中的現有代理
- 讀取本地代理檔案的 YAML 前言資訊以提取描述
- 與本儲存庫中的現有代理進行比較以避免重複
- 專注於目前代理程式庫涵蓋範圍的差距
- 驗證所建議的代理與儲存庫的目的和標準一致
- 為每項建議提供清晰的原因
- 包含 awesome-copilot 代理和類似本地代理的連結
- 除了表格和分析外，不要提供任何額外資訊或內容

## 圖示參考

- ✅ 已在儲存庫中安裝
- ❌ 未在儲存庫中安裝
