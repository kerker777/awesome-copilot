---
description: 'Azure Functions 的 TypeScript 模式'
applyTo: '**/*.ts, **/*.js, **/*.json'
---

## 程式碼產生指南
- 為 Node.js 產生現代 TypeScript 程式碼
- 對非同步程式碼使用 `async/await`
- 盡可能使用 Node.js v20 內建模組而不是外部套件
- 始終使用 Node.js 非同步函數,如 `node:fs/promises` 而不是 `fs`,以避免阻塞事件迴圈
- 在新增任何額外依賴項到專案之前先詢問
- API 使用 `@azure/functions@4` 套件建置的 Azure Functions。
- 每個端點應該有自己的函數文件,並使用以下命名慣例:`src/functions/<resource-name>-<http-verb>.ts`
- 在對 API 進行變更時,請確保相應地更新 OpenAPI 架構(如果存在)和 `README.md` 文件。
