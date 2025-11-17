---
description: 'Best practices and patterns for building Model Context Protocol (MCP) servers in Swift using the official MCP Swift SDK package.'
applyTo: "**/*.swift, **/Package.swift, **/Package.resolved"
---

# Swift MCP 伺服器開發指南

在 Swift 中建構 MCP 伺服器時，請遵循以下使用官方 Swift SDK 的最佳實踐和模式。

## 伺服器設定

使用具有功能的 `Server` 類別建立 MCP 伺服器：

```swift
import MCP

let server = Server(
    name: "MyServer",
    version: "1.0.0",
    capabilities: .init(
        prompts: .init(listChanged: true),
        resources: .init(subscribe: true, listChanged: true),
        tools: .init(listChanged: true)
    )
)
```

## 新增工具

使用 `withMethodHandler` 註冊工具處理程式：

```swift
// 註冊工具列表處理程式
await server.withMethodHandler(ListTools.self) { _ in
    let tools = [
        Tool(
            name: "search",
            description: "Search for information",
            inputSchema: .object([
                "properties": .object([
                    "query": .string("Search query"),
                    "limit": .number("Maximum results")
                ]),
                "required": .array([.string("query")])
            ])
        )
    ]
    return .init(tools: tools)
}

// 註冊工具呼叫處理程式
await server.withMethodHandler(CallTool.self) { params in
    switch params.name {
    case "search":
        let query = params.arguments?["query"]?.stringValue ?? ""
        let limit = params.arguments?["limit"]?.intValue ?? 10

        // 執行搜尋
        let results = performSearch(query: query, limit: limit)
        
        return .init(
            content: [.text("Found \(results.count) results")],
            isError: false
        )
        
    default:
        return .init(
            content: [.text("Unknown tool")],
            isError: true
        )
    }
}
```

## 新增資源

實作資源處理程式以進行資料存取：

```swift
// 註冊資源列表處理程式
await server.withMethodHandler(ListResources.self) { params in
    let resources = [
        Resource(
            name: "Data File",
            uri: "resource://data/example.txt",
            description: "Example data file",
            mimeType: "text/plain"
        )
    ]
    return .init(resources: resources, nextCursor: nil)
}

// 註冊資源讀取處理程式
await server.withMethodHandler(ReadResource.self) { params in
    switch params.uri {
    case "resource://data/example.txt":
        let content = loadResourceContent(uri: params.uri)
        return .init(contents: [
            Resource.Content.text(
                content,
                uri: params.uri,
                mimeType: "text/plain"
            )
        ])
        
    default:
        throw MCPError.invalidParams("Unknown resource URI: \(params.uri)")
    }
}

// 註冊資源訂閱處理程式
await server.withMethodHandler(ResourceSubscribe.self) { params in
    // 追蹤訂閱以接收通知
    subscriptions.insert(params.uri)
    print("Client subscribed to \(params.uri)")
    return .init()
}
```

## 新增提示

實作提示處理程式以進行樣板化對話：

```swift
// 註冊提示列表處理程式
await server.withMethodHandler(ListPrompts.self) { params in
    let prompts = [
        Prompt(
            name: "analyze",
            description: "Analyze a topic",
            arguments: [
                .init(name: "topic", description: "Topic to analyze", required: true),
                .init(name: "depth", description: "Analysis depth", required: false)
            ]
        )
    ]
    return .init(prompts: prompts, nextCursor: nil)
}

// 註冊提示取得處理程式
await server.withMethodHandler(GetPrompt.self) { params in
    switch params.name {
    case "analyze":
        let topic = params.arguments?["topic"]?.stringValue ?? "general"
        let depth = params.arguments?["depth"]?.stringValue ?? "basic"
        
        let description = "Analysis of \(topic) at \(depth) level"
        let messages: [Prompt.Message] = [
            .user("Please analyze this topic: \(topic)"),
            .assistant("I'll provide a \(depth) analysis of \(topic)")
        ]
        
        return .init(description: description, messages: messages)
        
    default:
        throw MCPError.invalidParams("Unknown prompt: \(params.name)")
    }
}
```

## 傳輸設定

### 標準輸入/輸出傳輸

用於本地子程序通訊：

```swift
import MCP
import Logging

let logger = Logger(label: "com.example.mcp-server")
let transport = StdioTransport(logger: logger)

try await server.start(transport: transport)
```

### HTTP 傳輸（用戶端側）

用於遠端伺服器連線：

```swift
let transport = HTTPClientTransport(
    endpoint: URL(string: "http://localhost:8080")!,
    streaming: true  // Enable Server-Sent Events
)

try await client.connect(transport: transport)
```

## 並行與角色

伺服器是一個角色，確保執行緒安全的存取：

```swift
actor ServerState {
    private var subscriptions: Set<String> = []
    private var cache: [String: Any] = [:]
    
    func addSubscription(_ uri: String) {
        subscriptions.insert(uri)
    }
    
    func getSubscriptions() -> Set<String> {
        return subscriptions
    }
}

let state = ServerState()

await server.withMethodHandler(ResourceSubscribe.self) { params in
    await state.addSubscription(params.uri)
    return .init()
}
```

## 錯誤處理

使用 Swift 的錯誤處理搭配 `MCPError`：

```swift
await server.withMethodHandler(CallTool.self) { params in
    do {
        guard let query = params.arguments?["query"]?.stringValue else {
            throw MCPError.invalidParams("Missing query parameter")
        }
        
        let result = try performOperation(query: query)
        
        return .init(
            content: [.text(result)],
            isError: false
        )
    } catch let error as MCPError {
        return .init(
            content: [.text(error.localizedDescription)],
            isError: true
        )
    } catch {
        return .init(
            content: [.text("Unexpected error: \(error.localizedDescription)")],
            isError: true
        )
    }
}
```

## JSON 結構描述與 Value 型別

使用 `Value` 型別來定義 JSON 結構描述：

```swift
let schema = Value.object([
    "type": .string("object"),
    "properties": .object([
        "name": .object([
            "type": .string("string"),
            "description": .string("User's name")
        ]),
        "age": .object([
            "type": .string("integer"),
            "minimum": .number(0),
            "maximum": .number(150)
        ]),
        "email": .object([
            "type": .string("string"),
            "format": .string("email")
        ])
    ]),
    "required": .array([.string("name")])
])
```

## Swift 套件管理器設定

建立你的 `Package.swift`：

```swift
// swift-tools-version: 6.0
import PackageDescription

let package = Package(
    name: "MyMCPServer",
    platforms: [
        .macOS(.v13),
        .iOS(.v16)
    ],
    dependencies: [
        .package(
            url: "https://github.com/modelcontextprotocol/swift-sdk.git",
            from: "0.10.0"
        ),
        .package(
            url: "https://github.com/apple/swift-log.git",
            from: "1.5.0"
        )
    ],
    targets: [
        .executableTarget(
            name: "MyMCPServer",
            dependencies: [
                .product(name: "MCP", package: "swift-sdk"),
                .product(name: "Logging", package: "swift-log")
            ]
        )
    ]
)
```

## 使用 ServiceLifecycle 進行優雅關閉

使用 Swift Service Lifecycle 以進行適當的關閉：

```swift
import MCP
import ServiceLifecycle
import Logging

struct MCPService: Service {
    let server: Server
    let transport: Transport
    
    func run() async throws {
        try await server.start(transport: transport)
        try await Task.sleep(for: .days(365 * 100))
    }
    
    func shutdown() async throws {
        await server.stop()
    }
}

let logger = Logger(label: "com.example.mcp-server")
let transport = StdioTransport(logger: logger)
let mcpService = MCPService(server: server, transport: transport)

let serviceGroup = ServiceGroup(
    services: [mcpService],
    configuration: .init(
        gracefulShutdownSignals: [.sigterm, .sigint]
    ),
    logger: logger
)

try await serviceGroup.run()
```

## 非同步/等待模式

所有伺服器操作都使用 Swift 並行機制：

```swift
await server.withMethodHandler(CallTool.self) { params in
    async let result1 = fetchData1()
    async let result2 = fetchData2()
    
    let combined = await "\(result1) and \(result2)"
    
    return .init(
        content: [.text(combined)],
        isError: false
    )
}
```

## 日誌記錄

使用 swift-log 進行結構化日誌記錄：

```swift
import Logging

let logger = Logger(label: "com.example.mcp-server")

await server.withMethodHandler(CallTool.self) { params in
    logger.info("Tool called", metadata: [
        "name": .string(params.name),
        "args": .string("\(params.arguments ?? [:])")
    ])
    
    // Process tool call
    
    logger.debug("Tool completed successfully")
    
    return .init(content: [.text("Result")], isError: false)
}
```

## 測試

使用非同步/等待測試你的伺服器：

```swift
import XCTest
@testable import MyMCPServer

final class ServerTests: XCTestCase {
    func testToolCall() async throws {
        let server = createTestServer()
        
        // 測試工具呼叫邏輯
        let params = CallTool.Params(
            name: "search",
            arguments: ["query": .string("test")]
        )

        // 驗證行為
        XCTAssertNoThrow(try await processToolCall(params))
    }
}
```

## 初始化掛鉤

使用初始化掛鉤驗證用戶端連線：

```swift
try await server.start(transport: transport) { clientInfo, clientCapabilities in
    // 驗證用戶端
    guard clientInfo.name != "BlockedClient" else {
        throw MCPError.invalidRequest("Client not allowed")
    }

    // 檢查功能
    if clientCapabilities.sampling == nil {
        logger.warning("Client doesn't support sampling")
    }

    logger.info("Client connected", metadata: [
        "name": .string(clientInfo.name),
        "version": .string(clientInfo.version)
    ])
}
```

## 常見模式

### 內容型別

處理不同的內容型別：

```swift
return .init(
    content: [
        .text("Plain text response"),
        .image(imageData, mimeType: "image/png", metadata: [
            "width": 1024,
            "height": 768
        ]),
        .resource(
            uri: "resource://data",
            mimeType: "application/json",
            text: jsonString
        )
    ],
    isError: false
)
```

### 嚴格設定

使用嚴格模式在缺少功能時快速失敗：

```swift
let client = Client(
    name: "StrictClient",
    version: "1.0.0",
    configuration: .strict
)

// 如果功能無法使用，將立即拋出例外
try await client.listTools()
```

### 請求批次

有效率地傳送多個請求：

```swift
var tasks: [Task<CallTool.Result, Error>] = []

try await client.withBatch { batch in
    for i in 0..<10 {
        tasks.append(
            try await batch.addRequest(
                CallTool.request(.init(
                    name: "process",
                    arguments: ["id": .number(Double(i))]
                ))
            )
        )
    }
}

for (index, task) in tasks.enumerated() {
    let result = try await task.value
    print("\(index): \(result.content)")
}
```
