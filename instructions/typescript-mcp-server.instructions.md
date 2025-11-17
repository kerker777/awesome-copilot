---
description: 'Instructions for building Model Context Protocol (MCP) servers using the TypeScript SDK'
applyTo: '**/*.ts, **/*.js, **/package.json'
---

# TypeScript MCP 伺服器開發

## 開發指南

- 使用 **@modelcontextprotocol/sdk** npm 套件：`npm install @modelcontextprotocol/sdk`
- 從特定路徑匯入：`@modelcontextprotocol/sdk/server/mcp.js`、`@modelcontextprotocol/sdk/server/stdio.js` 等
- 使用 `McpServer` 類別進行高階伺服器實作，具備自動協議處理
- 使用 `Server` 類別進行低階控制，搭配手動請求處理器
- 使用 **zod** 進行輸入/輸出結構描述驗證：`npm install zod@3`
- 務必為工具、資源和提示詞提供 `title` 欄位，以改善 UI 顯示效果
- 使用 `registerTool()`、`registerResource()` 和 `registerPrompt()` 方法（建議優先使用，勝於舊版 API）
- 使用 zod 定義結構描述：`{ inputSchema: { param: z.string() }, outputSchema: { result: z.string() } }`
- 從工具返回 `content`（用於顯示）和 `structuredContent`（用於結構化資料）
- HTTP 伺服器使用 `StreamableHTTPServerTransport` 搭配 Express 或類似框架
- 本機整合使用 `StdioServerTransport` 進行標準輸入/輸出通訊
- 為每個請求建立新的傳輸實例以防止請求 ID 衝突（無狀態模式）
- 對於有狀態伺服器，使用搭配 `sessionIdGenerator` 的工作階段管理
- 為本機伺服器啟用 DNS 重新繫結保護：`enableDnsRebindingProtection: true`
- 設定 CORS 標頭並公開 `Mcp-Session-Id` 供瀏覽器型客戶端使用
- 使用 `ResourceTemplate` 處理帶 URI 參數的動態資源：`new ResourceTemplate('resource://{param}', { list: undefined })`
- 使用來自 `@modelcontextprotocol/sdk/server/completable.js` 的 `completable()` 包裝器支援自動完成，以提升使用者體驗
- 使用 `server.server.createMessage()` 實作取樣以向客戶端請求 LLM 完成
- 使用 `server.server.elicitInput()` 在工具執行期間請求額外的使用者輸入
- 為批量更新啟用通知去抖動：`debouncedNotificationMethods: ['notifications/tools/list_changed']`
- 動態更新：在註冊項目上呼叫 `.enable()`、`.disable()`、`.update()` 或 `.remove()` 以發出 `listChanged` 通知
- 使用來自 `@modelcontextprotocol/sdk/shared/metadataUtils.js` 的 `getDisplayName()` 以取得 UI 顯示名稱
- 使用 MCP Inspector 測試伺服器：`npx @modelcontextprotocol/inspector`

## 最佳實踐

- 保持工具實作聚焦於單一職責
- 為 LLM 理解提供清晰、描述性的標題和說明
- 為所有參數和傳回值使用適當的 TypeScript 型別
- 使用 try-catch 區塊實作全面的錯誤處理
- 在工具結果中返回 `isError: true` 以表示錯誤條件
- 對所有非同步操作使用 async/await
- 正確關閉資料庫連接並清理資源
- 在處理之前驗證輸入參數
- 使用結構化日誌進行偵錯，而不污染標準輸出/標準錯誤
- 在公開檔案系統或網路存取時考慮安全性含義
- 在傳輸關閉事件時實作適當的資源清理
- 使用環境變數進行設定（連接埠、API 金鑰等）
- 清楚地記錄工具功能和限制
- 使用多個客戶端進行測試以確保相容性

## 常見模式

### 基礎伺服器設定 (HTTP)
```typescript
import { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js';
import { StreamableHTTPServerTransport } from '@modelcontextprotocol/sdk/server/streamableHttp.js';
import express from 'express';

const server = new McpServer({
    name: 'my-server',
    version: '1.0.0'
});

const app = express();
app.use(express.json());

app.post('/mcp', async (req, res) => {
    const transport = new StreamableHTTPServerTransport({
        sessionIdGenerator: undefined,
        enableJsonResponse: true
    });
    
    res.on('close', () => transport.close());
    
    await server.connect(transport);
    await transport.handleRequest(req, res, req.body);
});

app.listen(3000);
```

### 基礎伺服器設定 (stdio)
```typescript
import { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';

const server = new McpServer({
    name: 'my-server',
    version: '1.0.0'
});

// ... register tools, resources, prompts ...

const transport = new StdioServerTransport();
await server.connect(transport);
```

### 簡單的工具
```typescript
import { z } from 'zod';

server.registerTool(
    'calculate',
    {
        title: 'Calculator',
        description: 'Perform basic calculations',
        inputSchema: { a: z.number(), b: z.number(), op: z.enum(['+', '-', '*', '/']) },
        outputSchema: { result: z.number() }
    },
    async ({ a, b, op }) => {
        const result = op === '+' ? a + b : op === '-' ? a - b : 
                      op === '*' ? a * b : a / b;
        const output = { result };
        return {
            content: [{ type: 'text', text: JSON.stringify(output) }],
            structuredContent: output
        };
    }
);
```

### 動態資源
```typescript
import { ResourceTemplate } from '@modelcontextprotocol/sdk/server/mcp.js';

server.registerResource(
    'user',
    new ResourceTemplate('users://{userId}', { list: undefined }),
    {
        title: 'User Profile',
        description: 'Fetch user profile data'
    },
    async (uri, { userId }) => ({
        contents: [{
            uri: uri.href,
            text: `User ${userId} data here`
        }]
    })
);
```

### 具有取樣功能的工具
```typescript
server.registerTool(
    'summarize',
    {
        title: 'Text Summarizer',
        description: 'Summarize text using LLM',
        inputSchema: { text: z.string() },
        outputSchema: { summary: z.string() }
    },
    async ({ text }) => {
        const response = await server.server.createMessage({
            messages: [{
                role: 'user',
                content: { type: 'text', text: `Summarize: ${text}` }
            }],
            maxTokens: 500
        });
        
        const summary = response.content.type === 'text' ? 
            response.content.text : 'Unable to summarize';
        const output = { summary };
        return {
            content: [{ type: 'text', text: JSON.stringify(output) }],
            structuredContent: output
        };
    }
);
```

### 具有自動完成的提示詞
```typescript
import { completable } from '@modelcontextprotocol/sdk/server/completable.js';

server.registerPrompt(
    'review',
    {
        title: 'Code Review',
        description: 'Review code with specific focus',
        argsSchema: {
            language: completable(z.string(), value => 
                ['typescript', 'python', 'javascript', 'java']
                    .filter(l => l.startsWith(value))
            ),
            code: z.string()
        }
    },
    ({ language, code }) => ({
        messages: [{
            role: 'user',
            content: {
                type: 'text',
                text: `Review this ${language} code:\n\n${code}`
            }
        }]
    })
);
```

### 錯誤處理
```typescript
server.registerTool(
    'risky-operation',
    {
        title: 'Risky Operation',
        description: 'An operation that might fail',
        inputSchema: { input: z.string() },
        outputSchema: { result: z.string() }
    },
    async ({ input }) => {
        try {
            const result = await performRiskyOperation(input);
            const output = { result };
            return {
                content: [{ type: 'text', text: JSON.stringify(output) }],
                structuredContent: output
            };
        } catch (err: unknown) {
            const error = err as Error;
            return {
                content: [{ type: 'text', text: `Error: ${error.message}` }],
                isError: true
            };
        }
    }
);
```
