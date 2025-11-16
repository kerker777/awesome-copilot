---
description: '使用 C# SDK 建構模型上下文協定 (MCP) 伺服器的指引'
applyTo: '**/*.cs, **/*.csproj'
---

# C# MCP 伺服器開發

## 指引

- 對大多數專案使用 **ModelContextProtocol** NuGet 套件 (預發行版): `dotnet add package ModelContextProtocol --prerelease`
- 對基於 HTTP 的 MCP 伺服器使用 **ModelContextProtocol.AspNetCore**
- 對最小相依性使用 **ModelContextProtocol.Core** (僅用戶端或低階伺服器 API)
- 務必使用 `LogToStandardErrorThreshold = LogLevel.Trace` 將記錄設定為 stderr,以避免干擾 stdio 傳輸
- 在包含 MCP 工具的類別上使用 `[McpServerToolType]` 屬性
- 在方法上使用 `[McpServerTool]` 屬性以將其公開為工具
- 使用 `System.ComponentModel` 中的 `[Description]` 屬性來記錄工具和參數
- 支援工具方法中的相依性注入 - 將 `McpServer`、`HttpClient` 或其他服務作為參數注入
- 使用 `McpServer.AsSamplingChatClient()` 從工具內向用戶端發出取樣請求
- 在類別上使用 `[McpServerPromptType]`,在方法上使用 `[McpServerPrompt]` 來公開提示
- 對於 stdio 傳輸,在建構伺服器時使用 `WithStdioServerTransport()`
- 使用 `WithToolsFromAssembly()` 自動探索並註冊來自目前組件的所有工具
- 工具方法可以是同步或非同步的(回傳 `Task` 或 `Task<T>`)
- 務必為工具和參數包含全面的描述,以幫助 LLM 理解其用途
- 在非同步工具中使用 `CancellationToken` 參數以正確支援取消
- 回傳簡單類型(string、int 等)或可序列化為 JSON 的複雜物件
- 對於精細控制,使用 `McpServerOptions` 搭配自訂處理器,如 `ListToolsHandler` 和 `CallToolHandler`
- 使用 `McpProtocolException` 處理協定層級錯誤,並使用適當的 `McpErrorCode` 值
- 使用相同 SDK 的 `McpClient` 或任何符合標準的 MCP 用戶端測試 MCP 伺服器
- 使用 Microsoft.Extensions.Hosting 結構化專案以進行適當的 DI 和生命週期管理

## 最佳實踐

- 保持工具方法專注且單一用途
- 使用清楚指示其功能的有意義工具名稱
- 提供詳細描述,說明工具的功能、預期參數以及回傳內容
- 驗證輸入參數,對於無效輸入拋出 `McpProtocolException` 並使用 `McpErrorCode.InvalidParams`
- 使用結構化記錄來幫助除錯,而不會污染 stdout
- 使用 `[McpServerToolType]` 將相關工具組織到邏輯類別中
- 在公開存取外部資源的工具時考慮安全性影響
- 使用內建的 DI 容器來管理服務生命週期和相依性
- 實作適當的錯誤處理並回傳有意義的錯誤訊息
- 在與 LLM 整合之前單獨測試工具

## Common Patterns

### Basic Server Setup
```csharp
var builder = Host.CreateApplicationBuilder(args);
builder.Logging.AddConsole(options => 
    options.LogToStandardErrorThreshold = LogLevel.Trace);
builder.Services
    .AddMcpServer()
    .WithStdioServerTransport()
    .WithToolsFromAssembly();
await builder.Build().RunAsync();
```

### Simple Tool
```csharp
[McpServerToolType]
public static class MyTools
{
    [McpServerTool, Description("Description of what the tool does")]
    public static string ToolName(
        [Description("Parameter description")] string param) => 
        $"Result: {param}";
}
```

### Tool with Dependency Injection
```csharp
[McpServerTool, Description("Fetches data from a URL")]
public static async Task<string> FetchData(
    HttpClient httpClient,
    [Description("The URL to fetch")] string url,
    CancellationToken cancellationToken) =>
    await httpClient.GetStringAsync(url, cancellationToken);
```

### Tool with Sampling
```csharp
[McpServerTool, Description("Analyzes content using the client's LLM")]
public static async Task<string> Analyze(
    McpServer server,
    [Description("Content to analyze")] string content,
    CancellationToken cancellationToken)
{
    var messages = new ChatMessage[]
    {
        new(ChatRole.User, $"Analyze this: {content}")
    };
    return await server.AsSamplingChatClient()
        .GetResponseAsync(messages, cancellationToken: cancellationToken);
}
```
