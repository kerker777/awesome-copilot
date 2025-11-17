---
mode: 'agent'
description: 'Create a new implementation plan file for new features, refactoring existing code or upgrading packages, design, architecture or infrastructure.'
tools: ['changes', 'search/codebase', 'edit/editFiles', 'extensions', 'fetch', 'githubRepo', 'openSimpleBrowser', 'problems', 'runTasks', 'search', 'search/searchResults', 'runCommands/terminalLastCommand', 'runCommands/terminalSelection', 'testFailure', 'usages', 'vscodeAPI']
---
# 建立實作計畫

## 主要指示

您的目標是為 `${input:PlanPurpose}` 建立新的實作計畫檔案。您的輸出必須是機器可讀的、確定性的，並結構化以供其他 AI 系統或人類自主執行。

## 執行背景

此提示設計用於 AI 到 AI 的通信和自動化處理。所有指示必須按字面意思解釋，並系統地執行，毋需人類解釋或澄清。

## 核心要求

- 生成完全可由 AI 代理或人類執行的實作計畫
- 使用確定性語言，零歧義
- 將所有內容結構化以供自動化解析和執行
- 確保完整的自包含性，無需外部相依性以理解

## 計畫結構要求

計畫必須由包含可執行任務的離散、原子性階段組成。每個階段必須由 AI 代理或人類獨立處理，除非明確聲明跨階段相依性。

## 階段架構

- 每個階段必須有可衡量的完成標準
- 階段內的任務必須可並行執行，除非指定了相依性
- 所有任務描述必須包含特定的檔案路徑、函數名稱和精確實作詳情
- 任何任務都不應需要人類解釋或決策

## AI 最佳化的實作標準

- 使用明確、無歧義的語言，毋需解釋
- 將所有內容結構化為機器可解析的格式（表格、列表、結構化資料）
- 包含特定的檔案路徑、行號和精確代碼參考（適用時）
- 明確定義所有變數、常數和配置值
- 在每個任務描述中提供完整的背景資訊
- 為所有識別符號使用標準化前綴（REQ-、TASK-等）
- 包含可以自動驗證的驗證標準

## 輸出檔案規範

- 將實作計畫檔案儲存在 `/plan/` 目錄
- 使用命名慣例：`[purpose]-[component]-[version].md`
- 目的前綴：`upgrade|refactor|feature|data|infrastructure|process|architecture|design`
- 範例：`upgrade-system-command-4.md`、`feature-auth-module-1.md`
- 檔案必須是有效的 Markdown，並具有正確的前置資訊結構

## 強制性模板結構

所有實作計畫必須嚴格遵守以下模板。每個部分都是必需的，必須填入特定、可執行的內容。AI 代理必須在執行前驗證模板合規性。

## 模板驗證規則

- 所有前置資訊欄位必須存在且格式正確
- 所有部分標題必須完全匹配（區分大小寫）
- 所有識別符號前綴必須遵循指定的格式
- 表格必須包含所有必需的欄位
- 最終輸出中不得保留任何佔位符文字

## 狀態

實作計畫的狀態必須在前置資訊中清楚定義，並且必須反映計畫的當前狀態。狀態可以是下列之一（括號中的 status_color）：`Completed`（亮綠色徽章）、`In progress`（黃色徽章）、`Planned`（藍色徽章）、`Deprecated`（紅色徽章）或 `On Hold`（橙色徽章）。它也應在介紹部分顯示為徽章。

```md
---
goal: [簡潔的標題，描述套件實作計畫的目標]
version: [選用：例如 1.0、日期]
date_created: [YYYY-MM-DD]
last_updated: [選用：YYYY-MM-DD]
owner: [選用：負責此規格的團隊/個人]
status: 'Completed'|'In progress'|'Planned'|'Deprecated'|'On Hold'
tags: [選用：相關標籤或類別列表，例如 `feature`、`upgrade`、`chore`、`architecture`、`migration`、`bug` 等]
---

# 介紹

![Status: <status>](https://img.shields.io/badge/status-<status>-<status_color>)

[計畫的簡短而簡潔的介紹及其旨在達成的目標。]

## 1. 要求與限制

[明確列出影響計畫且限制其實作方式的所有要求和限制。使用項目符號或表格以清楚呈現。]

- **REQ-001**: 要求 1
- **SEC-001**: 安全要求 1
- **[3 LETTERS]-001**: 其他要求 1
- **CON-001**: 限制 1
- **GUD-001**: 指南 1
- **PAT-001**: 要遵循的模式 1

## 2. 實作步驟

### 實作階段 1

- GOAL-001: [描述此階段的目標，例如「實作功能 X」、「重構模組 Y」等。]

| 任務 | 描述 | 已完成 | 日期 |
|------|------|--------|------|
| TASK-001 | 任務 1 的描述 | ✅ | 2025-04-25 |
| TASK-002 | 任務 2 的描述 | |  |
| TASK-003 | 任務 3 的描述 | |  |

### 實作階段 2

- GOAL-002: [描述此階段的目標，例如「實作功能 X」、「重構模組 Y」等。]

| 任務 | 描述 | 已完成 | 日期 |
|------|------|--------|------|
| TASK-004 | 任務 4 的描述 | |  |
| TASK-005 | 任務 5 的描述 | |  |
| TASK-006 | 任務 6 的描述 | |  |

## 3. 替代方案

[所考慮的任何替代方法的項目符號列表，以及為什麼未選擇它們。這有助於為所選方法提供背景和基本理由。]

- **ALT-001**: 替代方法 1
- **ALT-002**: 替代方法 2

## 4. 相依性

[列出需要處理的任何相依性，例如計畫所依賴的程式庫、架構或其他元件。]

- **DEP-001**: 相依性 1
- **DEP-002**: 相依性 2

## 5. 檔案

[列出將受功能或重構任務影響的檔案。]

- **FILE-001**: 檔案 1 的描述
- **FILE-002**: 檔案 2 的描述

## 6. 測試

[列出需要實作的測試，以驗證功能或重構任務。]

- **TEST-001**: 測試 1 的描述
- **TEST-002**: 測試 2 的描述

## 7. 風險與假設

[列出與計畫實作相關的任何風險或假設。]

- **RISK-001**: 風險 1
- **ASSUMPTION-001**: 假設 1

## 8. 相關規格 / 進階閱讀

[相關規格 1 的連結]
[相關外部文件的連結]
```
