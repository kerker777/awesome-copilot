---
description: 'Best practices for building Model Context Protocol servers in Rust using the official rmcp SDK with async/await patterns'
applyTo: '**/*.rs'
---

# Rust MCP 伺服器開發最佳實踐

本指南提供了使用官方 Rust SDK (`rmcp`) 建立 Model Context Protocol (MCP) 伺服器的最佳實踐。

## 安裝與設定

### 新增相依套件

將 `rmcp` crate 新增至您的 `Cargo.toml`：

```toml
[dependencies]
rmcp = { version = "0.8.1", features = ["server"] }
tokio = { version = "1", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
anyhow = "1.0"
tracing = "0.1"
tracing-subscriber = "0.3"
```

若需要巨集支援：

```toml
[dependencies]
rmcp-macros = "0.8"
schemars = { version = "0.8", features = ["derive"] }
```

### 專案結構

組織您的 Rust MCP 伺服器專案：

```
my-mcp-server/
├── Cargo.toml
├── src/
│   ├── main.rs           # Server entry point
│   ├── handler.rs        # ServerHandler implementation
│   ├── tools/
│   │   ├── mod.rs
│   │   ├── calculator.rs
│   │   └── greeter.rs
│   ├── prompts/
│   │   ├── mod.rs
│   │   └── code_review.rs
│   └── resources/
│       ├── mod.rs
│       └── data.rs
└── tests/
    └── integration_tests.rs
```

## 伺服器實作

### 基本伺服器設定

以 stdio 傳輸建立伺服器：

```rust
use rmcp::{
    protocol::ServerCapabilities,
    server::{Server, ServerHandler},
    transport::StdioTransport,
};
use tokio::signal;

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    tracing_subscriber::fmt::init();

    let handler = MyServerHandler::new();
    let transport = StdioTransport::new();

    let server = Server::builder()
        .with_handler(handler)
        .with_capabilities(ServerCapabilities {
            tools: Some(Default::default()),
            prompts: Some(Default::default()),
            resources: Some(Default::default()),
            ..Default::default()
        })
        .build(transport)?;

    server.run(signal::ctrl_c()).await?;

    Ok(())
}
```

### ServerHandler 實作

實作 `ServerHandler` trait：

```rust
use rmcp::{
    model::*,
    protocol::*,
    server::{RequestContext, ServerHandler, RoleServer},
    ErrorData,
};

pub struct MyServerHandler {
    tool_router: ToolRouter,
}

impl MyServerHandler {
    pub fn new() -> Self {
        Self {
            tool_router: Self::create_tool_router(),
        }
    }

    fn create_tool_router() -> ToolRouter {
        // Initialize and return tool router
        ToolRouter::new()
    }
}

#[async_trait::async_trait]
impl ServerHandler for MyServerHandler {
    async fn list_tools(
        &self,
        _request: Option<PaginatedRequestParam>,
        _context: RequestContext<RoleServer>,
    ) -> Result<ListToolsResult, ErrorData> {
        let items = self.tool_router.list_all();
        Ok(ListToolsResult::with_all_items(items))
    }

    async fn call_tool(
        &self,
        request: CallToolRequestParam,
        context: RequestContext<RoleServer>,
    ) -> Result<CallToolResult, ErrorData> {
        let tcc = ToolCallContext::new(self, request, context);
        self.tool_router.call(tcc).await
    }
}
```

## 工具開發

### 使用巨集定義工具

使用 `#[tool]` 巨集進行宣告式的工具定義：

```rust
use rmcp::tool;
use rmcp::model::Parameters;
use serde::{Deserialize, Serialize};
use schemars::JsonSchema;

#[derive(Debug, Deserialize, JsonSchema)]
pub struct CalculateParams {
    pub a: f64,
    pub b: f64,
    pub operation: String,
}

/// Performs mathematical calculations
#[tool(
    name = "calculate",
    description = "Performs basic arithmetic operations",
    annotations(read_only_hint = true)
)]
pub async fn calculate(params: Parameters<CalculateParams>) -> Result<f64, String> {
    let p = params.inner();
    match p.operation.as_str() {
        "add" => Ok(p.a + p.b),
        "subtract" => Ok(p.a - p.b),
        "multiply" => Ok(p.a * p.b),
        "divide" => {
            if p.b == 0.0 {
                Err("Division by zero".to_string())
            } else {
                Ok(p.a / p.b)
            }
        }
        _ => Err(format!("Unknown operation: {}", p.operation)),
    }
}
```

### 使用巨集進行工具路由

使用 `#[tool_router]` 和 `#[tool_handler]` 巨集：

```rust
use rmcp::{tool_router, tool_handler};

pub struct ToolsHandler {
    tool_router: ToolRouter,
}

#[tool_router]
impl ToolsHandler {
    #[tool]
    async fn greet(params: Parameters<GreetParams>) -> String {
        format!("Hello, {}!", params.inner().name)
    }

    #[tool(annotations(destructive_hint = true))]
    async fn reset_counter() -> String {
        "Counter reset".to_string()
    }

    pub fn new() -> Self {
        Self {
            tool_router: Self::tool_router(),
        }
    }
}

#[tool_handler]
impl ServerHandler for ToolsHandler {
    // Other handler methods...
}
```

### 工具註記

使用註記來提供關於工具行為的提示：

```rust
#[tool(
    name = "delete_file",
    annotations(
        destructive_hint = true,
        read_only_hint = false,
        idempotent_hint = false
    )
)]
pub async fn delete_file(params: Parameters<DeleteParams>) -> Result<(), String> {
    // Delete file logic
}

#[tool(
    name = "search_data",
    annotations(
        read_only_hint = true,
        idempotent_hint = true,
        open_world_hint = true
    )
)]
pub async fn search_data(params: Parameters<SearchParams>) -> Vec<String> {
    // Search logic
}
```

### 傳回豐富內容

從工具傳回結構化的內容：

```rust
use rmcp::model::{ToolResponseContent, TextContent, ImageContent};

#[tool]
async fn analyze_code(params: Parameters<CodeParams>) -> ToolResponseContent {
    ToolResponseContent::from(vec![
        TextContent::text(format!("Analysis of {}:", params.inner().filename)),
        TextContent::text("No issues found."),
    ])
}
```

## 提示實作

### 提示處理器

實作提示處理器：

```rust
use rmcp::model::{Prompt, PromptArgument, PromptMessage, GetPromptResult};

async fn list_prompts(
    &self,
    _request: Option<PaginatedRequestParam>,
    _context: RequestContext<RoleServer>,
) -> Result<ListPromptsResult, ErrorData> {
    let prompts = vec![
        Prompt {
            name: "code-review".to_string(),
            description: Some("Review code for best practices".to_string()),
            arguments: Some(vec![
                PromptArgument {
                    name: "language".to_string(),
                    description: Some("Programming language".to_string()),
                    required: Some(true),
                },
            ]),
        },
    ];

    Ok(ListPromptsResult { prompts })
}

async fn get_prompt(
    &self,
    request: GetPromptRequestParam,
    _context: RequestContext<RoleServer>,
) -> Result<GetPromptResult, ErrorData> {
    match request.name.as_str() {
        "code-review" => {
            let language = request.arguments
                .as_ref()
                .and_then(|args| args.get("language"))
                .ok_or_else(|| ErrorData::invalid_params("language required"))?;

            Ok(GetPromptResult {
                description: Some("Code review prompt".to_string()),
                messages: vec![
                    PromptMessage::user(format!(
                        "Review this {} code for best practices and suggest improvements",
                        language
                    )),
                ],
            })
        }
        _ => Err(ErrorData::invalid_params("Unknown prompt")),
    }
}
```

## 資源實作

### 資源處理器

實作資源處理器：

```rust
use rmcp::model::{Resource, ResourceContents, ReadResourceResult};

async fn list_resources(
    &self,
    _request: Option<PaginatedRequestParam>,
    _context: RequestContext<RoleServer>,
) -> Result<ListResourcesResult, ErrorData> {
    let resources = vec![
        Resource {
            uri: "file:///data/config.json".to_string(),
            name: "Configuration".to_string(),
            description: Some("Server configuration".to_string()),
            mime_type: Some("application/json".to_string()),
        },
    ];

    Ok(ListResourcesResult { resources })
}

async fn read_resource(
    &self,
    request: ReadResourceRequestParam,
    _context: RequestContext<RoleServer>,
) -> Result<ReadResourceResult, ErrorData> {
    match request.uri.as_str() {
        "file:///data/config.json" => {
            let content = r#"{"version": "1.0", "enabled": true}"#;
            Ok(ReadResourceResult {
                contents: vec![
                    ResourceContents::text(content.to_string())
                        .with_uri(request.uri)
                        .with_mime_type("application/json"),
                ],
            })
        }
        _ => Err(ErrorData::invalid_params("Unknown resource")),
    }
}
```

## 傳輸選項

### Stdio 傳輸

用於 CLI 整合的標準輸入/輸出傳輸：

```rust
use rmcp::transport::StdioTransport;

let transport = StdioTransport::new();
let server = Server::builder()
    .with_handler(handler)
    .build(transport)?;
```

### SSE (伺服器傳送事件) 傳輸

基於 HTTP 的 SSE 傳輸：

```rust
use rmcp::transport::SseServerTransport;
use std::net::SocketAddr;

let addr: SocketAddr = "127.0.0.1:8000".parse()?;
let transport = SseServerTransport::new(addr);

let server = Server::builder()
    .with_handler(handler)
    .build(transport)?;

server.run(signal::ctrl_c()).await?;
```

### 串流式 HTTP 傳輸

使用 Axum 的 HTTP 串流傳輸：

```rust
use rmcp::transport::StreamableHttpTransport;
use axum::{Router, routing::post};

let transport = StreamableHttpTransport::new();
let app = Router::new()
    .route("/mcp", post(transport.handler()));

let listener = tokio::net::TcpListener::bind("127.0.0.1:3000").await?;
axum::serve(listener, app).await?;
```

### 自訂傳輸

實作自訂傳輸（TCP、Unix Socket、WebSocket）：

```rust
use rmcp::transport::Transport;
use tokio::net::TcpListener;

// See examples/transport/ for TCP, Unix Socket, WebSocket implementations
```

## 錯誤處理

### ErrorData 使用

傳回正確的 MCP 錯誤：

```rust
use rmcp::ErrorData;

fn validate_params(value: &str) -> Result<(), ErrorData> {
    if value.is_empty() {
        return Err(ErrorData::invalid_params("Value cannot be empty"));
    }
    Ok(())
}

async fn call_tool(
    &self,
    request: CallToolRequestParam,
    context: RequestContext<RoleServer>,
) -> Result<CallToolResult, ErrorData> {
    validate_params(&request.name)?;

    // Tool execution...

    Ok(CallToolResult {
        content: vec![TextContent::text("Success")],
        is_error: Some(false),
    })
}
```

### Anyhow 整合

使用 `anyhow` 來處理應用程式層級的錯誤：

```rust
use anyhow::{Context, Result};

async fn load_config() -> Result<Config> {
    let content = tokio::fs::read_to_string("config.json")
        .await
        .context("Failed to read config file")?;

    let config: Config = serde_json::from_str(&content)
        .context("Failed to parse config")?;

    Ok(config)
}
```

## 測試

### 單元測試

為工具和處理器編寫單元測試：

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[tokio::test]
    async fn test_calculate_add() {
        let params = Parameters::new(CalculateParams {
            a: 5.0,
            b: 3.0,
            operation: "add".to_string(),
        });

        let result = calculate(params).await.unwrap();
        assert_eq!(result, 8.0);
    }

    #[tokio::test]
    async fn test_divide_by_zero() {
        let params = Parameters::new(CalculateParams {
            a: 5.0,
            b: 0.0,
            operation: "divide".to_string(),
        });

        let result = calculate(params).await;
        assert!(result.is_err());
    }
}
```

### 整合測試

測試完整的伺服器互動：

```rust
#[tokio::test]
async fn test_server_list_tools() {
    let handler = MyServerHandler::new();
    let context = RequestContext::default();

    let result = handler.list_tools(None, context).await.unwrap();

    assert!(!result.tools.is_empty());
    assert!(result.tools.iter().any(|t| t.name == "calculate"));
}
```

## 進度通知

### 報告進度

在長時間執行的操作期間傳送進度通知：

```rust
use rmcp::model::ProgressNotification;

#[tool]
async fn process_large_file(
    params: Parameters<ProcessParams>,
    context: RequestContext<RoleServer>,
) -> Result<String, String> {
    let total = 100;

    for i in 0..=total {
        // Do work...

        if i % 10 == 0 {
            context.notify_progress(ProgressNotification {
                progress: i,
                total: Some(total),
            }).await.ok();
        }
    }

    Ok("Processing complete".to_string())
}
```

## OAuth 驗證

### OAuth 整合

實作 OAuth 以進行安全存取：

```rust
use rmcp::oauth::{OAuthConfig, OAuthProvider};

let oauth_config = OAuthConfig {
    authorization_endpoint: "https://auth.example.com/authorize".to_string(),
    token_endpoint: "https://auth.example.com/token".to_string(),
    client_id: env::var("CLIENT_ID")?,
    client_secret: env::var("CLIENT_SECRET")?,
    scopes: vec!["read".to_string(), "write".to_string()],
};

let oauth_provider = OAuthProvider::new(oauth_config);
// See examples/servers/complex_auth_sse.rs for complete implementation
```

## 效能最佳實踐

### 非同步操作

使用 async/await 進行非阻塞操作：

```rust
#[tool]
async fn fetch_data(params: Parameters<FetchParams>) -> Result<String, String> {
    let client = reqwest::Client::new();
    let response = client
        .get(&params.inner().url)
        .send()
        .await
        .map_err(|e| e.to_string())?;

    let text = response.text().await.map_err(|e| e.to_string())?;
    Ok(text)
}
```

### 狀態管理

使用 `Arc` 和 `RwLock` 來管理共享狀態：

```rust
use std::sync::Arc;
use tokio::sync::RwLock;

pub struct ServerState {
    counter: Arc<RwLock<i32>>,
}

impl ServerState {
    pub fn new() -> Self {
        Self {
            counter: Arc::new(RwLock::new(0)),
        }
    }

    pub async fn increment(&self) -> i32 {
        let mut counter = self.counter.write().await;
        *counter += 1;
        *counter
    }
}
```

## 記錄與追蹤

### 設定追蹤

配置追蹤以獲得可觀測性：

```rust
use tracing::{info, warn, error, debug};
use tracing_subscriber;

fn init_logging() {
    tracing_subscriber::fmt()
        .with_max_level(tracing::Level::DEBUG)
        .with_target(false)
        .with_thread_ids(true)
        .init();
}

#[tool]
async fn my_tool(params: Parameters<MyParams>) -> String {
    debug!("Tool called with params: {:?}", params);
    info!("Processing request");

    // Tool logic...

    info!("Request completed");
    "Done".to_string()
}
```

## 部署

### 二進位分發

建立最佳化的發行版二進位：

```bash
cargo build --release --target x86_64-unknown-linux-gnu
cargo build --release --target x86_64-pc-windows-msvc
cargo build --release --target x86_64-apple-darwin
```

### 交叉編譯

使用 cross 進行跨平台建置：

```bash
cargo install cross
cross build --release --target aarch64-unknown-linux-gnu
```

### Docker 部署

建立 Dockerfile：

```dockerfile
FROM rust:1.75 as builder
WORKDIR /app
COPY . .
RUN cargo build --release

FROM debian:bookworm-slim
RUN apt-get update && apt-get install -y ca-certificates
COPY --from=builder /app/target/release/my-mcp-server /usr/local/bin/
CMD ["my-mcp-server"]
```

## 其他資源

- [rmcp 文件](https://docs.rs/rmcp)
- [rmcp-macros 文件](https://docs.rs/rmcp-macros)
- [範例儲存庫](https://github.com/modelcontextprotocol/rust-sdk/tree/main/examples)
- [MCP 規範](https://spec.modelcontextprotocol.io/)
- [Rust 非同步書籍](https://rust-lang.github.io/async-book/)
