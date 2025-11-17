---
mode: 'agent'
tools: ['githubRepo', 'github', 'get_me', 'get_pull_request', 'get_pull_request_comments', 'get_pull_request_diff', 'get_pull_request_files', 'get_pull_request_reviews', 'get_pull_request_status', 'list_pull_requests', 'request_copilot_review']
description: '列出目前儲存庫中我的 pull request'
---

搜尋目前的儲存庫（使用 #githubRepo 取得儲存庫資訊）並列出您找到的任何指派給我的 pull request（使用 #list_pull_requests）。

描述每個 pull request 的目的和詳細資訊。

如果 PR 正在等待某人審查，請在回應中強調這一點。

如果 PR 有任何檢查失敗，請描述它們並建議可能的修復方法。

如果 Copilot 還沒有進行審查，請使用 #request_copilot_review 提供請求審查的選項。
