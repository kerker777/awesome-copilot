---
description: 'Generate a complete Model Context Protocol server project in Swift using the official MCP Swift SDK package.'
mode: agent
---

# Swift MCP 伺服器產生器

使用官方 Swift SDK 套件產生一個完整、可用於生產環境的 MCP 伺服器。

## 專案產生

當要求建立 Swift MCP 伺服器時，產生一個具有以下結構的完整專案：

```
my-mcp-server/
├── Package.swift
├── Sources/
│   └── MyMCPServer/
│       ├── main.swift
│       ├── Server.swift
│       ├── Tools/
│       │   ├── ToolDefinitions.swift
│       │   └── ToolHandlers.swift
│       ├── Resources/
│       │   ├── ResourceDefinitions.swift
│       │   └── ResourceHandlers.swift
│       └── Prompts/
│           ├── PromptDefinitions.swift
│           └── PromptHandlers.swift
├── Tests/
│   └── MyMCPServerTests/
│       └── ServerTests.swift
└── README.md
```

## Package.swift 樣板

```swift
// swift-tools-version: 6.0
import PackageDescription

let package = Package(
    name: "MyMCPServer",
    platforms: [
        .macOS(.v13),
        .iOS(.v16),
        .watchOS(.v9),
        .tvOS(.v16),
        .visionOS(.v1)
    ],
    dependencies: [
        .package(
            url: "https://github.com/modelcontextprotocol/swift-sdk.git",
            from: "0.10.0"
        ),
        .package(
            url: "https://github.com/apple/swift-log.git",
            from: "1.5.0"
        ),
        .package(
            url: "https://github.com/swift-server/swift-service-lifecycle.git",
            from: "2.0.0"
        )
    ],
    targets: [
        .executableTarget(
            name: "MyMCPServer",
            dependencies: [
                .product(name: "MCP", package: "swift-sdk"),
                .product(name: "Logging", package: "swift-log"),
                .product(name: "ServiceLifecycle", package: "swift-service-lifecycle")
            ]
        ),
        .testTarget(
            name: "MyMCPServerTests",
            dependencies: ["MyMCPServer"]
        )
    ]
)
```

## main.swift 樣板

```swift
import MCP
import Logging
import ServiceLifecycle

struct MCPService: Service {
    let server: Server
    let transport: Transport
    
    func run() async throws {
        try await server.start(transport: transport) { clientInfo, capabilities in
            logger.info("Client connected", metadata: [
                "name": .string(clientInfo.name),
                "version": .string(clientInfo.version)
            ])
        }
        
        // Keep service running
        try await Task.sleep(for: .days(365 * 100))
    }
    
    func shutdown() async throws {
        logger.info("Shutting down MCP server")
        await server.stop()
    }
}

var logger = Logger(label: "com.example.mcp-server")
logger.logLevel = .info

do {
    let server = await createServer()
    let transport = StdioTransport(logger: logger)
    let service = MCPService(server: server, transport: transport)
    
    let serviceGroup = ServiceGroup(
        services: [service],
        configuration: .init(
            gracefulShutdownSignals: [.sigterm, .sigint]
        ),
        logger: logger
    )
    
    try await serviceGroup.run()
} catch {
    logger.error("Fatal error", metadata: ["error": .string("\(error)")])
    throw error
}
```

## Server.swift 樣板

```swift
import MCP
import Logging

func createServer() async -> Server {
    let server = Server(
        name: "MyMCPServer",
        version: "1.0.0",
        capabilities: .init(
            prompts: .init(listChanged: true),
            resources: .init(subscribe: true, listChanged: true),
            tools: .init(listChanged: true)
        )
    )
    
    // Register tool handlers
    await registerToolHandlers(server: server)
    
    // Register resource handlers
    await registerResourceHandlers(server: server)
    
    // Register prompt handlers
    await registerPromptHandlers(server: server)
    
    return server
}
```

## ToolDefinitions.swift 樣板

```swift
import MCP

func getToolDefinitions() -> [Tool] {
    [
        Tool(
            name: "greet",
            description: "Generate a greeting message",
            inputSchema: .object([
                "type": .string("object"),
                "properties": .object([
                    "name": .object([
                        "type": .string("string"),
                        "description": .string("Name to greet")
                    ])
                ]),
                "required": .array([.string("name")])
            ])
        ),
        Tool(
            name: "calculate",
            description: "Perform mathematical calculations",
            inputSchema: .object([
                "type": .string("object"),
                "properties": .object([
                    "operation": .object([
                        "type": .string("string"),
                        "enum": .array([
                            .string("add"),
                            .string("subtract"),
                            .string("multiply"),
                            .string("divide")
                        ]),
                        "description": .string("Operation to perform")
                    ]),
                    "a": .object([
                        "type": .string("number"),
                        "description": .string("First operand")
                    ]),
                    "b": .object([
                        "type": .string("number"),
                        "description": .string("Second operand")
                    ])
                ]),
                "required": .array([
                    .string("operation"),
                    .string("a"),
                    .string("b")
                ])
            ])
        )
    ]
}
```

## ToolHandlers.swift 樣板

```swift
import MCP
import Logging

private let logger = Logger(label: "com.example.mcp-server.tools")

func registerToolHandlers(server: Server) async {
    await server.withMethodHandler(ListTools.self) { _ in
        logger.debug("Listing available tools")
        return .init(tools: getToolDefinitions())
    }
    
    await server.withMethodHandler(CallTool.self) { params in
        logger.info("Tool called", metadata: ["name": .string(params.name)])
        
        switch params.name {
        case "greet":
            return handleGreet(params: params)
            
        case "calculate":
            return handleCalculate(params: params)
            
        default:
            logger.warning("Unknown tool requested", metadata: ["name": .string(params.name)])
            return .init(
                content: [.text("Unknown tool: \(params.name)")],
                isError: true
            )
        }
    }
}

private func handleGreet(params: CallTool.Params) -> CallTool.Result {
    guard let name = params.arguments?["name"]?.stringValue else {
        return .init(
            content: [.text("Missing 'name' parameter")],
            isError: true
        )
    }
    
    let greeting = "Hello, \(name)! Welcome to MCP."
    logger.debug("Generated greeting", metadata: ["name": .string(name)])
    
    return .init(
        content: [.text(greeting)],
        isError: false
    )
}

private func handleCalculate(params: CallTool.Params) -> CallTool.Result {
    guard let operation = params.arguments?["operation"]?.stringValue,
          let a = params.arguments?["a"]?.doubleValue,
          let b = params.arguments?["b"]?.doubleValue else {
        return .init(
            content: [.text("Missing or invalid parameters")],
            isError: true
        )
    }
    
    let result: Double
    switch operation {
    case "add":
        result = a + b
    case "subtract":
        result = a - b
    case "multiply":
        result = a * b
    case "divide":
        guard b != 0 else {
            return .init(
                content: [.text("Division by zero")],
                isError: true
            )
        }
        result = a / b
    default:
        return .init(
            content: [.text("Unknown operation: \(operation)")],
            isError: true
        )
    }
    
    logger.debug("Calculation performed", metadata: [
        "operation": .string(operation),
        "result": .string("\(result)")
    ])
    
    return .init(
        content: [.text("Result: \(result)")],
        isError: false
    )
}
```

## ResourceDefinitions.swift 樣板

```swift
import MCP

func getResourceDefinitions() -> [Resource] {
    [
        Resource(
            name: "Example Data",
            uri: "resource://data/example",
            description: "Example resource data",
            mimeType: "application/json"
        ),
        Resource(
            name: "Configuration",
            uri: "resource://config",
            description: "Server configuration",
            mimeType: "application/json"
        )
    ]
}
```

## ResourceHandlers.swift 樣板

```swift
import MCP
import Logging
import Foundation

private let logger = Logger(label: "com.example.mcp-server.resources")

actor ResourceState {
    private var subscriptions: Set<String> = []
    
    func addSubscription(_ uri: String) {
        subscriptions.insert(uri)
    }
    
    func removeSubscription(_ uri: String) {
        subscriptions.remove(uri)
    }
    
    func isSubscribed(_ uri: String) -> Bool {
        subscriptions.contains(uri)
    }
}

private let state = ResourceState()

func registerResourceHandlers(server: Server) async {
    await server.withMethodHandler(ListResources.self) { params in
        logger.debug("Listing available resources")
        return .init(resources: getResourceDefinitions(), nextCursor: nil)
    }
    
    await server.withMethodHandler(ReadResource.self) { params in
        logger.info("Reading resource", metadata: ["uri": .string(params.uri)])
        
        switch params.uri {
        case "resource://data/example":
            let jsonData = """
            {
                "message": "Example resource data",
                "timestamp": "\(Date())"
            }
            """
            return .init(contents: [
                .text(jsonData, uri: params.uri, mimeType: "application/json")
            ])
            
        case "resource://config":
            let config = """
            {
                "serverName": "MyMCPServer",
                "version": "1.0.0"
            }
            """
            return .init(contents: [
                .text(config, uri: params.uri, mimeType: "application/json")
            ])
            
        default:
            logger.warning("Unknown resource requested", metadata: ["uri": .string(params.uri)])
            throw MCPError.invalidParams("Unknown resource URI: \(params.uri)")
        }
    }
    
    await server.withMethodHandler(ResourceSubscribe.self) { params in
        logger.info("Client subscribed to resource", metadata: ["uri": .string(params.uri)])
        await state.addSubscription(params.uri)
        return .init()
    }
    
    await server.withMethodHandler(ResourceUnsubscribe.self) { params in
        logger.info("Client unsubscribed from resource", metadata: ["uri": .string(params.uri)])
        await state.removeSubscription(params.uri)
        return .init()
    }
}
```

## PromptDefinitions.swift 樣板

```swift
import MCP

func getPromptDefinitions() -> [Prompt] {
    [
        Prompt(
            name: "code-review",
            description: "Generate a code review prompt",
            arguments: [
                .init(name: "language", description: "Programming language", required: true),
                .init(name: "focus", description: "Review focus area", required: false)
            ]
        )
    ]
}
```

## PromptHandlers.swift 樣板

```swift
import MCP
import Logging

private let logger = Logger(label: "com.example.mcp-server.prompts")

func registerPromptHandlers(server: Server) async {
    await server.withMethodHandler(ListPrompts.self) { params in
        logger.debug("Listing available prompts")
        return .init(prompts: getPromptDefinitions(), nextCursor: nil)
    }
    
    await server.withMethodHandler(GetPrompt.self) { params in
        logger.info("Getting prompt", metadata: ["name": .string(params.name)])
        
        switch params.name {
        case "code-review":
            return handleCodeReviewPrompt(params: params)
            
        default:
            logger.warning("Unknown prompt requested", metadata: ["name": .string(params.name)])
            throw MCPError.invalidParams("Unknown prompt: \(params.name)")
        }
    }
}

private func handleCodeReviewPrompt(params: GetPrompt.Params) -> GetPrompt.Result {
    guard let language = params.arguments?["language"]?.stringValue else {
        return .init(
            description: "Missing language parameter",
            messages: []
        )
    }
    
    let focus = params.arguments?["focus"]?.stringValue ?? "general quality"
    
    let description = "Code review for \(language) with focus on \(focus)"
    let messages: [Prompt.Message] = [
        .user("Please review this \(language) code with focus on \(focus)."),
        .assistant("I'll review the code focusing on \(focus). Please share the code."),
        .user("Here's the code to review: [paste code here]")
    ]
    
    logger.debug("Generated code review prompt", metadata: [
        "language": .string(language),
        "focus": .string(focus)
    ])
    
    return .init(description: description, messages: messages)
}
```

## ServerTests.swift 樣板

```swift
import XCTest
@testable import MyMCPServer

final class ServerTests: XCTestCase {
    func testGreetTool() async throws {
        let params = CallTool.Params(
            name: "greet",
            arguments: ["name": .string("Swift")]
        )
        
        let result = handleGreet(params: params)
        
        XCTAssertFalse(result.isError ?? true)
        XCTAssertEqual(result.content.count, 1)
        
        if case .text(let message) = result.content[0] {
            XCTAssertTrue(message.contains("Swift"))
        } else {
            XCTFail("Expected text content")
        }
    }
    
    func testCalculateTool() async throws {
        let params = CallTool.Params(
            name: "calculate",
            arguments: [
                "operation": .string("add"),
                "a": .number(5),
                "b": .number(3)
            ]
        )
        
        let result = handleCalculate(params: params)
        
        XCTAssertFalse(result.isError ?? true)
        XCTAssertEqual(result.content.count, 1)
        
        if case .text(let message) = result.content[0] {
            XCTAssertTrue(message.contains("8"))
        } else {
            XCTFail("Expected text content")
        }
    }
    
    func testDivideByZero() async throws {
        let params = CallTool.Params(
            name: "calculate",
            arguments: [
                "operation": .string("divide"),
                "a": .number(10),
                "b": .number(0)
            ]
        )
        
        let result = handleCalculate(params: params)
        
        XCTAssertTrue(result.isError ?? false)
    }
}
```

## README.md 樣板

```markdown
# MyMCPServer

使用 Swift 建立的 Model Context Protocol 伺服器。

## 功能

- ✅ 工具：greet、calculate
- ✅ 資源：example data、configuration
- ✅ 提示：code-review
- ✅ 使用 ServiceLifecycle 的優雅關閉
- ✅ 使用 swift-log 的結構化日誌
- ✅ 完整的測試覆蓋範圍

## 需求

- Swift 6.0+
- macOS 13+、iOS 16+ 或 Linux

## 安裝

```bash
swift build -c release
```

## 用法

執行伺服器：

```bash
swift run
```

或使用日誌：

```bash
LOG_LEVEL=debug swift run
```

## 測試

```bash
swift test
```

## 開發

伺服器使用：
- [MCP Swift SDK](https://github.com/modelcontextprotocol/swift-sdk) - MCP 協定實作
- [swift-log](https://github.com/apple/swift-log) - 結構化日誌
- [swift-service-lifecycle](https://github.com/swift-server/swift-service-lifecycle) - 優雅關閉

## 專案結構

- `Sources/MyMCPServer/main.swift` - 入口點，包含 ServiceLifecycle
- `Sources/MyMCPServer/Server.swift` - 伺服器設定
- `Sources/MyMCPServer/Tools/` - 工具定義與處理程式
- `Sources/MyMCPServer/Resources/` - 資源定義與處理程式
- `Sources/MyMCPServer/Prompts/` - 提示定義與處理程式
- `Tests/` - 單位測試

## 授權

MIT
```

## 產生指示

1. **詢問專案名稱與描述**
2. **使用正確的命名產生所有檔案**
3. **使用 Actor 型狀態**以保證執行緒安全
4. **包含完整的日誌記錄**使用 swift-log
5. **使用 ServiceLifecycle 實作優雅關閉**
6. **為所有處理程式新增測試**
7. **使用現代 Swift 並行機制** (async/await)
8. **遵循 Swift 命名慣例** (camelCase、PascalCase)
9. **包含錯誤處理**使用適當的 MCPError 用法
10. **使用文件註解為公開 API 建立文件**

## 建置與執行

```bash
# 建置
swift build

# 執行
swift run

# 測試
swift test

# 發行版本建置
swift build -c release

# 安裝
swift build -c release
cp .build/release/MyMCPServer /usr/local/bin/
```

## 與 Claude Desktop 整合

新增至 `claude_desktop_config.json`：

```json
{
  "mcpServers": {
    "my-mcp-server": {
      "command": "/path/to/MyMCPServer"
    }
  }
}
```
