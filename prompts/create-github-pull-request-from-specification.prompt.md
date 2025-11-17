---
mode: 'agent'
description: 'Create GitHub Pull Request for feature request from specification file using pull_request_template.md template.'
tools: ['search/codebase', 'search', 'github', 'create_pull_request', 'update_pull_request', 'get_pull_request_diff']
---
# 從規格文件建立 GitHub Pull Request

為位於 `${workspaceFolder}/.github/pull_request_template.md` 的規格文件建立 GitHub Pull Request。

## 流程

1. 分析來自 '${workspaceFolder}/.github/pull_request_template.md' 的規格文件樣板，使用 'search' 工具提取需求。
2. 使用 'create_pull_request' 工具建立 Pull Request 草稿樣板至 `${input:targetBranch}`。並確保使用 `get_pull_request` 檢查當前分支不存在任何 Pull Request。如果存在，則跳至步驟 4，略過步驟 3。
3. 使用 'get_pull_request_diff' 工具取得 Pull Request 中的變更，以分析 Pull Request 中改變的資訊。
4. 使用 'update_pull_request' 工具更新前一步驟建立的 Pull Request 內容和標題。根據需要，將第一步驟取得的樣板資訊整合至內容和標題中。
5. 使用 'update_pull_request' 工具將草稿轉換為準備好審查的狀態，以更新 Pull Request 的狀態。
6. 使用 'get_me' 取得建立 Pull Request 的人員用戶名，並指派給 `update_issue` 工具來指派 Pull Request。
7. 將建立的 Pull Request URL 回傳給使用者。

## 需求
- 為完整規格文件建立單一 Pull Request
- 標題清晰的 Pull Request（識別規格文件）
- 在 pull_request_template.md 中填入足夠的資訊
- 建立前驗證是否與現有 Pull Request 衝突
