---
mode: 'agent'
description: 'Create comprehensive, standardized documentation for object-oriented components following industry best practices and architectural documentation standards.'
tools: ['changes', 'search/codebase', 'edit/editFiles', 'extensions', 'fetch', 'githubRepo', 'openSimpleBrowser', 'problems', 'runTasks', 'search', 'search/searchResults', 'runCommands/terminalLastCommand', 'runCommands/terminalSelection', 'testFailure', 'usages', 'vscodeAPI']
---
# 產生標準物件導向元件文件

為以下位置的物件導向元件建立全面文件：`${input:ComponentPath}`。

透過檢查所提供路徑中的程式碼來分析元件。如果是資料夾，分析所有原始碼檔案。如果是單一檔案，將其視為主要元件並分析同一目錄中的相關檔案。

## 文件標準

- DOC-001：遵循 C4 模型文件階層（背景、容器、元件、程式碼）
- DOC-002：符合 Arc42 軟體架構文件範本
- DOC-003：遵守 IEEE 1016 軟體設計說明標準
- DOC-004：使用敏捷文件原則（僅提供能增加價值的足夠文件）
- DOC-005：將開發人員和維護人員視為主要對象

## 分析指示

- ANA-001：決定路徑類型（資料夾或單一檔案）並識別主要元件
- ANA-002：檢查原始碼檔案以瞭解類別結構和繼承
- ANA-003：識別設計模式和架構決策
- ANA-004：記錄公開 API、介面和相依項目
- ANA-005：識別建立式/結構式/行為式模式
- ANA-006：記錄方法參數、傳回值、例外狀況
- ANA-007：評估效能、安全性、可靠性、可維護性
- ANA-008：推斷整合模式和資料流

## 語言特定最佳化

- LNG-001：**C#/.NET** - 非同步/等待、相依性注入、設定、處置
- LNG-002：**Java** - Spring 框架、註解、例外狀況處理、封裝
- LNG-003：**TypeScript/JavaScript** - 模組、非同步模式、型別、npm
- LNG-004：**Python** - 套件、虛擬環境、型別提示、測試

## 錯誤處理

- ERR-001：路徑不存在 - 提供正確格式指引
- ERR-002：找不到原始碼檔案 - 建議替代位置
- ERR-003：不清楚的結構 - 記錄發現並請求澄清
- ERR-004：非標準模式 - 記錄自訂方法
- ERR-005：程式碼不足 - 專注於可用資訊，突出差距

## 輸出格式

使用清晰的標題階層、程式碼區塊、表格、項目符號和適當格式生成結構良好的 Markdown，以提高可讀性和可維護性。

## 檔案位置

文件應儲存在 `/docs/components/` 目錄中，並按慣例命名：`[component-name]-documentation.md`。

## 必需的文件結構

文件檔案必須遵循以下範本，確保所有部分都適當填寫。Markdown 的前置設定應按照以下範例正確結構化：

```md
---
title: [Component Name] - Technical Documentation
component_path: `${input:ComponentPath}`
version: [Optional: e.g., 1.0, Date]
date_created: [YYYY-MM-DD]
last_updated: [Optional: YYYY-MM-DD]
owner: [Optional: Team/Individual responsible for this component]
tags: [Optional: List of relevant tags or categories, e.g., `component`,`service`,`tool`,`infrastructure`,`documentation`,`architecture` etc]
---

# [Component Name] Documentation

[A short concise introduction to the component and its purpose within the system.]

## 1. 元件概覽

### 目的/責任
- OVR-001：陳述元件的主要責任
- OVR-002：定義範圍（包含/排除的功能）
- OVR-003：描述系統背景和關係

## 2. 架構部分

- ARC-001：記錄使用的設計模式（Repository、Factory、Observer 等）
- ARC-002：列出內部和外部相依項目及其目的
- ARC-003：記錄元件互動和關係
- ARC-004：包含視覺圖表（UML 類別、序列、元件）
- ARC-005：建立顯示元件結構、關係和相依項目的 mermaid 圖表

### 元件結構和相依項目圖表

包含顯示以下內容的全面 mermaid 圖表：
- **元件結構** - 主要類別、介面及其關係
- **內部相依項目** - 元件如何在系統中互動
- **外部相依項目** - 外部程式庫、服務、資料庫、API
- **資料流** - 相依項目和互動的方向
- **繼承/組合** - 類別階層和組合關係

```mermaid
graph TD
    subgraph "Component System"
        A[Main Component] --> B[Internal Service]
        A --> C[Internal Repository]
        B --> D[Business Logic]
        C --> E[Data Access Layer]
    end

    subgraph "External Dependencies"
        F[External API]
        G[Database]
        H[Third-party Library]
        I[Configuration Service]
    end

    A --> F
    E --> G
    B --> H
    A --> I

    classDiagram
        class MainComponent {
            +property: Type
            +method(): ReturnType
            +asyncMethod(): Promise~Type~
        }
        class InternalService {
            +businessOperation(): Result
        }
        class ExternalAPI {
            <<external>>
            +apiCall(): Data
        }

        MainComponent --> InternalService
        MainComponent --> ExternalAPI
```

## 3. 介面文件

- INT-001：記錄所有公開介面和使用模式
- INT-002：建立方法/屬性參考表
- INT-003：記錄事件/回呼/通知機制

| 方法/屬性 | 目的 | 參數 | 傳回型別 | 使用備註 |
|----------|------|------|---------|--------|
| [名稱] | [目的] | [參數] | [型別] | [備註] |

## 4. 實作詳細資訊

- IMP-001：記錄主要實作類別和責任
- IMP-002：描述設定需求和初始化
- IMP-003：記錄關鍵演算法和業務邏輯
- IMP-004：注意效能特性和瓶頸

## 5. 使用範例

### 基本用法

```csharp
// Basic usage example
var component = new ComponentName();
component.DoSomething();
```

### 進階用法

```csharp
// Advanced configuration patterns
var options = new ComponentOptions();
var component = ComponentFactory.Create(options);
await component.ProcessAsync(data);
```

- USE-001：提供基本使用範例
- USE-002：展示進階設定模式
- USE-003：記錄最佳實踐和建議模式

## 6. 品質屬性

- QUA-001：安全性（驗證、授權、資料保護）
- QUA-002：效能（特性、可擴充性、資源使用）
- QUA-003：可靠性（錯誤處理、容錯、復原）
- QUA-004：可維護性（標準、測試、文件）
- QUA-005：可擴充性（擴充點、自訂選項）

## 7. 參考資訊

- REF-001：列出相依項目、版本和目的
- REF-002：完整設定選項參考
- REF-003：測試指南和模擬設定
- REF-004：故障排除（常見問題、錯誤訊息）
- REF-005：相關文件連結
- REF-006：變更歷程和遷移備註

```
