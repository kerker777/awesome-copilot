---
mode: agent
description: 'Generate a complete Go MCP server project with proper structure, dependencies, and implementation using the official github.com/modelcontextprotocol/go-sdk.'
---

# Go MCP 伺服器專案產生器

產生一個完整、生產就緒的 Go 語言 Model Context Protocol (MCP) 伺服器專案。

## 專案需求

你將建立一個具有下列特性的 Go MCP 伺服器：

1. **專案結構**：適當的 Go 模組佈局
2. **依賴套件**：官方 MCP SDK 及必要的套件
3. **伺服器設置**：配置 MCP 伺服器及傳輸層
4. **工具**：至少 2-3 個實用工具，具有型別化的輸入/輸出
5. **錯誤處理**：適當的錯誤處理及上下文使用
6. **文件**：包含設置及使用說明的 README
7. **測試**：基本測試結構

## 樣板結構

```
myserver/
├── go.mod
├── go.sum
├── main.go
├── tools/
│   ├── tool1.go
│   └── tool2.go
├── resources/
│   └── resource1.go
├── config/
│   └── config.go
├── README.md
└── main_test.go
```

## go.mod 樣板

```go
module github.com/yourusername/{{PROJECT_NAME}}

go 1.23

require (
    github.com/modelcontextprotocol/go-sdk v1.0.0
)
```

## main.go 樣板

```go
package main

import (
    "context"
    "log"
    "os"
    "os/signal"
    "syscall"

    "github.com/modelcontextprotocol/go-sdk/mcp"
    "github.com/yourusername/{{PROJECT_NAME}}/config"
    "github.com/yourusername/{{PROJECT_NAME}}/tools"
)

func main() {
    cfg := config.Load()
    
    ctx, cancel := context.WithCancel(context.Background())
    defer cancel()

    // Handle graceful shutdown
    sigCh := make(chan os.Signal, 1)
    signal.Notify(sigCh, os.Interrupt, syscall.SIGTERM)
    go func() {
        <-sigCh
        log.Println("Shutting down...")
        cancel()
    }()

    // Create server
    server := mcp.NewServer(
        &mcp.Implementation{
            Name:    cfg.ServerName,
            Version: cfg.Version,
        },
        &mcp.Options{
            Capabilities: &mcp.ServerCapabilities{
                Tools:     &mcp.ToolsCapability{},
                Resources: &mcp.ResourcesCapability{},
                Prompts:   &mcp.PromptsCapability{},
            },
        },
    )

    // Register tools
    tools.RegisterTools(server)

    // Run server
    transport := &mcp.StdioTransport{}
    if err := server.Run(ctx, transport); err != nil {
        log.Fatalf("Server error: %v", err)
    }
}
```

## tools/tool1.go 樣板

```go
package tools

import (
    "context"
    "fmt"

    "github.com/modelcontextprotocol/go-sdk/mcp"
)

type Tool1Input struct {
    Param1 string `json:"param1" jsonschema:"required,description=First parameter"`
    Param2 int    `json:"param2,omitempty" jsonschema:"description=Optional second parameter"`
}

type Tool1Output struct {
    Result string `json:"result" jsonschema:"description=The result of the operation"`
    Status string `json:"status" jsonschema:"description=Operation status"`
}

func Tool1Handler(ctx context.Context, req *mcp.CallToolRequest, input Tool1Input) (
    *mcp.CallToolResult,
    Tool1Output,
    error,
) {
    // Validate input
    if input.Param1 == "" {
        return nil, Tool1Output{}, fmt.Errorf("param1 is required")
    }

    // Check context
    if ctx.Err() != nil {
        return nil, Tool1Output{}, ctx.Err()
    }

    // Perform operation
    result := fmt.Sprintf("Processed: %s", input.Param1)

    return nil, Tool1Output{
        Result: result,
        Status: "success",
    }, nil
}

func RegisterTool1(server *mcp.Server) {
    mcp.AddTool(server,
        &mcp.Tool{
            Name:        "tool1",
            Description: "Description of what tool1 does",
        },
        Tool1Handler,
    )
}
```

## tools/registry.go 樣板

```go
package tools

import "github.com/modelcontextprotocol/go-sdk/mcp"

func RegisterTools(server *mcp.Server) {
    RegisterTool1(server)
    RegisterTool2(server)
    // Register additional tools here
}
```

## config/config.go 樣板

```go
package config

import "os"

type Config struct {
    ServerName string
    Version    string
    LogLevel   string
}

func Load() *Config {
    return &Config{
        ServerName: getEnv("SERVER_NAME", "{{PROJECT_NAME}}"),
        Version:    getEnv("VERSION", "v1.0.0"),
        LogLevel:   getEnv("LOG_LEVEL", "info"),
    }
}

func getEnv(key, defaultValue string) string {
    if value := os.Getenv(key); value != "" {
        return value
    }
    return defaultValue
}
```

## main_test.go 樣板

```go
package main

import (
    "context"
    "testing"

    "github.com/yourusername/{{PROJECT_NAME}}/tools"
)

func TestTool1Handler(t *testing.T) {
    ctx := context.Background()
    input := tools.Tool1Input{
        Param1: "test",
        Param2: 42,
    }

    result, output, err := tools.Tool1Handler(ctx, nil, input)
    if err != nil {
        t.Fatalf("Tool1Handler failed: %v", err)
    }

    if output.Status != "success" {
        t.Errorf("Expected status 'success', got '%s'", output.Status)
    }

    if result != nil {
        t.Error("Expected result to be nil")
    }
}
```

## README.md 樣板

```markdown
# {{PROJECT_NAME}}

A Model Context Protocol (MCP) server built with Go.

## Description

{{PROJECT_DESCRIPTION}}

## Installation

\`\`\`bash
go mod download
go build -o {{PROJECT_NAME}}
\`\`\`

## Usage

Run the server with stdio transport:

\`\`\`bash
./{{PROJECT_NAME}}
\`\`\`

## Configuration

Configure via environment variables:

- `SERVER_NAME`: Server name (default: "{{PROJECT_NAME}}")
- `VERSION`: Server version (default: "v1.0.0")
- `LOG_LEVEL`: Logging level (default: "info")

## Available Tools

### tool1
{{TOOL1_DESCRIPTION}}

**Input:**
- `param1` (string, required): First parameter
- `param2` (int, optional): Second parameter

**Output:**
- `result` (string): Operation result
- `status` (string): Status of the operation

## Development

Run tests:

\`\`\`bash
go test ./...
\`\`\`

Build:

\`\`\`bash
go build -o {{PROJECT_NAME}}
\`\`\`

## License

MIT
```

## 產生指示

在產生 Go MCP 伺服器時：

1. **初始化模組**：使用適當的模組路徑建立 `go.mod`
2. **結構**：遵循樣板目錄結構
3. **型別安全**：為所有輸入/輸出使用具有 JSON schema 標籤的結構體
4. **錯誤處理**：驗證輸入、檢查上下文、包裝錯誤
5. **文件**：新增清楚的說明及範例
6. **測試**：每個工具至少包含一個測試
7. **設定**：使用環境變數進行設定
8. **日誌記錄**：使用結構化日誌 (log/slog)
9. **優雅關閉**：適當處理信號
10. **傳輸**：預設使用標準輸入/輸出，文件記錄其他方案

## 最佳實踐

- 保持工具專注且單一目的
- 為型別及函式使用描述性名稱
- 在結構體標籤中包含 JSON schema 文件
- 總是尊重上下文取消
- 返回描述性錯誤
- 保持 main.go 最小化，邏輯放在套件中
- 為工具處理器撰寫測試
- 記錄所有匯出的函式
