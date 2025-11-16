---
mode: 'agent'
description: '為 AI 優化的決策文件建立架構決策記錄 (ADR) 文件。'
tools: ['changes', 'search/codebase', 'edit/editFiles', 'extensions', 'fetch', 'githubRepo', 'openSimpleBrowser', 'problems', 'runTasks', 'search', 'search/searchResults', 'runCommands/terminalLastCommand', 'runCommands/terminalSelection', 'testFailure', 'usages', 'vscodeAPI']
---
# 建立架構決策記錄

為 `${input:DecisionTitle}` 建立 ADR 文件，使用針對 AI 處理和人類可讀性優化的結構化格式。

## 輸入

- **Context**: `${input:Context}`
- **Decision**: `${input:Decision}`
- **Alternatives**: `${input:Alternatives}`
- **Stakeholders**: `${input:Stakeholders}`

## 輸入驗證
如果未提供任何必需的輸入或無法從對話記錄中確定，請在繼續生成 ADR 之前要求使用者提供缺失的資訊。

## 要求

- 使用精確、明確的語言
- 遵循帶有前言的標準化 ADR 格式
- 包括正面和負面的後果
- 記錄備選方案及拒絕理由
- 為機器解析和人工參考建立結構
- 對多項目章節使用編碼項目符號（3-4 字母代碼 + 3 位數字）

ADR 必須使用命名慣例儲存在 `/docs/adr/` 目錄中：`adr-NNNN-[title-slug].md`，其中 NNNN 是下一個順序的 4 位數字（例如 `adr-0001-database-selection.md`）。

## 必需的文件結構

文件檔案必須遵循以下範本，確保所有章節都正確填寫。markdown 的前言應該按照以下範例正確結構化：

```md
---
title: "ADR-NNNN: [決策標題]"
status: "Proposed"
date: "YYYY-MM-DD"
authors: "[利害關係人姓名/角色]"
tags: ["architecture", "decision"]
supersedes: ""
superseded_by: ""
---

# ADR-NNNN: [決策標題]

## 狀態

**Proposed** | Accepted | Rejected | Superseded | Deprecated

## 背景

[問題陳述、技術限制、業務需求以及需要此決策的環境因素。]

## 決策

[所選解決方案及明確的選擇理由。]

## 後果

### 正面

- **POS-001**: [有益的結果和優勢]
- **POS-002**: [效能、可維護性、可擴展性的改進]
- **POS-003**: [與架構原則的一致性]

### 負面

- **NEG-001**: [取捨、限制、缺點]
- **NEG-002**: [引入的技術債務或複雜性]
- **NEG-003**: [風險和未來挑戰]

## 考慮的備選方案

### [備選方案 1 名稱]

- **ALT-001**: **描述**: [簡要的技術描述]
- **ALT-002**: **拒絕原因**: [為什麼未選擇此選項]

### [備選方案 2 名稱]

- **ALT-003**: **描述**: [簡要的技術描述]
- **ALT-004**: **拒絕原因**: [為什麼未選擇此選項]

## 實作注意事項

- **IMP-001**: [關鍵實作考量]
- **IMP-002**: [遷移或推出策略（如適用）]
- **IMP-003**: [監控和成功標準]

## 參考資料

- **REF-001**: [相關 ADR]
- **REF-002**: [外部文件]
- **REF-003**: [引用的標準或框架]
```
