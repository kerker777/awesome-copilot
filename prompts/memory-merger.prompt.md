---
description: 'Merges mature lessons from a domain memory file into its instruction file. Syntax: `/memory-merger >domain [scope]` where scope is `global` (default), `user`, `workspace`, or `ws`.'
---

# 記憶合併器

您將網域記憶檔案中的成熟學習內容合併到其指令檔案中，確保知識保留且冗餘性最小。

**使用待辦清單**追蹤您在流程步驟中的進度，並讓使用者瞭解狀況。

## 範圍

記憶指令可以儲存在兩個範圍：

- **全域**（`global` 或 `user`）- 儲存在 `<global-prompts>`（`vscode-userdata:/User/prompts/`）中，適用於所有 VS Code 專案
- **工作區**（`workspace` 或 `ws`）- 儲存在 `<workspace-instructions>`（`<workspace-root>/.github/instructions/`）中，僅適用於目前的專案

預設範圍是**全域**。

在本提示的整個過程中，`<global-prompts>` 和 `<workspace-instructions>` 是指這些目錄。

## 語法

```
/memory-merger >domain-name [scope]
```

- `>domain-name` - 必需。要合併的網域（例如：`>clojure`、`>git-workflow`、`>prompt-engineering`）
- `[scope]` - 選擇性。下列其中之一：`global`、`user`（兩者皆表全域）、`workspace` 或 `ws`。預設為 `global`

**範例：**
- `/memory-merger >prompt-engineering` - 合併全域提示工程記憶
- `/memory-merger >clojure workspace` - 合併工作區 clojure 記憶
- `/memory-merger >git-workflow ws` - 合併工作區 git-workflow 記憶

## 流程

### 1. 解析輸入並讀取檔案

- **提取**使用者輸入的網域和範圍
- **決定**檔案路徑：
  - 全域：`<global-prompts>/{domain}-memory.instructions.md` → `<global-prompts>/{domain}.instructions.md`
  - 工作區：`<workspace-instructions>/{domain}-memory.instructions.md` → `<workspace-instructions>/{domain}.instructions.md`
- 使用者可能拼錯網域名稱，如果您找不到記憶檔案，請使用 glob 搜尋目錄，判斷是否有可能的配對。有疑慮時，請向使用者確認。
- **讀取**兩個檔案（記憶檔案必須存在；指令檔案可能不存在）

### 2. 分析並提案

檢視所有記憶區段，並將其呈現供合併考量：

```
## 擬合併的記憶

### 記憶：[標題]
**內容：**[關鍵要點]
**位置：**[在指令中的適當位置]

[其他記憶]...
```

說：「請檢視這些記憶。輸入 'go' 核准全部，或指定要跳過的項目。」

**停止並等待使用者輸入。**

### 3. 定義品質標準

為優秀的合併結果指令建立 10/10 標準：
1. **零知識遺失** - 保留每個細節、範例和細微差別
2. **最小冗餘** - 統合重疊的指導
3. **最大可掃描性** - 清晰的層級、平行結構、策略性加粗、邏輯分組

### 4. 合併並迭代

開發最終合併的指令**但先不更新檔案**：

1. 起草合併的指令，納入已批准的記憶
2. 評估是否符合品質標準
3. 精化結構、措辭、組織
4. 重複直到合併的指令符合 10/10 標準

### 5. 更新檔案

一旦最終合併的指令符合 10/10 標準：

- **建立或更新**指令檔案，使用最終合併的內容
  - 如果建立新檔案，請包含適當的 frontmatter
  - **合併** `applyTo` 型樣，如果兩個檔案都存在的話，從記憶和指令檔案中合併，確保全面涵蓋而無重複
- **移除**記憶檔案中的已合併區段

## 範例

```
User: "/memory-merger >clojure"

Agent:
1. Reads clojure-memory.instructions.md and clojure.instructions.md
2. Proposes 3 memories for merger
3. [STOPS]

User: "go"

Agent:
4. Defines quality bar for 10/10
5. Merges new instructions candidate, iterates to 10/10
6. Updates clojure.instructions.md
7. Cleans clojure-memory.instructions.md
```
