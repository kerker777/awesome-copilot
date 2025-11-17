---
mode: 'agent'
description: 'Create time-boxed technical spike documents for researching and resolving critical development decisions before implementation.'
tools: ['runCommands', 'runTasks', 'edit', 'search', 'extensions', 'usages', 'vscodeAPI', 'think', 'problems', 'changes', 'testFailure', 'openSimpleBrowser', 'fetch', 'githubRepo', 'todos', 'Microsoft Docs', 'search']
---

# 建立技術尖峰文件

建立有時間限制的技術尖峰文件，用於研究關鍵問題，這些問題必須在開發進行前解決。每個尖峰專注於特定的技術決策，並具有明確的交付成果和時間表。

## 文件結構

在 `${input:FolderPath|docs/spikes}` 目錄中建立個別文件。使用以下模式命名每個文件：`[category]-[short-description]-spike.md`（例如：`api-copilot-integration-spike.md`、`performance-realtime-audio-spike.md`）。

```md
---
title: "${input:SpikeTitle}"
category: "${input:Category|Technical}"
status: "🔴 Not Started"
priority: "${input:Priority|High}"
timebox: "${input:Timebox|1 week}"
created: [YYYY-MM-DD]
updated: [YYYY-MM-DD]
owner: "${input:Owner}"
tags: ["technical-spike", "${input:Category|technical}", "research"]
---

# ${input:SpikeTitle}

## Summary

**Spike Objective:** [Clear, specific question or decision that needs resolution]

**Why This Matters:** [Impact on development/architecture decisions]

**Timebox:** [How much time allocated to this spike]

**Decision Deadline:** [When this must be resolved to avoid blocking development]

## Research Question(s)

**Primary Question:** [Main technical question that needs answering]

**Secondary Questions:**

- [Related question 1]
- [Related question 2]
- [Related question 3]

## Investigation Plan

### Research Tasks

- [ ] [Specific research task 1]
- [ ] [Specific research task 2]
- [ ] [Specific research task 3]
- [ ] [Create proof of concept/prototype]
- [ ] [Document findings and recommendations]

### Success Criteria

**This spike is complete when:**

- [ ] [Specific criteria 1]
- [ ] [Specific criteria 2]
- [ ] [Clear recommendation documented]
- [ ] [Proof of concept completed (if applicable)]

## Technical Context

**Related Components:** [List system components affected by this decision]

**Dependencies:** [What other spikes or decisions depend on resolving this]

**Constraints:** [Known limitations or requirements that affect the solution]

## Research Findings

### Investigation Results

[Document research findings, test results, and evidence gathered]

### Prototype/Testing Notes

[Results from any prototypes, spikes, or technical experiments]

### External Resources

- [Link to relevant documentation]
- [Link to API references]
- [Link to community discussions]
- [Link to examples/tutorials]

## Decision

### Recommendation

[Clear recommendation based on research findings]

### Rationale

[Why this approach was chosen over alternatives]

### Implementation Notes

[Key considerations for implementation]

### Follow-up Actions

- [ ] [Action item 1]
- [ ] [Action item 2]
- [ ] [Update architecture documents]
- [ ] [Create implementation tasks]

## Status History

| Date   | Status         | Notes                      |
| ------ | -------------- | -------------------------- |
| [Date] | 🔴 Not Started | Spike created and scoped   |
| [Date] | 🟡 In Progress | Research commenced         |
| [Date] | 🟢 Complete    | [Resolution summary]       |

---

_Last updated: [Date] by [Name]_
```

## 技術尖峰的類別

### API 整合

- 第三方 API 能力與限制
- 整合模式與認證
- 速率限制與性能特性

### 架構與設計

- 系統架構決策
- 設計模式適用性
- 元件交互模型

### 性能與可擴展性

- 性能需求與約束
- 可擴展性瓶頸與解決方案
- 資源利用模式

### 平台與基礎設施

- 平台能力與限制
- 基礎設施需求
- 部署與託管考量

### 安全與合規性

- 安全需求與實作
- 合規性約束
- 認證與授權方案

### 使用者體驗

- 使用者互動模式
- 無障礙需求
- 介面設計決策

## 文件命名慣例

使用描述性的kebab-case名稱，表明類別與特定的未知項目：

**API/整合範例：**

- `api-copilot-chat-integration-spike.md`
- `api-azure-speech-realtime-spike.md`
- `api-vscode-extension-capabilities-spike.md`

**性能範例：**

- `performance-audio-processing-latency-spike.md`
- `performance-extension-host-limitations-spike.md`
- `performance-webrtc-reliability-spike.md`

**架構範例：**

- `architecture-voice-pipeline-design-spike.md`
- `architecture-state-management-spike.md`
- `architecture-error-handling-strategy-spike.md`

## AI 代理的最佳做法

1. **每個尖峰一個問題：** 每個文件專注於單一技術決策或研究問題

2. **有時間限制的研究：** 為每個尖峰定義具體的時間限制與交付成果

3. **基於證據的決策：** 要求具體的證據（測試、原型、文件）才能標記為完成

4. **清晰的建議：** 文件化具體的建議與實作的理由

5. **依賴項追蹤：** 識別尖峰如何相互關聯以及如何影響專案決策

6. **以結果為導向：** 每個尖峰都必須產生可操作的決策或建議

## 研究策略

### 第 1 階段：信息收集

1. **搜尋現有文件** 使用搜尋/取得工具
2. **分析代碼庫** 找出現有的模式與約束
3. **研究外部資源** (API、程式庫、範例)

### 第 2 階段：驗證與測試

1. **建立專注的原型** 測試具體的假設
2. **執行針對性的實驗** 驗證假設
3. **文件化測試結果** 附帶支持性證據

### 第 3 階段：決策與文件

1. **合成研究結果** 形成清晰的建議
2. **文件化實作指導** 供開發團隊使用
3. **建立後續任務** 供實作使用

## 工具使用

- **search/searchResults：** 研究現有的解決方案與文件
- **fetch/githubRepo：** 分析外部 API、程式庫與範例
- **codebase：** 瞭解現有的系統約束與模式
- **runTasks：** 執行原型與驗證測試
- **editFiles：** 更新研究進度與結果
- **vscodeAPI：** 測試 VS Code 擴充套件的能力與限制

專注於有時間限制的研究，解決關鍵技術決策並消除開發進度的阻礙。
