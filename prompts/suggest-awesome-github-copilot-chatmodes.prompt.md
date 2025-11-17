---
mode: 'agent'
description: 'Suggest relevant GitHub Copilot Custom Chat Modes files from the awesome-copilot repository based on current repository context and chat history, avoiding duplicates with existing custom chat modes in this repository.'
tools: ['edit', 'search', 'runCommands', 'runTasks', 'think', 'changes', 'testFailure', 'openSimpleBrowser', 'fetch', 'githubRepo', 'todos', 'search']
---

# 建議優質 GitHub Copilot 自訂聊天模式

分析目前儲存庫的情境，並建議來自 [GitHub awesome-copilot 儲存庫](https://github.com/github/awesome-copilot/blob/main/docs/README.chatmodes.md) 中尚未在本儲存庫提供的相關自訂聊天模式檔案。自訂聊天模式檔案位於 awesome-copilot 儲存庫的 [chatmodes](https://github.com/github/awesome-copilot/tree/main/chatmodes) 資料夾中。

## 處理流程

1. **取得可用的自訂聊天模式**：從 [awesome-copilot README.chatmodes.md](https://github.com/github/awesome-copilot/blob/main/docs/README.chatmodes.md) 中提取自訂聊天模式清單及說明。必須使用 `#fetch` 工具。
2. **掃描本地自訂聊天模式**：探索 `.github/chatmodes/` 資料夾中現有的自訂聊天模式檔案
3. **提取說明**：從本地自訂聊天模式檔案中讀取前置元數據以取得說明
4. **分析情境**：檢視聊天記錄、儲存庫檔案及目前專案需求
5. **比較現有項目**：檢查本儲存庫中已有的自訂聊天模式
6. **符合相關性**：將可用的自訂聊天模式與已識別的模式及需求進行比較
7. **呈現選項**：顯示相關的自訂聊天模式，包含說明、理由及可用狀態
8. **驗證**：確保建議的聊天模式能提供現有聊天模式尚未涵蓋的價值
9. **輸出**：提供結構化表格，包含建議、說明及 awesome-copilot 自訂聊天模式與相似本地自訂聊天模式的連結
   **等待** 使用者要求繼續安裝特定的自訂聊天模式。未經指示，請勿安裝。
10. **下載資源**：針對使用者要求的聊天模式，自動下載並安裝個別聊天模式至 `.github/chatmodes/` 資料夾。請勿修改檔案內容。使用 `#todos` 工具追蹤進度。優先使用 `#fetch` 工具下載資源，但必要時可使用 `curl` 搭配 `#runInTerminal` 工具以確保取得所有內容。

## 情境分析標準

🔍 **儲存庫模式**：
- 使用的程式語言 (.cs, .js, .py 等)
- 框架指標 (ASP.NET、React、Azure 等)
- 專案類型 (網頁應用程式、API、程式庫、工具)
- 文件需求 (README、規格書、ADR)

🗨️ **聊天記錄情境**：
- 近期討論及痛點
- 功能要求或實作需求
- 程式碼審查模式
- 開發工作流程需求

## 輸出格式

以結構化表格顯示分析結果，比較 awesome-copilot 自訂聊天模式與現有儲存庫的自訂聊天模式：

| Awesome-Copilot 自訂聊天模式 | 說明 | 已安裝 | 相似的本地自訂聊天模式 | 建議理由 |
|---------------------------|-------------|-------------------|-------------------------|---------------------|
| [code-reviewer.chatmode.md](https://github.com/github/awesome-copilot/blob/main/chatmodes/code-reviewer.chatmode.md) | 專用程式碼審查自訂聊天模式 | ❌ 否 | 無 | 將透過專門的程式碼審查協助強化開發工作流程 |
| [architect.chatmode.md](https://github.com/github/awesome-copilot/blob/main/chatmodes/architect.chatmode.md) | 軟體架構指導 | ✅ 是 | azure_principal_architect.chatmode.md | 已由現有的架構自訂聊天模式涵蓋 |
| [debugging-expert.chatmode.md](https://github.com/github/awesome-copilot/blob/main/chatmodes/debugging-expert.chatmode.md) | 偵錯協助自訂聊天模式 | ❌ 否 | 無 | 可改善開發團隊的疑難排解效率 |

## 本地聊天模式探索流程

1. 列出 `.github/chatmodes/` 目錄中所有的 `*.chatmode.md` 檔案
2. 對每個發現的檔案，讀取前置元數據以提取 `description`
3. 建立現有聊天模式的完整清單
4. 使用此清單避免建議重複項目

## 需求

- 使用 `githubRepo` 工具從 awesome-copilot 儲存庫的聊天模式資料夾取得內容
- 掃描本地檔案系統以尋找 `.github/chatmodes/` 目錄中現有的聊天模式
- 從本地聊天模式檔案中讀取 YAML 前置元數據以提取說明
- 與本儲存庫中現有的聊天模式進行比較以避免重複
- 著重於目前聊天模式程式庫涵蓋的缺口
- 驗證建議的聊天模式與儲存庫的目的及標準相符
- 為每項建議提供清晰的理由
- 包含 awesome-copilot 聊天模式及相似本地聊天模式的連結
- 不提供超出表格及分析以外的其他資訊或情境

## 圖示參考

- ✅ 已在儲存庫中安裝
- ❌ 未在儲存庫中安裝
