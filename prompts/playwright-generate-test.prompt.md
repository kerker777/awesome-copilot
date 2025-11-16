---
mode: agent
description: '使用 Playwright MCP 根據情境產生 Playwright 測試'
tools: ['changes', 'search/codebase', 'edit/editFiles', 'fetch', 'problems', 'runCommands', 'runTasks', 'runTests', 'search', 'search/searchResults', 'runCommands/terminalLastCommand', 'runCommands/terminalSelection', 'testFailure', 'playwright/*']
model: 'Claude Sonnet 4.5'
---

# 使用 Playwright MCP 產生測試

您的目標是在完成所有規定的步驟後，根據提供的情境產生 Playwright 測試。

## 具體指示

- 您會獲得一個情境，您需要為其產生 playwright 測試。如果使用者未提供情境，您將要求他們提供一個。
- 不要過早產生測試程式碼，或僅根據情境而未完成所有規定的步驟就產生程式碼。
- 一定要使用 Playwright MCP 提供的工具逐步執行步驟。
- 僅在完成所有步驟後，才根據訊息歷史記錄發出使用 `@playwright/test` 的 Playwright TypeScript 測試
- 將產生的測試檔案儲存在 tests 目錄中
- 執行測試檔案並迭代直到測試通過
