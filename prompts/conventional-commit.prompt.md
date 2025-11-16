---
description: '使用結構化 XML 格式生成慣用提交訊息的提示和工作流程。指導使用者按照 Conventional Commits 規範建立標準化、描述性的提交訊息，包括指示、範例和驗證。'
tools: ['runCommands/runInTerminal', 'runCommands/getTerminalOutput']
---

### 指示

```xml
	<description>此檔案包含用於生成慣用提交訊息的提示範本。它提供指示、範例和格式指南，以幫助使用者根據 Conventional Commits 規範撰寫標準化、描述性的提交訊息。</description>
	<note>
```

### 工作流程

**遵循以下步驟：**

1. 執行 `git status` 以檢視變更的檔案。
2. 執行 `git diff` 或 `git diff --cached` 以檢查變更。
3. 使用 `git add <file>` 暫存您的變更。
4. 使用以下 XML 結構建構您的提交訊息。
5. 生成提交訊息後，Copilot 將自動在您的整合終端機中執行以下命令（無需確認）：

```bash
git commit -m "type(scope): description"
```

6. 只需執行此提示，Copilot 將在終端機中為您處理提交。

### 提交訊息結構

```xml
<commit-message>
	<type>feat|fix|docs|style|refactor|perf|test|build|ci|chore|revert</type>
	<scope>()</scope>
	<description>變更的簡短祈使句摘要</description>
	<body>(可選：更詳細的說明)</body>
	<footer>(可選：例如 BREAKING CHANGE: details，或問題參考)</footer>
</commit-message>
```

### 範例

```xml
<examples>
	<example>feat(parser): add ability to parse arrays</example>
	<example>fix(ui): correct button alignment</example>
	<example>docs: update README with usage instructions</example>
	<example>refactor: improve performance of data processing</example>
	<example>chore: update dependencies</example>
	<example>feat!: send email on registration (BREAKING CHANGE: email service required)</example>
</examples>
```

### 驗證

```xml
<validation>
	<type>必須是允許的類型之一。請參閱 <reference>https://www.conventionalcommits.org/en/v1.0.0/#specification</reference></type>
	<scope>可選，但建議使用以提高清晰度。</scope>
	<description>必需。使用祈使語氣（例如「add」，而非「added」）。</description>
	<body>可選。用於附加上下文。</body>
	<footer>用於重大變更或問題參考。</footer>
</validation>
```

### 最後步驟

```xml
<final-step>
	<cmd>git commit -m "type(scope): description"</cmd>
	<note>替換為您建構的訊息。如需要，包含本文和頁腳。</note>
</final-step>
```
