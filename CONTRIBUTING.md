# 參與 Awesome GitHub Copilot 專案

感謝您有意願為 Awesome GitHub Copilot 專案做出貢獻！我們歡迎社群的貢獻,協助擴充我們的自訂指引與提示詞集合。

## 如何貢獻

### 新增指引

指引用於客製化 GitHub Copilot 針對特定技術、程式開發實踐或領域的行為。

1. **建立您的指引檔案**：在 `instructions/` 目錄中新增一個 `.md` 檔案
2. **遵循命名慣例**：使用具描述性的小寫檔名,以連字號分隔（例如：`python-django.instructions.md`）
3. **組織您的內容結構**：以清楚的標題開始,並有條理地組織您的指引
4. **測試您的指引**：確保您的指引能與 GitHub Copilot 良好配合

#### 指引格式範例

```markdown
---
description: '針對特定技術與實踐客製化 GitHub Copilot 行為的指引'
---

# 您的技術/框架名稱

## 指引

- 為 GitHub Copilot 提供清楚、明確的引導
- 包含最佳實踐與慣例
- 使用條列式要點以利閱讀

## 其他準則

- 任何額外的情境或範例
```

### 新增提示詞

提示詞是針對特定開發情境與任務的即用範本。

1. **建立您的提示詞檔案**：在 `prompts/` 目錄中新增一個 `.prompt.md` 檔案
2. **遵循命名慣例**：使用具描述性的小寫檔名,以連字號分隔,並使用 `.prompt.md` 副檔名（例如：`react-component-generator.prompt.md`）
3. **包含 frontmatter**：在檔案開頭新增詮釋資料（選用但建議）
4. **組織您的提示詞結構**：提供清楚的情境與具體的指示

#### 提示詞格式範例

```markdown
---
mode: 'agent'
tools: ['codebase', 'terminalCommand']
description: '此提示詞功能的簡短說明'
---

# 提示詞標題

您的目標是...

## 具體指示

- 清楚、可執行的指示
- 在有幫助的地方包含範例
```

### 新增聊天模式

聊天模式是專業化的配置,能將 GitHub Copilot Chat 轉換為針對特定開發情境的領域專屬助手或角色。

1. **建立您的聊天模式檔案**：在 `chatmodes/` 目錄中新增一個 `.chatmode.md` 檔案
2. **遵循命名慣例**：使用具描述性的小寫檔名,以連字號分隔,並使用 `.chatmode.md` 副檔名（例如：`react-performance-expert.chatmode.md`）
3. **包含 frontmatter**：在檔案開頭新增包含必要欄位的詮釋資料
4. **定義角色**：為聊天模式建立清楚的身分與專業領域
5. **測試您的聊天模式**：確保聊天模式在其領域中提供有用且準確的回應

#### 聊天模式格式範例

```markdown
---
description: '聊天模式及其用途的簡短說明'
model: 'gpt-5'
tools: ['codebase', 'terminalCommand']
---

# 聊天模式標題

您是一位在 [特定領域] 擁有深厚知識的 [領域/角色] 專家。

## 您的專長

- [特定技能 1]
- [特定技能 2]
- [特定技能 3]

## 您的方法

- [您如何協助使用者]
- [您的溝通風格]
- [您優先考量的事項]

## 準則

- [回應的具體指示]
- [限制或約束條件]
- [應遵循的最佳實踐]
```

### 新增集合

集合將相關的提示詞、指引與聊天模式依特定主題或工作流程分組,讓使用者更容易探索與採用完整的工具組。

1. **建立您的集合清單**：在 `collections/` 目錄中新增一個 `.collection.yml` 檔案
2. **遵循命名慣例**：使用具描述性的小寫檔名,以連字號分隔（例如：`python-web-development.collection.yml`）
3. **參照現有項目**：集合應只參照專案中已存在的檔案
4. **測試您的集合**：驗證所有參照的檔案都存在且能良好配合

#### 建立集合

```bash
# 使用建立腳本
node create-collection.js my-collection-id

# 或使用 VS Code 任務：Ctrl+Shift+P > "Tasks: Run Task" > "create-collection"
```

#### 集合格式範例

```yaml
id: my-collection-id
name: My Collection Name
description: 關於此集合提供什麼以及誰應該使用它的簡短說明。
tags: [tag1, tag2, tag3] # 選用的探索標籤
items:
  - path: prompts/my-prompt.prompt.md
    kind: prompt
  - path: instructions/my-instructions.instructions.md
    kind: instruction
  - path: chatmodes/my-chatmode.chatmode.md
    kind: chat-mode
    usage: |
     recommended # 或 "optional"（如果對工作流程不是必要的話）

     此聊天模式需要以下指引/提示詞/MCP：
      - Instruction 1
      - Prompt 1
      - MCP 1

     此聊天模式適用於...
      - 使用情境 1
      - 使用情境 2

      以下是使用範例：
      ```markdown, task-plan.prompt.md
      ---
      mode: task-planner
      title: Plan microsoft fabric realtime intelligence terraform support
      ---
      #file: <file including in chat context>
      Do an action to achieve goal.
      ```

      為了獲得最佳結果,請考慮...
      - 提示 1
      - 提示 2

display:
  ordering: alpha # 或 "manual" 以保留上述順序
  show_badge: false # 設為 true 以顯示集合徽章
```

完整使用範例請參考 edge-ai tasks 集合：
- [edge-ai-tasks.collection.yml](./collections/edge-ai-tasks.collection.yml)
- [edge-ai-tasks.md](./collections/edge-ai-tasks.md)

#### 集合準則

- **聚焦於工作流程**：將適用於特定使用情境的項目分組
- **合理的大小**：通常 3-10 個項目效果較好
- **測試組合**：確保項目彼此有效互補
- **明確的目的**：集合應解決特定的問題或工作流程
- **提交前驗證**：執行 `node validate-collections.js` 以確保您的清單有效

## 提交您的貢獻

1. **Fork 此專案**
2. **建立新的分支**用於您的貢獻
3. **新增您的指引、提示詞檔案、聊天模式或集合**,遵循上述準則
4. **執行更新腳本**：執行 `npm start` 以使用您的新檔案更新 README（如果您尚未執行,請先執行 `npm install`）
   - GitHub Actions 工作流程將驗證此步驟是否正確執行
   - 如果執行腳本後會修改 README.md,PR 檢查將失敗並顯示註解說明所需的變更
5. **提交 pull request**,包含：
   - 清楚描述您貢獻內容的標題
   - 您的指引/提示詞功能的簡短說明
   - 任何相關的情境或使用說明

**注意**：您的貢獻一旦合併,您將自動加入我們的[貢獻者](./README.md#contributors-)區塊！我們使用 [all-contributors](https://github.com/all-contributors/all-contributors) 來表彰對專案的各種貢獻。

## 我們接受什麼

我們歡迎涵蓋任何技術、框架或開發實踐的貢獻,只要能協助開發者更有效地使用 GitHub Copilot。包括：

- 程式語言與框架
- 開發方法論與最佳實踐
- 架構模式與設計原則
- 測試策略與品質保證
- DevOps 與部署實踐
- 無障礙與包容性設計
- 效能最佳化技術

## 我們不接受什麼

為了維護一個安全、負責且具建設性的社群,我們**不會接受**以下貢獻：

- **違反負責任 AI 原則**：試圖規避 Microsoft/GitHub 負責任 AI 準則或提倡有害 AI 使用的內容
- **危害安全性**：旨在繞過安全性政策、利用漏洞或削弱系統安全性的指引
- **促成惡意活動**：意圖危害其他系統、使用者或組織的內容
- **利用弱點**：利用其他平台或服務漏洞的指引
- **推廣有害內容**：可能導致產生有害、歧視性或不當內容的引導
- **規避平台政策**：試圖繞過 GitHub、Microsoft 或其他平台服務條款的內容

## 品質準則

- **明確具體**：通用的指引不如具體、可執行的引導有用
- **測試您的內容**：確保您的指引或提示詞能與 GitHub Copilot 良好配合
- **遵循慣例**：使用一致的格式與命名
- **保持專注**：每個檔案應針對特定的技術、框架或使用情境
- **清楚撰寫**：使用簡單、直接的語言
- **推廣最佳實踐**：鼓勵安全、可維護與符合倫理的開發實踐

## 貢獻者表彰

本專案使用 [all-contributors](https://github.com/all-contributors/all-contributors) 來表彰貢獻者。當您做出貢獻時,您將自動出現在我們的貢獻者清單中！

我們歡迎各種類型的貢獻,包括：

- 📝 文件改進
- 💻 程式碼貢獻
- 🐛 錯誤回報與修正
- 🎨 設計改進
- 💡 想法與建議
- 🤔 回答問題
- 📢 推廣專案

您的貢獻有助於讓這項資源對整個 GitHub Copilot 社群更加完善！

## 行為準則

請注意,本專案發布時包含[貢獻者行為準則](CODE_OF_CONDUCT.md)。參與本專案即表示您同意遵守其條款。

## 授權條款

對本專案做出貢獻,即表示您同意您的貢獻將採用 MIT License 授權。
