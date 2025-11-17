---
mode: 'agent'
description: 'Suggest relevant GitHub Copilot prompt files from the awesome-copilot repository based on current repository context and chat history, avoiding duplicates with existing prompts in this repository.'
tools: ['edit', 'search', 'runCommands', 'runTasks', 'think', 'changes', 'testFailure', 'openSimpleBrowser', 'fetch', 'githubRepo', 'todos', 'search']
---
# 建議優秀的 GitHub Copilot 提示

分析目前的版本庫背景，並建議來自 [GitHub awesome-copilot 版本庫](https://github.com/github/awesome-copilot/blob/main/docs/README.prompts.md) 中這個版本庫尚未提供的相關提示檔案。

## 流程

1. **取得可用提示**：從 [awesome-copilot README.prompts.md](https://github.com/github/awesome-copilot/blob/main/docs/README.prompts.md) 提取提示清單和說明。必須使用 `#fetch` 工具。
2. **掃描本地提示**：發現 `.github/prompts/` 資料夾中的現有提示檔案
3. **提取說明**：從本地提示檔案讀取 frontmatter 以取得說明
4. **分析背景**：檢查對話歷史、版本庫檔案和目前的專案需求
5. **比較現有項目**：檢查此版本庫中已有的提示
6. **配對相關性**：將可用提示與已識別的模式和需求進行比較
7. **展示選項**：顯示相關提示的說明、理由和可用狀態
8. **驗證**：確保建議的提示可以增加現有提示未涵蓋的價值
9. **輸出**：提供結構化表格，包含建議、說明和 awesome-copilot 提示和類似本地提示的連結
   **等待**使用者要求繼續安裝特定指令。除非被指示，否則不要安裝。
10. **下載資源**：對於要求的指令，自動下載並安裝個別指令到 `.github/prompts/` 資料夾。不要調整檔案內容。使用 `#todos` 工具來追蹤進度。優先使用 `#fetch` 工具下載資源，但可能會使用 `curl` 搭配 `#runInTerminal` 工具以確保所有內容都被取得。

## 背景分析準則

🔍 **版本庫模式**：
- 使用的程式語言（.cs、.js、.py 等）
- 框架指標（ASP.NET、React、Azure 等）
- 專案類型（網頁應用、API、程式庫、工具）
- 文件需求（README、規格、決策記錄）

🗨️ **對話歷史背景**：
- 最近的討論和痛點
- 功能請求或實作需求
- 程式碼審查模式
- 開發工作流程需求

## 輸出格式

使用結構化表格顯示分析結果，比較 awesome-copilot 提示和現有版本庫提示：

| Awesome-Copilot 提示 | 說明 | 已安裝 | 類似本地提示 | 建議原因 |
|-------------------------|-------------|-------------------|---------------------|---------------------|
| [code-review.md](https://github.com/github/awesome-copilot/blob/main/prompts/code-review.md) | 自動化程式碼審查提示 | ❌ 否 | 無 | 將以標準化程式碼審查流程增強開發工作流程 |
| [documentation.md](https://github.com/github/awesome-copilot/blob/main/prompts/documentation.md) | 產生專案文件 | ✅ 是 | create_oo_component_documentation.prompt.md | 已由現有文件提示涵蓋 |
| [debugging.md](https://github.com/github/awesome-copilot/blob/main/prompts/debugging.md) | 除錯協助提示 | ❌ 否 | 無 | 可提高開發小組的故障排除效率 |

## 本地提示發現流程

1. 列出 `.github/prompts/` 目錄中的所有 `*.prompt.md` 檔案。
2. 對於每個發現的檔案，讀取 frontmatter 以提取 `description`
3. 建立現有提示的全面清單
4. 使用此清單來避免建議重複項目

## 需求

- 使用 `githubRepo` 工具從 awesome-copilot 版本庫取得內容
- 掃描本地檔案系統中 `.github/prompts/` 目錄的現有提示
- 從本地提示檔案讀取 YAML frontmatter 以提取說明
- 與此版本庫中現有的提示進行比較，以避免重複
- 專注於目前提示程式庫涵蓋範圍的差距
- 驗證建議的提示是否符合版本庫的目的和標準
- 為每個建議提供明確的理由
- 包含 awesome-copilot 提示和類似本地提示的連結
- 不要提供超過表格和分析以外的任何額外資訊或背景


## 圖示參考

- ✅ 已安裝在版本庫中
- ❌ 未安裝在版本庫中
