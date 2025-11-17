---
mode: 'agent'
description: 'Create GitHub Issue for feature request from specification file using feature_request.yml template.'
tools: ['search/codebase', 'search', 'github', 'create_issue', 'search_issues', 'update_issue']
---
# 從規格說明建立 GitHub Issue

為位於 `${file}` 的規格說明建立 GitHub Issue。

## 流程

1. 分析規格說明檔案以萃取需求
2. 使用 `search_issues` 檢查現有的 Issue
3. 使用 `create_issue` 建立新 Issue 或使用 `update_issue` 更新現有的
4. 使用 `feature_request.yml` 範本（如無法取得則使用預設值）

## 要求

- 一個 Issue 涵蓋完整的規格說明
- 清楚的標題以識別該規格說明
- 只包含規格說明所需的變更
- 在建立前根據現有的 Issue 進行驗證

## Issue 內容

- 標題：規格說明中的功能名稱
- 描述：問題陳述、提議的解決方案和背景
- 標籤：feature、enhancement（視情況而定）
