---
mode: 'agent'
description: 'Create GitHub Issues from implementation plan phases using feature_request.yml or chore_request.yml templates.'
tools: ['search/codebase', 'search', 'github', 'create_issue', 'search_issues', 'update_issue']
---
# 從實施計劃建立 GitHub Issue

為位於 `${file}` 的實施計劃建立 GitHub Issues。

## 流程

1. 分析計劃檔案以識別各個階段
2. 使用 `search_issues` 檢查現有的 issues
3. 使用 `create_issue` 為每個階段建立新的 issue，或使用 `update_issue` 更新現有 issue
4. 使用 `feature_request.yml` 或 `chore_request.yml` 範本（如果沒有則使用預設）

## 需求

- 每個實施階段需要一個 issue
- 清晰、結構化的標題和描述
- 只包含計劃所需的變更
- 建立前驗證現有的 issues

## Issue 內容

- 標題：來自實施計劃的階段名稱
- 描述：階段詳細資訊、需求和背景
- 標籤：適用於 issue 類型（feature/chore）
