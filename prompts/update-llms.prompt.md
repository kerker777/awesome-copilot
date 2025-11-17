---
mode: 'agent'
description: 'Update the llms.txt file in the root folder to reflect changes in documentation or specifications following the llms.txt specification at https://llmstxt.org/'
tools: ['changes', 'search/codebase', 'edit/editFiles', 'extensions', 'fetch', 'githubRepo', 'openSimpleBrowser', 'problems', 'runTasks', 'search', 'search/searchResults', 'runCommands/terminalLastCommand', 'runCommands/terminalSelection', 'testFailure', 'usages', 'vscodeAPI']
---
# 更新 LLMs.txt 檔案

更新版本庫根目錄中現有的 `llms.txt` 檔案，以反映文件、規格或版本庫結構的變更。此檔案根據 llms.txt 規格 (https://llmstxt.org/) 為大型語言模型 (LLM) 提供高層級引導，指示它們在何處找到相關內容以了解版本庫的目的和規格。

## 主要指令

更新現有的 `llms.txt` 檔案，以保持準確性並符合 llms.txt 規格，同時反映當前版本庫的結構和內容。此檔案必須保持針對 LLM 消費進行最佳化，同時保持人類可讀。

## 分析與規劃階段

在更新 `llms.txt` 檔案之前，你必須完成徹底的分析：

### 第 1 步：檢視現有檔案與規格
- 讀取現有的 `llms.txt` 檔案，了解當前的結構
- 檢視 https://llmstxt.org/ 的正式規格，以確保持續符合規格
- 根據版本庫的變更識別可能需要更新的領域

### 第 2 步：版本庫結構分析
- 使用適當的工具檢查當前版本庫結構
- 將當前結構與現有 `llms.txt` 中的文件進行比較
- 識別應包含的新目錄、檔案或文件
- 記錄需要更新的任何已移除或重新定位的檔案

### 第 3 步：內容發現與變更偵測
- 識別新的 README 檔案及其位置
- 尋找新的文件檔案 (`/docs/`、`/spec/` 等中的 `.md` 檔案)
- 定位新的規格檔案及其用途
- 探索新的組態檔案及其相關性
- 尋找新的範例檔案和程式碼樣本
- 識別現有文件結構的任何變更

### 第 4 步：建立更新計畫
根據你的分析，建立包含以下內容的結構化計畫：
- 保持準確性所需的變更
- 要新增至 llms.txt 的新檔案
- 要移除或更新的過期參考
- 保持清晰度的組織改進

## 實施要求

### 格式符合性
更新後的 `llms.txt` 檔案必須按照規格維持此確切結構：

1. **H1 標題**：包含版本庫/專案名稱的單行（必需）
2. **引用摘要**：以區塊引用格式的簡短說明（選用但建議）
3. **其他詳細資訊**：零個或多個沒有標題的 markdown 章節供參考
4. **檔案列表章節**：零個或多個包含標記連結清單的 H2 章節

### 內容要求

#### 必需元素
- **專案名稱**：作為 H1 的清晰、詳細的標題
- **摘要**：簡潔的區塊引用，說明版本庫的目的
- **關鍵檔案**：按類別組織的必要檔案（H2 章節）

#### 檔案連結格式
每個檔案連結必須遵循：`[descriptive-name](relative-url): optional description`

#### 章節組織
將檔案組織為邏輯 H2 章節，例如：
- **文件**：核心文件檔案
- **規格**：技術規格和要求
- **範例**：樣本程式碼和使用方式範例
- **組態**：設定和組態檔案
- **選用**：次要檔案（特殊含義 - 可針對較短的內容跳過）

### 內容指導方針

#### 語言與風格
- 使用簡潔、清晰、無歧義的語言
- 避免沒有解釋的術語
- 為人類和 LLM 讀者撰寫
- 在說明中具體且具有資訊性

#### 檔案選擇標準
包含以下檔案：
- 說明版本庫的目的和範圍
- 提供必要的技術文件
- 展示使用方式範例和模式
- 定義介面和規格
- 包含組態和設定說明

排除以下檔案：
- 純粹的實作細節
- 包含冗餘資訊
- 組建成品或產生的內容
- 與了解專案無關

## 執行步驟

### 第 1 步：當前狀態分析
1. 徹底讀取現有的 `llms.txt` 檔案
2. 完全檢查當前版本庫結構
3. 將現有檔案參考與實際版本庫內容進行比較
4. 識別過期、遺失或不正確的參考
5. 記錄當前檔案的任何結構問題

### 第 2 步：內容規劃
1. 判斷主要目的陳述是否需要更新
2. 檢視並在需要時更新摘要區塊引用
3. 為新檔案和目錄規劃新增項目
4. 為過期或已移動的內容規劃移除項目
5. 如果需要，重新組織章節以獲得更好的清晰度

### 第 3 步：檔案更新
1. 更新版本庫根目錄中的現有 `llms.txt` 檔案
2. 保持符合確切的格式規格
3. 新增具有適當說明的新檔案參考
4. 移除或更新過期的參考
5. 確保所有連結都是有效的相對路徑

### 第 4 步：驗證
1. 驗證繼續符合 https://llmstxt.org/ 規格
2. 檢查所有連結是否有效且可存取
3. 確保檔案仍然充當有效的 LLM 導覽工具
4. 確認檔案保持人類和機器可讀

## 品質保證

### 格式驗證
- ✅ 帶有專案名稱的 H1 標題
- ✅ 區塊引用摘要（如果包含）
- ✅ 檔案列表的 H2 章節
- ✅ 適當的 markdown 連結格式
- ✅ 沒有斷開的或無效的連結
- ✅ 整個頁面的一致格式

### 內容驗證
- ✅ 清晰、無歧義的語言
- ✅ 必要檔案的全面涵蓋
- ✅ 內容的邏輯組織
- ✅ 適當的檔案說明
- ✅ 充當有效的 LLM 導覽工具

### 規格符合性
- ✅ 完全遵循 https://llmstxt.org/ 格式
- ✅ 使用必需的 markdown 結構
- ✅ 適當實施選用章節
- ✅ 檔案位於版本庫根目錄 (`/llms.txt`)

## 更新策略

### 新增程序
新增新內容時：
1. 為新檔案識別適當的章節
2. 為連結建立清晰、詳細的名稱
3. 撰寫簡潔但有資訊的說明
4. 保持章節內的按字母順序或邏輯順序
5. 考慮是否需要為新內容型態建立新章節

### 移除程序
移除過期內容時：
1. 驗證檔案是否真的被移除或重新定位
2. 檢查重新定位的檔案是否應更新而不是移除
3. 如果章節變空，移除整個章節
4. 如果需要，更新交叉參考

### 重新組織程序
重新組織內容時：
1. 保持從一般到特定的邏輯流程
2. 在主要章節中保持必要的文件
3. 如果適當，將次要內容移至「選用」章節
4. 確保新組織改進 LLM 導覽

`llms.txt` 的範例結構：

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

## 成功標準

更新後的 `llms.txt` 檔案應該：
1. 準確反映當前版本庫的結構和內容
2. 保持符合 llms.txt 規格
3. 提供對必要文件的清晰導覽
4. 移除過期或不正確的參考
5. 包含新的重要檔案和文件
6. 為簡易 LLM 消費保持邏輯組織
7. 在整個頁面使用清晰、無歧義的語言
8. 繼續有效地為人類和機器讀者服務
