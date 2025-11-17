# 為 Awesome GitHub Copilot 做出貢獻

感謝您有興趣為 Awesome GitHub Copilot 專案做出貢獻！我們歡迎社群的貢獻，協助擴充我們的自訂指令與提示詞集合。

## 如何貢獻

### 新增指令

指令協助針對特定技術、程式碼實踐或領域自訂 GitHub Copilot 的行為。

1. **建立您的指令檔案**：在 `instructions/` 目錄中新增一個 `.md` 檔案
2. **遵循命名慣例**：使用描述性的小寫檔名並以連字號分隔（例如：`python-django.instructions.md`）
3. **組織您的內容**：從清楚的標題開始，並邏輯性地組織您的指令
4. **測試您的指令**：確保您的指令與 GitHub Copilot 運作良好

#### 指令範例格式

```markdown
---
description: '針對特定技術與實踐自訂 GitHub Copilot 行為的指令'
---

# 您的技術/框架名稱

## 指令

- 為 GitHub Copilot 提供清楚、明確的指引
- 包含最佳實踐與慣例
- 使用條列式清單便於閱讀

## 額外指引

- 任何額外的情境或範例
```

### 新增提示詞

提示詞是針對特定開發情境與任務的即用模板。

1. **建立您的提示詞檔案**：在 `prompts/` 目錄中新增一個 `.prompt.md` 檔案
2. **遵循命名慣例**：使用描述性的小寫檔名並以連字號分隔，副檔名為 `.prompt.md`（例如：`react-component-generator.prompt.md`）
3. **包含 frontmatter**：在檔案開頭新增中繼資料（選用但建議加上）
4. **組織您的提示詞**：提供清楚的情境與明確的指示

#### 提示詞範例格式

```markdown
---
mode: 'agent'
tools: ['codebase', 'terminalCommand']
description: '此提示詞功能的簡要說明'
---

# 提示詞標題

您的目標是...

## 具體指示

- 清楚、可執行的指示
- 適時包含範例
```

### 新增對話模式

對話模式是專業的設定，能將 GitHub Copilot Chat 轉換為特定領域的助手或角色，用於特定的開發情境。

1. **建立您的對話模式檔案**：在 `chatmodes/` 目錄中新增一個 `.chatmode.md` 檔案
2. **遵循命名慣例**：使用描述性的小寫檔名並以連字號分隔，副檔名為 `.chatmode.md`（例如：`react-performance-expert.chatmode.md`）
3. **包含 frontmatter**：在檔案開頭新增包含必要欄位的中繼資料
4. **定義角色**：為對話模式建立清楚的身分與專業領域
5. **測試您的對話模式**：確保對話模式在其領域中提供有用且準確的回應

#### 對話模式範例格式

```markdown
---
description: '對話模式與其用途的簡要說明'
model: 'gpt-5'
tools: ['codebase', 'terminalCommand']
---

# 對話模式標題

您是 [領域/角色] 專家，在 [特定領域] 擁有深入的知識。

## 您的專業

- [特定技能 1]
- [特定技能 2]
- [特定技能 3]

## 您的方法

- [如何協助使用者]
- [您的溝通風格]
- [您優先考慮的事項]

## 指引

- [回應的具體指示]
- [限制或約束]
- [應遵循的最佳實踐]
```

### 新增集合

集合將相關的提示詞、指令與對話模式依特定主題或工作流程組織起來，讓使用者更容易探索並採用完整的工具組。

1. **建立您的集合清單**：在 `collections/` 目錄中新增一個 `.collection.yml` 檔案
2. **遵循命名慣例**：使用描述性的小寫檔名並以連字號分隔（例如：`python-web-development.collection.yml`）
3. **參照現有項目**：集合應該只參照專案中已存在的檔案
4. **測試您的集合**：驗證所有參照的檔案都存在且能良好協作

#### 建立集合

```bash
# 使用建立腳本
node create-collection.js my-collection-id

# 或使用 VS Code 任務：Ctrl+Shift+P > "Tasks: Run Task" > "create-collection"
```

#### 集合範例格式

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
    usage: |
     recommended # or "optional" if not essential to the workflow

     This chat mode requires the following instructions/prompts/MCPs:
      - Instruction 1
      - Prompt 1
      - MCP 1

     This chat mode is ideal for...
      - Use case 1
      - Use case 2
    
      Here is an example of how to use it:
      ```markdown, task-plan.prompt.md
      ---
      mode: task-planner
      title: Plan microsoft fabric realtime intelligence terraform support
      ---
      #file: <file including in chat context>
      Do an action to achieve goal.
      ```

      To get the best results, consider...
      - Tip 1
      - Tip 2
    
display:
  ordering: alpha # or "manual" to preserve order above
  show_badge: false # set to true to show collection badge
```

完整使用範例請參考 edge-ai tasks 集合：
- [edge-ai-tasks.collection.yml](./collections/edge-ai-tasks.collection.yml)
- [edge-ai-tasks.md](./collections/edge-ai-tasks.md)

#### 集合指引

- **專注於工作流程**：將特定使用情境中能協作的項目組合在一起
- **合理的大小**：通常 3-10 個項目效果最佳
- **測試組合**：確保項目之間能有效互補
- **明確的目的**：集合應該解決特定的問題或工作流程
- **提交前驗證**：執行 `node validate-collections.js` 以確保您的清單有效

## 提交您的貢獻

1. **Fork 此專案**
2. **建立新分支**供您的貢獻使用
3. **依照上述指引新增您的指令、提示詞檔案、對話模式或集合**
4. **執行更新腳本**：執行 `npm start` 以用您的新檔案更新 README（如果還沒執行過，請先執行 `npm install`）
   - GitHub Actions 工作流程會驗證此步驟是否正確執行
   - 如果執行腳本會修改 README.md，PR 檢查將會失敗並顯示需要的變更註解
5. **提交 pull request**，包含：
   - 描述您貢獻的清楚標題
   - 您的指令/提示詞功能的簡要說明
   - 任何相關的情境或使用說明

**注意**：您的貢獻合併後，您將自動被加入我們的[貢獻者](./README.md#contributors-)區段！我們使用 [all-contributors](https://github.com/all-contributors/all-contributors) 來表彰各類貢獻。

## 我們接受的內容

我們歡迎任何能協助開發者更有效使用 GitHub Copilot 的技術、框架或開發實踐的貢獻。包括：

- 程式語言與框架
- 開發方法論與最佳實踐
- 架構模式與設計原則
- 測試策略與品質保證
- DevOps 與部署實踐
- 無障礙設計與包容性設計
- 效能最佳化技術

## 我們不接受的內容

為了維護安全、負責任且具建設性的社群，我們**不接受**以下貢獻：

- **違反負責任 AI 原則**：試圖規避 Microsoft/GitHub 負責任 AI 指引或推廣有害 AI 使用的內容
- **危害安全性**：旨在繞過安全性政策、利用漏洞或削弱系統安全性的指令
- **啟用惡意活動**：意圖傷害其他系統、使用者或組織的內容
- **利用弱點**：利用其他平台或服務漏洞的指令
- **推廣有害內容**：可能導致建立有害、歧視性或不當內容的指引
- **規避平台政策**：試圖繞過 GitHub、Microsoft 或其他平台服務條款的內容

## 品質指引

- **明確具體**：明確且可執行的指引比籠統的指令更有幫助
- **測試您的內容**：確保您的指令或提示詞與 GitHub Copilot 運作良好
- **遵循慣例**：使用一致的格式與命名
- **保持專注**：每個檔案應該針對特定的技術、框架或使用情境
- **清楚書寫**：使用簡單、直接的語言
- **推廣最佳實踐**：鼓勵安全、可維護且符合道德的開發實踐

## 貢獻者表彰

本專案使用 [all-contributors](https://github.com/all-contributors/all-contributors) 來表彰貢獻者。當您做出貢獻時，您將自動被納入我們的貢獻者清單！

我們歡迎各類貢獻，包括：

- 📝 改善文件
- 💻 程式碼貢獻
- 🐛 錯誤回報與修復
- 🎨 設計改善
- 💡 想法與建議
- 🤔 回答問題
- 📢 推廣專案

您的貢獻讓這個資源對整個 GitHub Copilot 社群更有幫助！

## 行為準則

請注意，本專案發布時附有[貢獻者行為準則](CODE_OF_CONDUCT.md)。參與本專案即表示您同意遵守其條款。

## 授權

向本專案做出貢獻，即表示您同意您的貢獻將以 MIT 授權釋出。
