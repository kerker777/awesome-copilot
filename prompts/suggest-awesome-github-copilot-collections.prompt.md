---
mode: 'agent'
description: 'Suggest relevant GitHub Copilot collections from the awesome-copilot repository based on current repository context and chat history, providing automatic download and installation of collection assets.'
tools: ['edit', 'search', 'runCommands', 'runTasks', 'think', 'changes', 'testFailure', 'openSimpleBrowser', 'fetch', 'githubRepo', 'todos', 'search']
---
# 建議引入 Awesome GitHub Copilot 集合

分析目前儲存庫的背景內容，並建議來自 [GitHub awesome-copilot 儲存庫](https://github.com/github/awesome-copilot/blob/main/docs/README.collections.md)中相關的集合，這些集合能夠增強此儲存庫的開發工作流程。

## 流程

1. **取得可用的集合**：從 [awesome-copilot README.collections.md](https://github.com/github/awesome-copilot/blob/main/docs/README.collections.md) 提取集合清單和說明。必須使用 `#fetch` 工具。
2. **掃描本機資源**：發現 `prompts/`、`instructions/` 和 `chatmodes/` 資料夾中現有的提示檔案、指令檔案和聊天模式
3. **提取本機說明**：讀取本機資源檔的前置資料 (front matter) 以瞭解現有的功能
4. **分析儲存庫背景**：檢視聊天記錄、儲存庫檔案、程式設計語言、框架和目前的專案需求
5. **比對集合相關性**：將可用的集合與已識別的模式和需求進行比較
6. **檢查資源重疊**：對於相關的集合，分析個別項目，避免與現有儲存庫資源重複
7. **呈現集合選項**：顯示相關的集合及其說明、項目計數和建議的理由
8. **提供使用指引**：說明已安裝的集合如何增強開發工作流程
   **靜候** 使用者的要求以繼續安裝特定集合。除非有明確指示，否則請勿安裝。
9. **下載資源**：對於要求的集合，自動下載並安裝每個個別資源（提示、指令、聊天模式）到適當的目錄。請勿調整檔案內容。優先使用 `#fetch` 工具下載資源，但可在必要時使用 `#runInTerminal` 工具中的 `curl` 確保所有內容都已取得。

## 背景分析準則

🔍 **儲存庫模式**：
- 使用的程式設計語言（.cs、.js、.py、.ts、.bicep、.tf 等）
- 框架指標（ASP.NET、React、Azure、Next.js、Angular 等）
- 專案類型（Web 應用程式、API、程式庫、工具、基礎結構）
- 文件需求（README、規格、ADR、架構決策）
- 開發工作流程指標（CI/CD、測試、部署）

🗨️ **聊天記錄背景**：
- 最近的討論和痛點
- 功能要求或實現需求
- 程式碼審查模式和品質關注事項
- 開發工作流程的需求和挑戰
- 技術堆疊和架構決策

## 輸出格式

在結構化表格中顯示分析結果，呈現相關集合及其潛在價值：

### 集合建議

| 集合名稱 | 說明 | 項目 | 資源重疊 | 建議理由 |
|---------|------|------|---------|--------|
| [Azure 與雲端開發](https://github.com/github/awesome-copilot/blob/main/collections/azure-cloud-development.md) | 全面的 Azure 雲端開發工具，包括基礎結構即程式碼 (IaC)、無伺服器函數、架構模式和成本最佳化 | 15 項 | 3 項相似 | 能透過 Bicep、Terraform 和成本最佳化工具來增強 Azure 開發工作流程 |
| [C# .NET 開發](https://github.com/github/awesome-copilot/blob/main/collections/csharp-dotnet-development.md) | C# 和 .NET 開發的必要提示、指令和聊天模式，包括測試、文件和最佳實踐 | 7 項 | 2 項相似 | 現有的 .NET 相關資源已涵蓋，但包含進階測試模式 |
| [測試與測試自動化](https://github.com/github/awesome-copilot/blob/main/collections/testing-automation.md) | 針對撰寫測試、測試自動化和測試驅動開發的全面集合 | 11 項 | 1 項相似 | 能透過 TDD 指引和自動化工具大幅改進測試實踐 |

### 建議集合的資源分析

針對每個建議的集合，詳細列出個別資源：

**Azure 與雲端開發集合分析：**
- ✅ **新資源（12 項）**：Azure 成本最佳化提示、Bicep 規劃模式、AVM 模組、Logic Apps 專家模式
- ⚠️ **相似資源（3 項）**：Azure DevOps 管線（類似於現有 CI/CD）、Terraform（基本重疊）、容器化（Docker 基礎已涵蓋）
- 🎯 **高價值**：成本最佳化工具、基礎結構即程式碼 (IaC) 專業知識、Azure 特定的架構指引

**安裝預覽：**
- 將安裝到 `prompts/`：4 個 Azure 特定的提示
- 將安裝到 `instructions/`：6 個基礎結構和 DevOps 最佳實踐
- 將安裝到 `chatmodes/`：5 個專門的 Azure 專家模式

## 本機資源發現流程

1. **掃描資源目錄**：
   - 列出 `prompts/` 目錄中的所有 `*.prompt.md` 檔案
   - 列出 `instructions/` 目錄中的所有 `*.instructions.md` 檔案
   - 列出 `chatmodes/` 目錄中的所有 `*.chatmode.md` 檔案

2. **提取資源中繼資料**：對於每個發現的檔案，讀取 YAML 前置資料以提取：
   - `description` - 主要目的和功能
   - `tools` - 所需的工具和功能
   - `mode` - 操作模式（針對提示）
   - `model` - 特定的模型需求（針對聊天模式）

3. **建立資源清冊**：建立現有功能的全面地圖，按以下方式組織：
   - **技術焦點**：程式設計語言、框架、平台
   - **工作流程類型**：開發、測試、部署、文件、規劃
   - **專業化級別**：通用 vs. 專門的專家模式

4. **識別涵蓋差距**：將現有資源與以下項目進行比較：
   - 儲存庫技術堆疊需求
   - 聊天記錄指示的開發工作流程需求
   - 已識別專案類型的業界最佳實踐
   - 缺少的專業領域（安全、效能、架構等）

## 集合資源下載流程

當使用者確認集合安裝時：

1. **取得集合清單**：從 awesome-copilot 儲存庫取得集合 YAML
2. **下載個別資源**：針對集合中的每個項目：
   - 從 GitHub 下載原始檔案內容
   - 驗證檔案格式和前置資料結構
   - 檢查命名慣例是否符合
3. **安裝到適當的目錄**：
   - `*.prompt.md` 檔案 → `prompts/` 目錄
   - `*.instructions.md` 檔案 → `instructions/` 目錄
   - `*.chatmode.md` 檔案 → `chatmodes/` 目錄
4. **避免重複**：跳過與現有資源基本相似的檔案
5. **報告安裝**：提供已安裝資源的摘要和使用指示

## 需求

- 使用 `fetch` 工具從 awesome-copilot 儲存庫取得集合資料
- 使用 `githubRepo` 工具取得個別資源內容以供下載
- 掃描本機檔案系統，尋找 `prompts/`、`instructions/` 和 `chatmodes/` 目錄中現有的資源
- 讀取本機資源檔的 YAML 前置資料以提取說明和功能
- 將集合與儲存庫背景進行比較，識別相關的相符項
- 專注於填補功能缺口的集合，而不是複製現有資源
- 驗證建議的集合與儲存庫的技術堆疊和開發需求是否一致
- 為每個集合建議提供明確的理由和具體好處
- 啟用集合資源的自動下載和安裝到適當的目錄
- 確保下載的資源遵循儲存庫命名慣例和格式化標準
- 提供使用指引，說明集合如何增強開發工作流程
- 包含指向 awesome-copilot 集合和集合內個別資源的連結

## 集合安裝工作流程

1. **使用者確認集合**：使用者選擇特定集合進行安裝
2. **取得集合清單**：從 awesome-copilot 儲存庫下載 YAML 清單
3. **資源下載迴圈**：針對集合中的每個資源：
   - 從 GitHub 儲存庫下載原始內容
   - 驗證檔案格式和結構
   - 檢查與現有本機資源的實質性重疊
   - 安裝到適當的目錄（`prompts/`、`instructions/` 或 `chatmodes/`）
4. **安裝摘要**：報告已安裝資源和使用指示
5. **工作流程增強指南**：說明集合如何改進開發功能

## 安裝後指引

安裝集合後，提供：
- **資源概觀**：已安裝的提示、指令和聊天模式清單
- **使用範例**：如何啟用和使用每種類型的資源
- **工作流程整合**：將資源納入開發流程的最佳實踐
- **自訂提示**：如何針對特定專案需求修改資源
- **相關集合**：建議與其他相關集合配合使用

## 圖示參考

- ✅ 建議安裝的集合
- ⚠️ 集合有部分資源重疊，但仍有價值
- ❌ 不建議的集合（重大重疊或不相關）
- 🎯 填補重大功能缺口的高價值集合
- 📁 部分安裝的集合（部分資源因重複而跳過）
- 🔄 需要針對儲存庫進行自訂的集合
