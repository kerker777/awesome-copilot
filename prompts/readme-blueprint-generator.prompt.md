---
description: '智慧 README.md 產生提示，分析專案文件結構並建立全面的儲存庫文件。掃描 .github/copilot 目錄檔案和 copilot-instructions.md 以擷取專案資訊、技術堆疊、架構、開發工作流程、編碼標準和測試方法，同時產生具有適當格式、交叉參考和以開發人員為中心內容的良好結構化 markdown 文件。'

mode: 'agent'
---

# README 產生器提示

透過分析 .github/copilot 目錄中的文件檔案和 copilot-instructions.md 檔案，為此儲存庫產生全面的 README.md。請依照以下步驟：

1. 掃描 .github/copilot 資料夾中的所有檔案，例如：
   - Architecture
   - Code_Exemplars
   - Coding_Standards
   - Project_Folder_Structure
   - Technology_Stack
   - Unit_Tests
   - Workflow_Analysis

2. 同時審查 .github 資料夾中的 copilot-instructions.md 檔案

3. 建立具有以下區段的 README.md：

## 專案名稱和描述
- 從文件中擷取專案名稱和主要目的
- 包含專案功能的簡潔描述

## 技術堆疊
- 列出使用的主要技術、語言和框架
- 可用時包含版本資訊
- 主要從 Technology_Stack 檔案取得此資訊

## 專案架構
- 提供架構的高階概述
- 如果文件中有描述，考慮包含簡單的圖表
- 從 Architecture 檔案取得

## 入門指南
- 根據技術堆疊包含安裝指示
- 新增設定和配置步驟
- 包含任何先決條件

## 專案結構
- 資料夾組織的簡要概述
- 從 Project_Folder_Structure 檔案取得

## 主要功能
- 列出專案的主要功能和特性
- 從各種文件檔案中擷取

## 開發工作流程
- 總結開發流程
- 可用時包含分支策略的資訊
- 從 Workflow_Analysis 檔案取得

## 編碼標準
- 總結關鍵的編碼標準和慣例
- 從 Coding_Standards 檔案取得

## 測試
- 解釋測試方法和工具
- 從 Unit_Tests 檔案取得

## 貢獻
- 貢獻專案的指南
- 參考任何程式碼範例作為指導
- 從 Code_Exemplars 和 copilot-instructions 取得

## 授權
- 可用時包含授權資訊

使用適當的 Markdown 格式化 README，包括：
- 清晰的標題和副標題
- 適當時使用程式碼區塊
- 更好可讀性的清單
- 其他文件檔案的連結
- 如果資訊可用，使用建置狀態、版本等的徽章

保持 README 簡潔但資訊豐富，專注於新開發人員或使用者需要了解的專案資訊。
