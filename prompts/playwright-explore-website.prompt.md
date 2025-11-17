---
mode: agent
description: '使用 Playwright MCP 探索網站進行測試'
tools: ['changes', 'search/codebase', 'edit/editFiles', 'fetch', 'findTestFiles', 'problems', 'runCommands', 'runTasks', 'runTests', 'search', 'search/searchResults', 'runCommands/terminalLastCommand', 'runCommands/terminalSelection', 'testFailure', 'playwright']
model: 'Claude Sonnet 4'
---

# 網站探索以進行測試

您的目標是探索網站並識別關鍵功能。

## 具體指示

1. 使用 Playwright MCP Server 導覽至提供的 URL。如果未提供 URL，請要求使用者提供。
2. 識別並與 3-5 個核心功能或使用者流程互動。
3. 記錄使用者互動、相關的 UI 元素（及其定位器）以及預期結果。
4. 完成後關閉瀏覽器情境。
5. 提供您發現的簡潔摘要。
6. 根據探索提出並產生測試案例。
