---
applyTo: '*'
description: 'Quarkus 和使用 HTTP SSE 傳輸的 MCP 伺服器開發標準和指引'
---
# Quarkus MCP 伺服器

使用 Java 21、Quarkus 和 HTTP SSE 傳輸構建 MCP 伺服器。

## 技術堆疊

- Java 21 搭配 Quarkus 框架
- MCP 伺服器擴充功能：`mcp-server-sse`
- CDI 用於依賴注入
- MCP 端點：`http://localhost:8080/mcp/sse`

## 快速開始

```bash
quarkus create app --no-code -x rest-client-jackson,qute,mcp-server-sse your-domain-mcp-server
```

## 結構

- 使用標準 Java 命名慣例（類別使用 PascalCase，方法使用 camelCase）
- 組織在以下套件中：`model`、`repository`、`service`、`mcp`
- 對不可變資料模型使用 Record 型別
- 不可變資料的狀態管理必須由儲存庫層管理
- 為公開方法新增 Javadoc

## MCP 工具

- 必須是 `@ApplicationScoped` CDI Bean 中的公開方法
- 使用 `@Tool(name="tool_name", description="清晰的描述")`
- 永遠不要回傳 `null` - 改為回傳錯誤訊息
- 始終驗證參數並優雅地處理錯誤

## 架構

- 關注點分離：MCP 工具 → 服務層 → 儲存庫
- 使用 `@Inject` 進行依賴注入
- 使資料操作具備執行緒安全性
- 使用 `Optional<T>` 避免空指標例外

## 常見問題

- 不要在 MCP 工具中放置業務邏輯（使用服務層）
- 不要從工具中拋出例外（回傳錯誤字串）
- 不要忘記驗證輸入參數
- 使用邊緣情況進行測試（null、空輸入）
