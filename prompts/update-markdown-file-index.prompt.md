---
mode: 'agent'
description: '使用指定資料夾中的檔案索引/表格更新 markdown 檔案區段。'
tools: ['changes', 'search/codebase', 'edit/editFiles', 'extensions', 'fetch', 'findTestFiles', 'githubRepo', 'openSimpleBrowser', 'problems', 'runCommands', 'runTasks', 'runTests', 'search', 'search/searchResults', 'runCommands/terminalLastCommand', 'runCommands/terminalSelection', 'testFailure', 'usages', 'vscodeAPI']
---
# 更新 Markdown 檔案索引

使用資料夾 `${input:folder}` 中的檔案索引/表格更新 markdown 檔案 `${file}`。

## 流程

1. **掃描**：讀取目標 markdown 檔案 `${file}` 以了解現有結構
2. **探索**：列出指定資料夾 `${input:folder}` 中符合模式 `${input:pattern}` 的所有檔案
3. **分析**：識別是否存在要更新的現有表格/索引區段，或建立新結構
4. **結構**：根據檔案類型和現有內容產生適當的表格/清單格式
5. **更新**：使用檔案索引替換現有區段或新增新區段
6. **驗證**：確保 markdown 語法有效且格式一致

## 檔案分析

對於每個探索到的檔案，擷取：

- **名稱**：根據背景決定是否包含副檔名的檔名
- **類型**：副檔名和類別（例如，`.md`、`.js`、`.py`）
- **描述**：第一行註解、標題或推斷的目的
- **大小**：檔案大小供參考（選用）
- **修改時間**：最後修改日期（選用）

## 表格結構選項

根據檔案類型和現有內容選擇格式：

### 選項 1：簡單清單

```markdown
## ${folder} 中的檔案

- [filename.ext](path/to/filename.ext) - 描述
- [filename2.ext](path/to/filename2.ext) - 描述
```

### 選項 2：詳細表格

| 檔案 | 類型 | 描述 |
|------|------|-------------|
| [filename.ext](path/to/filename.ext) | 副檔名 | 描述 |
| [filename2.ext](path/to/filename2.ext) | 副檔名 | 描述 |

### 選項 3：分類區段

依類型/類別將檔案分組，使用單獨的區段或子表格。

## 更新策略

- 🔄 **更新現有**：如果表格/索引區段存在，則在保留結構的同時替換內容
- ➕ **新增新的**：如果沒有現有區段，則使用最適合的格式建立新區段
- 📋 **保留**：維護現有的 markdown 格式、標題層級和文件流程
- 🔗 **連結**：對儲存庫內的檔案連結使用相對路徑

## 區段識別

尋找具有以下模式的現有區段：

- 包含以下內容的標題：「index」、「files」、「contents」、「directory」、「list」
- 具有檔案相關欄位的表格
- 具有檔案連結的清單
- 標記檔案索引區段的 HTML 註解

## 需求

- 保留現有的 markdown 結構和格式
- 對檔案連結使用相對路徑
- 可用時包含檔案描述
- 預設按字母順序排序檔案
- 處理檔名中的特殊字元
- 驗證所有產生的 markdown 語法
