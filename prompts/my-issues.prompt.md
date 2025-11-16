---
mode: 'agent'
tools: ['githubRepo', 'github', 'get_issue', 'get_issue_comments', 'get_me', 'list_issues']
description: '列出目前儲存庫中我的 issue'
---

搜尋目前的儲存庫（使用 #githubRepo 取得儲存庫資訊）並列出您找到的任何指派給我的 issue（使用 #list_issues）。

根據 issue 的年齡、留言數量和狀態（開啟/關閉）建議我可能想要關注的 issue。
