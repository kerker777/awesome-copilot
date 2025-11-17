---
mode: 'agent'
description: 'Create GitHub Issues for unimplemented requirements from specification files using feature_request.yml template.'
tools: ['search/codebase', 'search', 'github', 'create_issue', 'search_issues', 'update_issue']
---
# 為未實現的規格需求建立 GitHub Issues

為位於 `${file}` 的規格中未實現的需求建立 GitHub Issues。

## 流程

1. 分析規格文件以提取所有需求
2. 檢查程式碼庫中每個需求的實現狀態
3. 使用 `search_issues` 搜尋現有 Issues 以避免重複
4. 使用 `create_issue` 為每個未實現的需求建立新 Issue
5. 使用 `feature_request.yml` 範本（若無則使用預設）

## 需求

- 每個規格中的未實現需求建立一個 Issue
- 清晰的需求 ID 與描述對應
- 包含實現指南與驗收條件
- 在建立前針對現有 Issues 進行驗證

## Issue 內容

- 標題：需求 ID 與簡要描述
- 描述：詳細需求、實現方法與背景
- 標籤：feature、enhancement（視情況而定）

## 實現檢查

- 搜尋程式碼庫中相關的程式碼模式
- 檢查 `/spec/` 目錄中相關的規格文件
- 驗證需求未被部分實現
