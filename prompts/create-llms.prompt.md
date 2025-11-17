---
mode: 'agent'
description: 'Create an llms.txt file from scratch based on repository structure following the llms.txt specification at https://llmstxt.org/'
tools: ['changes', 'search/codebase', 'edit/editFiles', 'extensions', 'fetch', 'githubRepo', 'openSimpleBrowser', 'problems', 'runTasks', 'search', 'search/searchResults', 'runCommands/terminalLastCommand', 'runCommands/terminalSelection', 'testFailure', 'usages', 'vscodeAPI']
---
# 從儲存庫結構建立 LLMs.txt 文件

在儲存庫根目錄按照官方 llms.txt 規範（https://llmstxt.org/）從頭建立新的 `llms.txt` 文件。此文件為大型語言模型（LLM）提供高階指導，說明在何處可以找到相關內容，以便理解儲存庫的用途和規範。

## 主要目標

建立一份完整的 `llms.txt` 文件，作為 LLM 理解和有效瀏覽儲存庫的進入點。文件必須符合 llms.txt 規範，並針對 LLM 使用進行最佳化，同時保持人類可讀性。

## 分析和規劃階段

在建立 `llms.txt` 文件之前，必須完成全面的分析：

### 步驟 1：檢閱 llms.txt 規範

- 檢閱官方規範（https://llmstxt.org/）以確保完全符合
- 理解所需的格式結構和指南
- 注意特定的 markdown 結構要求

### 步驟 2：儲存庫結構分析

- 使用適當的工具檢查完整的儲存庫結構
- 確認儲存庫的主要目的和範圍
- 編錄所有重要目錄及其用途
- 列出對 LLM 理解有價值的關鍵檔案

### 步驟 3：內容探索

- 辨識 README 文件及其位置
- 尋找文檔檔案（`/docs/`、`/spec/` 等目錄中的 `.md` 檔案）
- 定位規範檔案及其用途
- 發現組態檔案及其相關性
- 尋找範例檔案和程式碼範本
- 辨識任何現有的文檔結構

### 步驟 4：建立實作計畫

根據分析，建立包含以下項目的結構化計畫：

- 儲存庫用途和範圍的摘要
- 依優先度排序的 LLM 理解所需必要檔案列表
- 提供額外內容的次要檔案
- llms.txt 文件的組織結構

## 實作要求

### 格式遵循

`llms.txt` 文件必須按照規範遵循此確切結構：

1. **H1 標題**：包含儲存庫/專案名稱的單一行（必要）
2. **引言摘要**：引言格式的簡短說明（可選但推薦）
3. **其他詳細資訊**：零個或多個不含標題的 markdown 段落，用於提供內容
4. **檔案列表段落**：零個或多個包含 markdown 連結列表的 H2 段落

### 內容要求

#### 必要元素

- **專案名稱**：作為 H1 的清晰描述性標題
- **摘要**：說明儲存庫用途的簡潔引言
- **關鍵檔案**：按類別組織的必要檔案（H2 段落）

#### 檔案連結格式

每個檔案連結必須遵循：`[descriptive-name](relative-url): optional description`

#### 段落組織

將檔案組織成邏輯性的 H2 段落，例如：

- **文檔**：核心文檔檔案
- **規範**：技術規範和要求
- **範例**：範本程式碼和使用範例
- **組態**：設定和組態檔案
- **可選**：次要檔案（特殊含義 - 在較短的內容中可以跳過）

### 內容指南

#### 語言和風格

- 使用簡潔、清晰、不含歧義的語言
- 避免未解釋的術語
- 編寫供人類和 LLM 讀者使用的內容
- 在描述中具體且提供資訊

#### 檔案選擇條件

包含以下檔案：
- 說明儲存庫的用途和範圍
- 提供必要的技術文檔
- 顯示使用範例和模式
- 定義介面和規範
- 包含組態和設定指示

排除以下檔案：
- 純粹的實作細節
- 包含冗餘資訊
- 為建置成品或生成內容
- 與理解專案無關

## 執行步驟

### 步驟 1：儲存庫分析

1. 完整檢查儲存庫結構
2. 閱讀主要 README.md 以理解專案
3. 辨識所有文檔目錄和檔案
4. 編錄規範檔案及其用途
5. 尋找範例檔案和組態檔案

### 步驟 2：內容規劃

1. 確定主要目的聲明
2. 為引言撰寫簡潔摘要
3. 將辨識的檔案分組成邏輯類別
4. 根據 LLM 理解的重要性排定檔案優先順序
5. 為每個檔案連結建立說明

### 步驟 3：檔案建立

1. 在儲存庫根目錄建立 `llms.txt` 檔案
2. 遵循確切的格式規範
3. 包含所有必要段落
4. 使用適當的 markdown 格式設定
5. 確保所有連結都是有效的相對路徑

### 步驟 4：驗證

1. 驗證是否符合 https://llmstxt.org/ 規範
2. 檢查所有連結是否有效且可訪問
3. 確保檔案作為有效的 LLM 瀏覽工具
4. 確認檔案既可供人類閱讀，也可供機器閱讀

## 品質保證

### 格式驗證

- ✅ 包含專案名稱的 H1 標題
- ✅ 引言摘要（如包含）
- ✅ 檔案列表的 H2 段落
- ✅ 適當的 markdown 連結格式
- ✅ 無損壞或無效連結
- ✅ 全程一致的格式設定

### 內容驗證

- ✅ 清晰、不含歧義的語言
- ✅ 全面涵蓋必要檔案
- ✅ 內容的邏輯組織
- ✅ 適當的檔案說明
- ✅ 作為有效的 LLM 瀏覽工具

### 規範遵循

- ✅ 確實遵循 https://llmstxt.org/ 格式
- ✅ 使用必要的 markdown 結構
- ✅ 適當地實作可選段落
- ✅ 檔案位於儲存庫根目錄（`/llms.txt`）

## 範例結構範本

```txt
# [Repository Name]

> [Concise description of the repository's purpose and scope]

[Optional additional context paragraphs without headings]

## Documentation

- [Main README](README.md): Primary project documentation and getting started guide
- [Contributing Guide](CONTRIBUTING.md): Guidelines for contributing to the project
- [Code of Conduct](CODE_OF_CONDUCT.md): Community guidelines and expectations

## Specifications

- [Technical Specification](spec/technical-spec.md): Detailed technical requirements and constraints
- [API Specification](spec/api-spec.md): Interface definitions and data contracts

## Examples

- [Basic Example](examples/basic-usage.md): Simple usage demonstration
- [Advanced Example](examples/advanced-usage.md): Complex implementation patterns

## Configuration

- [Setup Guide](docs/setup.md): Installation and configuration instructions
- [Deployment Guide](docs/deployment.md): Production deployment guidelines

## Optional

- [Architecture Documentation](docs/architecture.md): Detailed system architecture
- [Design Decisions](docs/decisions.md): Historical design decision records
```

## 成功條件

建立的 `llms.txt` 文件應該：
1. 讓 LLM 快速理解儲存庫的用途
2. 提供到必要文檔的清晰瀏覽
3. 確實遵循官方 llms.txt 規範
4. 全面但簡潔
5. 有效地為人類和機器讀者服務
6. 包含專案理解的所有關鍵檔案
7. 全程使用清晰、不含歧義的語言
8. 以邏輯方式組織內容以便於使用
