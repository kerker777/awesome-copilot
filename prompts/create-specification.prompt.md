---
mode: 'agent'
description: 'Create a new specification file for the solution, optimized for Generative AI consumption.'
tools: ['changes', 'search/codebase', 'edit/editFiles', 'extensions', 'fetch', 'githubRepo', 'openSimpleBrowser', 'problems', 'runTasks', 'search', 'search/searchResults', 'runCommands/terminalLastCommand', 'runCommands/terminalSelection', 'testFailure', 'usages', 'vscodeAPI']
---
# 建立規格說明書

你的目標是為 `${input:SpecPurpose}` 建立一份新的規格說明書。

規格說明書必須以清晰、明確且結構化的方式定義解決方案元件的需求、限制條件和介面，使其能被生成式 AI 有效利用。遵循既定的文件編寫標準，確保內容具可讀性並具獨立性。

## AI 友善規格說明書的最佳實踐

- 使用精確、明確且不含歧義的語言。
- 清楚區分需求、限制條件與建議。
- 使用結構化格式（標題、列表、表格）便於解析。
- 避免使用成語、隱喻或依賴背景知識的參照。
- 定義所有縮寫詞及領域專用術語。
- 在適當處包含範例和邊界案例。
- 確保文件具獨立性，不依賴外部背景知識。

規格說明書應儲存在 [/spec/](/spec/) 目錄中，並依照以下命名慣例命名：`spec-[a-z0-9-]+.md`，其中名稱應描述規格說明書的內容，並以高層級的目的開頭，該目的應為下列其中之一：[schema、tool、data、infrastructure、process、architecture 或 design]。

規格說明書檔案必須以格式正確的 Markdown 進行編寫。

規格說明書檔案必須遵循下列範本，確保所有章節都適當地被填入。Markdown 的前置資料應按照以下範例正確地結構化：

```md
---
title: [簡潔的標題，描述規格說明書的重點]
version: [選用：例如 1.0、日期]
date_created: [YYYY-MM-DD]
last_updated: [選用：YYYY-MM-DD]
owner: [選用：負責此規格說明書的團隊/個人]
tags: [選用：相關標籤或分類列表，例如 `infrastructure`、`process`、`design`、`app` 等]
---

# 簡介

[規格說明書的簡短簡介及其預期達成的目標。]

## 1. 目的與範圍

[清楚、簡潔地說明規格說明書的目的及其應用範圍。陳述預期的使用者及任何假設。]

## 2. 定義

[列出並定義本規格說明書中使用的所有縮寫詞、簡稱及領域專用術語。]

## 3. 需求、限制條件與方針

[明確列出所有需求、限制條件、規則及方針。使用項目符號或表格以提高清晰度。]

- **REQ-001**: 需求 1
- **SEC-001**: 安全性需求 1
- **[3 LETTERS]-001**: 其他需求 1
- **CON-001**: 限制條件 1
- **GUD-001**: 方針 1
- **PAT-001**: 要遵循的模式 1

## 4. 介面與資料契約

[描述介面、API、資料契約或整合點。使用表格或程式碼區塊以說明模式和範例。]

## 5. 驗收標準

[使用「Given-When-Then」格式定義清晰、可測試的驗收標準（如適用）。]

- **AC-001**: Given [背景條件]、When [動作]、Then [預期結果]
- **AC-002**: 系統應在 [條件] 時 [執行特定行為]
- **AC-003**: [視需要追加驗收標準]

## 6. 測試自動化策略

[定義測試方法、框架及自動化需求。]

- **測試層級**: 單元測試、整合測試、端對端測試
- **框架**: MSTest、FluentAssertions、Moq（用於 .NET 應用程式）
- **測試資料管理**: [測試資料建立和清理的方法]
- **CI/CD 整合**: [GitHub Actions 流程中的自動化測試]
- **涵蓋範圍需求**: [最低程式碼涵蓋範圍閾值]
- **效能測試**: [負載及效能測試的方法]

## 7. 理由與背景

[解釋需求、限制條件及方針背後的推理。提供設計決策的背景脈絡。]

## 8. 依賴項與外部整合

[定義此規格說明書所需的外部系統、服務和架構依賴項。重點應放在所需的「內容」而非「實現方式」。除非代表架構限制條件，否則應避免指定特定的套件或程式庫版本。]

### 外部系統
- **EXT-001**: [外部系統名稱] - [目的和整合類型]

### 第三方服務
- **SVC-001**: [服務名稱] - [必需的功能和 SLA 需求]

### 基礎設施依賴項
- **INF-001**: [基礎設施元件] - [需求和限制條件]

### 資料依賴項
- **DAT-001**: [外部資料來源] - [格式、頻率和存取需求]

### 技術平台依賴項
- **PLT-001**: [平台/執行時期需求] - [版本限制及理由]

### 合規性依賴項
- **COM-001**: [監管或合規需求] - [對實現的影響]

**注意**: 此章節應重點關注架構和業務依賴項，而非特定套件的實現。例如，應指定「OAuth 2.0 驗證程式庫」而非「Microsoft.AspNetCore.Authentication.JwtBearer v6.0.1」。

## 9. 範例與邊界案例

    ```code
    // Code snippet or data example demonstrating the correct application of the guidelines, including edge cases
    ```

## 10. 驗證標準

[列出必須滿足的標準或測試，以符合此規格說明書的規定。]

## 11. 相關規格說明書 / 進一步閱讀

[連結至相關的規格說明書 1]
[連結至相關的外部文件]

```
