---
description: '建立和管理 awesome-copilot 集合的指南'
applyTo: 'collections/*.collection.yml'
---

# 集合開發

## 集合指示

在 awesome-copilot 儲存庫中使用集合時:

- 在提交之前始終使用 `node validate-collections.js` 驗證集合
- 遵循已建立的 YAML 架構進行集合清單
- 僅參考儲存庫中的現有文件
- 使用小寫字母、數字和連字號的描述性集合 ID
- 保持集合專注於特定工作流程或主題
- 測試所有引用的項目能夠良好協作

## 集合結構

- **必填欄位**:id、name、description、items
- **選填欄位**:tags、display
- **項目要求**:path 必須存在,kind 必須與文件副檔名匹配
- **顯示選項**:ordering(alpha/manual)、show_badge(true/false)

## 驗證規則

- 集合 ID 在所有集合中必須唯一
- 文件路徑必須存在且與項目 kind 匹配
- 標籤僅使用小寫字母、數字和連字號
- 集合必須包含 1-50 個項目
- 描述必須為 1-500 個字元

## 最佳實踐

- 組合 3-10 個相關項目以獲得最佳可用性
- 使用清晰、描述性的名稱和描述
- 新增相關標籤以提高可發現性
- 測試集合啟用的完整工作流程
- 確保項目有效地相互補充

## 文件組織

- 集合不需要文件重組
- 項目可以位於儲存庫中的任何位置
- 使用從儲存庫根目錄的相對路徑
- 維護現有目錄結構(prompts/、instructions/、chatmodes/)

## 產生過程

- 集合透過 `npm start` 自動產生 README 文件
- 在 collections/ 目錄中建立個別集合頁面
- 主要集合概述產生為 README.collections.md
- 為每個項目自動建立 VS Code 安裝徽章
