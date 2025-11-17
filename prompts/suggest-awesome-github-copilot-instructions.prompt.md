---
mode: 'agent'
description: 'Suggest relevant GitHub Copilot instruction files from the awesome-copilot repository based on current repository context and chat history, avoiding duplicates with existing instructions in this repository.'
tools: ['edit', 'search', 'runCommands', 'runTasks', 'think', 'changes', 'testFailure', 'openSimpleBrowser', 'fetch', 'githubRepo', 'todos', 'search']
---
# 建議 GitHub Copilot 指令檔

根據目前儲存庫的上下文和聊天記錄，分析並建議從 [GitHub awesome-copilot 儲存庫](https://github.com/github/awesome-copilot/blob/main/docs/README.instructions.md) 中與本儲存庫不重複的相關 copilot-instruction 檔案。

## 流程

1. **取得可用指令**: 從 [awesome-copilot README.instructions.md](https://github.com/github/awesome-copilot/blob/main/docs/README.instructions.md) 提取指令清單和描述。必須使用 `#fetch` 工具。
2. **掃描本地指令**: 探索 `.github/instructions/` 資料夾中現有的指令檔案
3. **提取描述**: 讀取本地指令檔案的前置事項以取得描述和 `applyTo` 模式
4. **分析上下文**: 檢查聊天記錄、儲存庫檔案和目前的專案需求
5. **比較現有項目**: 檢查本儲存庫中已有的指令
6. **匹配相關性**: 將可用指令與已識別的模式和需求進行比較
7. **展示選項**: 顯示相關指令、描述、理由和可用性狀態
8. **驗證**: 確保建議的指令能增加現有指令尚未涵蓋的價值
9. **輸出**: 提供包含建議、描述和連結的結構化表格，連結到 awesome-copilot 指令和類似的本地指令
   **等待**使用者要求進行特定指令的安裝。除非受指示，否則不要安裝。
10. **下載資源**: 針對要求的指令，自動下載和安裝個別指令到 `.github/instructions/` 資料夾。不要調整檔案內容。使用 `#todos` 工具追蹤進度。優先使用 `#fetch` 工具下載資源，但可能使用 `curl` 搭配 `#runInTerminal` 工具以確保所有內容都被取得。

## 上下文分析準則

🔍 **儲存庫模式**:
- 使用的程式設計語言 (.cs, .js, .py, .ts 等)
- 框架指標 (ASP.NET、React、Azure、Next.js 等)
- 專案類型 (網頁應用程式、API、程式庫、工具)
- 開發工作流程需求 (測試、CI/CD、部署)

🗨️ **聊天記錄上下文**:
- 最近的討論和痛點
- 技術相關問題
- 編碼標準討論
- 開發工作流程需求

## 輸出格式

在結構化表格中顯示分析結果，比較 awesome-copilot 指令與現有儲存庫指令：

| Awesome-Copilot 指令 | 描述 | 已安裝 | 類似的本地指令 | 建議理由 |
|------------------------------|-------------|-------------------|---------------------------|---------------------|
| [blazor.instructions.md](https://github.com/github/awesome-copilot/blob/main/instructions/blazor.instructions.md) | Blazor 開發指南 | ❌ 否 | blazor.instructions.md | 已由現有 Blazor 指令涵蓋 |
| [reactjs.instructions.md](https://github.com/github/awesome-copilot/blob/main/instructions/reactjs.instructions.md) | ReactJS 開發標準 | ❌ 否 | 無 | 將以建立的模式增強 React 開發 |
| [java.instructions.md](https://github.com/github/awesome-copilot/blob/main/instructions/java.instructions.md) | Java 開發最佳實踐 | ❌ 否 | 無 | 可提升 Java 程式碼品質和一致性 |

## 本地指令發現流程

1. 列出 `instructions/` 目錄中的所有 `*.instructions.md` 檔案
2. 針對每個探索的檔案，讀取前置事項以提取 `description` 和 `applyTo` 模式
3. 建立現有指令的完整清單，包含其適用的檔案模式
4. 使用此清單避免建議重複項目

## 檔案結構需求

根據 GitHub 文件，copilot-instructions 檔案應為：
- **儲存庫範圍指令**: `.github/copilot-instructions.md` (適用於整個儲存庫)
- **路徑特定指令**: `.github/instructions/NAME.instructions.md` (透過 `applyTo` 前置事項適用於特定檔案模式)
- **社群指令**: `instructions/NAME.instructions.md` (用於共享和分發)

## 前置事項結構

awesome-copilot 中的指令檔案使用此前置事項格式：
```markdown
---
description: '此指令提供的內容的簡短描述'
applyTo: '**/*.js,**/*.ts' # 選項：檔案匹配的 glob 模式
---
```

## 需求

- 使用 `githubRepo` 工具從 awesome-copilot 儲存庫取得內容
- 掃描本地檔案系統中 `instructions/` 目錄內現有的指令
- 從本地指令檔案讀取 YAML 前置事項以提取描述和 `applyTo` 模式
- 與本儲存庫中現有的指令進行比較以避免重複
- 專注於目前指令程式庫涵蓋範圍的缺口
- 驗證建議的指令與儲存庫的目的和標準一致
- 為每項建議提供清晰的理由
- 包含連結到 awesome-copilot 指令和類似本地指令的連結
- 考慮技術堆棧相容性和專案特定需求
- 不要提供表格和分析之外的任何額外資訊或上下文

## 圖示參考

- ✅ 已在儲存庫中安裝
- ❌ 未在儲存庫中安裝
