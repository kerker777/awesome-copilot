# Collections 範本

使用此範本來建立一個包含相關 prompts、instructions 和 chat modes 的新 collection。

## 基本範本

```yaml
id: my-collection-id
name: My Collection Name
description: A brief description of what this collection provides and who should use it.
tags: [tag1, tag2, tag3] # Optional discovery tags
items:
  - path: prompts/my-prompt.prompt.md
    kind: prompt
  - path: instructions/my-instructions.instructions.md
    kind: instruction
  - path: chatmodes/my-chatmode.chatmode.md
    kind: chat-mode
display:
  ordering: alpha # or "manual" to preserve order above
  show_badge: false # set to true to show collection badge
```

## 欄位說明

- **id**：唯一識別碼，僅使用小寫字母、數字和連字號
- **name**：collection 的顯示名稱
- **description**：collection 用途的簡短說明（1-500 個字元）
- **tags**：選填的探索標籤陣列（最多 10 個，每個 1-30 個字元）
- **items**：collection 中的項目陣列（1-50 個項目）
  - **path**：從儲存庫根目錄到檔案的相對路徑
  - **kind**：必須是 `prompt`、`instruction` 或 `chat-mode`
- **display**：選填的顯示設定
  - **ordering**：`alpha`（字母順序）或 `manual`（保留順序）
  - **show_badge**：在項目上顯示 collection 徽章（true/false）

## 建立新的 Collection

### 使用 VS Code Tasks
1. 按下 `Ctrl+Shift+P`（Mac 上為 `Cmd+Shift+P`）
2. 輸入「Tasks: Run Task」
3. 選擇「create-collection」
4. 在提示時輸入您的 collection ID

### 使用命令列
```bash
node create-collection.js my-collection-id
```

### 手動建立
1. 建立 `collections/my-collection-id.collection.yml`
2. 使用上述範本作為起點
3. 加入您的項目並自訂設定
4. 執行 `npm run validate:collections` 進行驗證
5. 執行 `npm start` 生成文件

## 驗證

Collections 會自動驗證以確保：
- 必要欄位存在且有效
- 檔案路徑存在且符合項目類型
- ID 在所有 collections 中是唯一的
- 標籤和顯示設定符合結構定義

手動執行驗證：
```bash
npm run validate:collections
```

## 檔案組織

Collections 不需要重新組織現有檔案。只要在清單中的路徑正確，項目可以位於儲存庫中的任何位置。

## 最佳實務

1. **有意義的 Collections**：將能在特定工作流程或使用案例中良好協作的項目組合在一起
2. **清晰的命名**：使用能反映 collection 用途的描述性名稱和 ID
3. **良好的說明**：解釋誰應該使用這個 collection 以及它提供什麼好處
4. **相關的標籤**：加入能幫助使用者找到相關 collections 的探索標籤
5. **合理的大小**：保持 collections 的專注性 - 通常 3-10 個項目效果最好
6. **測試項目**：在加入 collection 之前，確保所有引用的檔案都存在且可正常運作
