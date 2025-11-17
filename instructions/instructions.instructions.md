---
description: 'Guidelines for creating high-quality custom instruction files for GitHub Copilot'
applyTo: '**/*.instructions.md'
---

# 自訂指令檔案指引

說明如何建立有效且可維護的自訂指令檔案，用以指導 GitHub Copilot 產生特定領域程式碼和遵循專案慣例。

## 專案背景

- 目標受眾：與特定領域程式碼互動的開發人員和 GitHub Copilot
- 檔案格式：Markdown 搭配 YAML frontmatter
- 檔案命名慣例：小寫加連字符（例如 `react-best-practices.instructions.md`）
- 位置：`.github/instructions/` 目錄
- 用途：為程式碼產生、審查和文件提供情境感知的指引

## 必要的 Frontmatter

每個指令檔案都必須包含具有以下欄位的 YAML frontmatter：

```yaml
---
description: 'Brief description of the instruction purpose and scope'
applyTo: 'glob pattern for target files (e.g., **/*.ts, **/*.py)'
---
```

### Frontmatter 指引

- **description**：單引號字串，1-500 字元，清楚說明用途
- **applyTo**：Glob 模式，指定這些指令適用於哪些檔案
  - 單一模式：`'**/*.ts'`
  - 多個模式：`'**/*.ts, **/*.tsx, **/*.js'`
  - 特定檔案：`'src/**/*.py'`
  - 所有檔案：`'**'`

## 檔案結構

結構良好的指令檔案應包含以下各節：

### 1. 標題和概覽

- 使用 `#` 標題的清晰、描述性標題
- 簡要介紹說明用途和範圍
- 可選：包含關鍵技術和版本的專案背景章節

### 2. 核心章節

根據領域將內容組織成邏輯章節：

- **通用指令**：高層級指南和原則
- **最佳做法**：建議的模式和方法
- **程式碼標準**：命名慣例、格式化、風格規則
- **架構/結構**：專案組織和設計模式
- **常見模式**：常用的實作方式
- **安全性**：安全考量（如適用）
- **效能**：最佳化指南（如適用）
- **測試**：測試標準和方法（如適用）

### 3. 範例和程式碼片段

提供具有清楚標籤的具體範例：

```markdown
### Good Example
\`\`\`language
// Recommended approach
code example here
\`\`\`

### Bad Example
\`\`\`language
// Avoid this pattern
code example here
\`\`\`
```

### 4. 驗證和確認（可選但建議）

- 用於驗證程式碼的建置指令
- 語法檢查和格式化工具
- 測試需求
- 驗證步驟

## 內容指引

### 寫作風格

- 使用清晰、簡潔的語言
- 用祈使語氣寫作（「使用」、「實作」、「避免」）
- 具體且可執行
- 避免使用模稜兩可的詞彙，如「應該」、「可能」、「或許」
- 使用項目符號和清單以增進可讀性
- 保持章節重點明確且易於掃瞄

### 最佳做法

- **具體明確**：提供具體範例而非抽象概念
- **說明原因**：在有幫助時說明建議背後的理由
- **使用表格**：用於比較選項、列舉規則或展示模式
- **包含範例**：實際程式碼片段比描述更有效
- **保持現況**：參考現行版本和最佳做法
- **連結資源**：包含官方文件和權威來源

### 包含的常見模式

1. **命名慣例**：如何命名變數、函式、類別、檔案
2. **程式碼組織**：檔案結構、模組組織、匯入順序
3. **錯誤處理**：偏好的錯誤處理模式
4. **依賴項**：如何管理和文件化依賴項
5. **註解和文件**：何時以及如何文件化程式碼
6. **版本資訊**：目標語言/框架版本

## 遵循的模式

### 項目符號和清單

```markdown
## Security Best Practices

- Always validate user input before processing
- Use parameterized queries to prevent SQL injection
- Store secrets in environment variables, never in code
- Implement proper authentication and authorization
- Enable HTTPS for all production endpoints
```

### 結構化資訊的表格

```markdown
## Common Issues

| Issue            | Solution            | Example                       |
| ---------------- | ------------------- | ----------------------------- |
| Magic numbers    | Use named constants | `const MAX_RETRIES = 3`       |
| Deep nesting     | Extract functions   | Refactor nested if statements |
| Hardcoded values | Use configuration   | Store API URLs in config      |
```

### 程式碼比較

```markdown
### Good Example - Using TypeScript interfaces
\`\`\`typescript
interface User {
  id: string;
  name: string;
  email: string;
}

function getUser(id: string): User {
  // Implementation
}
\`\`\`

### Bad Example - Using any type
\`\`\`typescript
function getUser(id: any): any {
  // Loses type safety
}
\`\`\`
```

### 條件指導

```markdown
## Framework Selection

- **For small projects**: Use Minimal API approach
- **For large projects**: Use controller-based architecture with clear separation
- **For microservices**: Consider domain-driven design patterns
```

## 應避免的模式

- **過度冗長的說明**：保持簡潔且易於掃瞄
- **過時資訊**：始終參考現行版本和做法
- **模糊的指南**：明確說明該做什麼或避免什麼
- **缺少範例**：沒有具體程式碼範例的抽象規則
- **相互矛盾的建議**：確保整個檔案的一致性
- **從文件複製貼上**：透過精煉和加入背景來增加價值

## 測試您的指令

在完成指令檔案之前：

1. **使用 Copilot 測試**：在 VS Code 中用實際提示試驗指令
2. **驗證範例**：確保程式碼範例正確且執行無誤
3. **檢查 Glob 模式**：確認 `applyTo` 模式與預期的檔案相符

## 範例結構

以下是新指令檔案的最小範例結構：

```markdown
---
description: 'Brief description of purpose'
applyTo: '**/*.ext'
---

# Technology Name Development

Brief introduction and context.

## General Instructions

- High-level guideline 1
- High-level guideline 2

## Best Practices

- Specific practice 1
- Specific practice 2

## Code Standards

### Naming Conventions
- Rule 1
- Rule 2

### File Organization
- Structure 1
- Structure 2

## Common Patterns

### Pattern 1
Description and example

\`\`\`language
code example
\`\`\`

### Pattern 2
Description and example

## Validation

- Build command: `command to verify`
- Linting: `command to lint`
- Testing: `command to test`
```

## 維護

- 當依賴項或框架更新時審查指令
- 更新範例以反映現行最佳做法
- 移除過時的模式或已棄用的功能
- 隨著社群中的新模式出現而新增
- 隨著專案結構演變保持 glob 模式的準確性

## 其他資源

- [Custom Instructions Documentation](https://code.visualstudio.com/docs/copilot/customization/custom-instructions)
- [Awesome Copilot Instructions](https://github.com/github/awesome-copilot/tree/main/instructions)
