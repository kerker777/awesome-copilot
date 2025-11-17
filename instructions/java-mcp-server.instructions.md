---
description: 'Best practices and patterns for building Model Context Protocol (MCP) servers in Java using the official MCP Java SDK with reactive streams and Spring integration.'
applyTo: "**/*.java, **/pom.xml, **/build.gradle, **/build.gradle.kts"
---

# Java MCP 伺服器開發指南

在 Java 中建構 MCP 伺服器時，請遵循使用官方 Java SDK 的以下最佳實踐和模式。

## 依賴項

將 MCP Java SDK 新增至你的 Maven 專案：

```xml
<dependencies>
    <dependency>
        <groupId>io.modelcontextprotocol.sdk</groupId>
        <artifactId>mcp</artifactId>
        <version>0.14.1</version>
    </dependency>
</dependencies>
```

或使用 Gradle：

```kotlin
dependencies {
    implementation("io.modelcontextprotocol.sdk:mcp:0.14.1")
}
```

## 伺服器設定

使用建構器模式建立 MCP 伺服器：

```java
import io.mcp.server.McpServer;
import io.mcp.server.McpServerBuilder;
import io.mcp.server.transport.StdioServerTransport;

McpServer server = McpServerBuilder.builder()
    .serverInfo("my-server", "1.0.0")
    .capabilities(capabilities -> capabilities
        .tools(true)
        .resources(true)
        .prompts(true))
    .build();

// 使用 stdio 傳輸啟動
StdioServerTransport transport = new StdioServerTransport();
server.start(transport).subscribe();
```

## 新增工具

向伺服器註冊工具處理器：

```java
import io.mcp.server.tool.Tool;
import io.mcp.server.tool.ToolHandler;
import reactor.core.publisher.Mono;

// 定義工具
Tool searchTool = Tool.builder()
    .name("search")
    .description("Search for information")
    .inputSchema(JsonSchema.object()
        .property("query", JsonSchema.string()
            .description("Search query")
            .required(true))
        .property("limit", JsonSchema.integer()
            .description("Maximum results")
            .defaultValue(10)))
    .build();

// 註冊工具處理器
server.addToolHandler("search", (arguments) -> {
    String query = arguments.get("query").asText();
    int limit = arguments.has("limit")
        ? arguments.get("limit").asInt()
        : 10;

    // 執行搜尋
    List<String> results = performSearch(query, limit);

    return Mono.just(ToolResponse.success()
        .addTextContent("Found " + results.size() + " results")
        .build());
});
```

## 新增資源

實現資源處理器以存取資料：

```java
import io.mcp.server.resource.Resource;
import io.mcp.server.resource.ResourceHandler;

// 註冊資源清單處理器
server.addResourceListHandler(() -> {
    List<Resource> resources = List.of(
        Resource.builder()
            .name("Data File")
            .uri("resource://data/example.txt")
            .description("Example data file")
            .mimeType("text/plain")
            .build()
    );
    return Mono.just(resources);
});

// 註冊資源讀取處理器
server.addResourceReadHandler((uri) -> {
    if (uri.equals("resource://data/example.txt")) {
        String content = loadResourceContent(uri);
        return Mono.just(ResourceContent.text(content, uri));
    }
    throw new ResourceNotFoundException(uri);
});

// 註冊資源訂閱處理器
server.addResourceSubscribeHandler((uri) -> {
    subscriptions.add(uri);
    log.info("Client subscribed to {}", uri);
    return Mono.empty();
});
```

## 新增提示

實現提示處理器以進行模板化對話：

```java
import io.mcp.server.prompt.Prompt;
import io.mcp.server.prompt.PromptMessage;
import io.mcp.server.prompt.PromptArgument;

// 註冊提示清單處理器
server.addPromptListHandler(() -> {
    List<Prompt> prompts = List.of(
        Prompt.builder()
            .name("analyze")
            .description("Analyze a topic")
            .argument(PromptArgument.builder()
                .name("topic")
                .description("Topic to analyze")
                .required(true)
                .build())
            .argument(PromptArgument.builder()
                .name("depth")
                .description("Analysis depth")
                .required(false)
                .build())
            .build()
    );
    return Mono.just(prompts);
});

// 註冊提示取得處理器
server.addPromptGetHandler((name, arguments) -> {
    if (name.equals("analyze")) {
        String topic = arguments.getOrDefault("topic", "general");
        String depth = arguments.getOrDefault("depth", "basic");

        List<PromptMessage> messages = List.of(
            PromptMessage.user("Please analyze this topic: " + topic),
            PromptMessage.assistant("I'll provide a " + depth + " analysis of " + topic)
        );

        return Mono.just(PromptResult.builder()
            .description("Analysis of " + topic + " at " + depth + " level")
            .messages(messages)
            .build());
    }
    throw new PromptNotFoundException(name);
});
```

## 反應式串流模式

Java SDK 使用反應式串流（Project Reactor）進行非同步處理：

```java
// 單一結果傳回 Mono
server.addToolHandler("process", (args) -> {
    return Mono.fromCallable(() -> {
        String result = expensiveOperation(args);
        return ToolResponse.success()
            .addTextContent(result)
            .build();
    }).subscribeOn(Schedulers.boundedElastic());
});

// 串流結果傳回 Flux
server.addResourceListHandler(() -> {
    return Flux.fromIterable(getResources())
        .map(r -> Resource.builder()
            .uri(r.getUri())
            .name(r.getName())
            .build())
        .collectList();
});
```

## 同步外觀

對於阻塞使用案例，請使用同步 API：

```java
import io.mcp.server.McpSyncServer;

McpSyncServer syncServer = server.toSyncServer();

// 阻塞工具處理器
syncServer.addToolHandler("greet", (args) -> {
    String name = args.get("name").asText();
    return ToolResponse.success()
        .addTextContent("Hello, " + name + "!")
        .build();
});
```

## 傳輸設定

### Stdio 傳輸

用於本機子程序通訊：

```java
import io.mcp.server.transport.StdioServerTransport;

StdioServerTransport transport = new StdioServerTransport();
server.start(transport).block();
```

### HTTP 傳輸（Servlet）

對於基於 HTTP 的伺服器：

```java
import io.mcp.server.transport.ServletServerTransport;
import jakarta.servlet.http.HttpServlet;

public class McpServlet extends HttpServlet {
    private final McpServer server;
    private final ServletServerTransport transport;

    public McpServlet() {
        this.server = createMcpServer();
        this.transport = new ServletServerTransport();
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) {
        transport.handleRequest(server, req, resp).block();
    }
}
```

## Spring Boot 整合

使用 Spring Boot 啟動器以無縫整合：

```xml
<dependency>
    <groupId>io.modelcontextprotocol.sdk</groupId>
    <artifactId>mcp-spring-boot-starter</artifactId>
    <version>0.14.1</version>
</dependency>
```

使用 Spring 設定伺服器：

```java
import org.springframework.context.annotation.Configuration;
import io.mcp.spring.McpServerConfigurer;

@Configuration
public class McpConfiguration {

    @Bean
    public McpServerConfigurer mcpServerConfigurer() {
        return server -> server
            .serverInfo("spring-server", "1.0.0")
            .capabilities(cap -> cap
                .tools(true)
                .resources(true)
                .prompts(true));
    }
}
```

將處理器註冊為 Spring Bean：

```java
import org.springframework.stereotype.Component;
import io.mcp.spring.ToolHandler;

@Component
public class SearchToolHandler implements ToolHandler {

    @Override
    public String getName() {
        return "search";
    }

    @Override
    public Tool getTool() {
        return Tool.builder()
            .name("search")
            .description("Search for information")
            .inputSchema(JsonSchema.object()
                .property("query", JsonSchema.string().required(true)))
            .build();
    }

    @Override
    public Mono<ToolResponse> handle(JsonNode arguments) {
        String query = arguments.get("query").asText();
        return Mono.just(ToolResponse.success()
            .addTextContent("Search results for: " + query)
            .build());
    }
}
```

## 錯誤處理

使用 MCP 例外進行適當的錯誤處理：

```java
server.addToolHandler("risky", (args) -> {
    return Mono.fromCallable(() -> {
        try {
            String result = riskyOperation(args);
            return ToolResponse.success()
                .addTextContent(result)
                .build();
        } catch (ValidationException e) {
            return ToolResponse.error()
                .message("Invalid input: " + e.getMessage())
                .build();
        } catch (Exception e) {
            log.error("Unexpected error", e);
            return ToolResponse.error()
                .message("Internal error occurred")
                .build();
        }
    });
});
```

## JSON 架構建構

使用流暢的架構建構器：

```java
import io.mcp.json.JsonSchema;

JsonSchema schema = JsonSchema.object()
    .property("name", JsonSchema.string()
        .description("User's name")
        .minLength(1)
        .maxLength(100)
        .required(true))
    .property("age", JsonSchema.integer()
        .description("User's age")
        .minimum(0)
        .maximum(150))
    .property("email", JsonSchema.string()
        .description("Email address")
        .format("email")
        .required(true))
    .property("tags", JsonSchema.array()
        .items(JsonSchema.string())
        .uniqueItems(true))
    .additionalProperties(false)
    .build();
```

## 日誌和可觀測性

使用 SLF4J 進行日誌記錄：

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

private static final Logger log = LoggerFactory.getLogger(MyMcpServer.class);

server.addToolHandler("process", (args) -> {
    log.info("Tool called: process, args: {}", args);

    return Mono.fromCallable(() -> {
        String result = process(args);
        log.debug("Processing completed successfully");
        return ToolResponse.success()
            .addTextContent(result)
            .build();
    }).doOnError(error -> {
        log.error("Processing failed", error);
    });
});
```

使用 Reactor 傳播內容：

```java
import reactor.util.context.Context;

server.addToolHandler("traced", (args) -> {
    return Mono.deferContextual(ctx -> {
        String traceId = ctx.get("traceId");
        log.info("Processing with traceId: {}", traceId);

        return Mono.just(ToolResponse.success()
            .addTextContent("Processed")
            .build());
    });
});
```

## 測試

使用同步 API 編寫測試：

```java
import org.junit.jupiter.api.Test;
import static org.assertj.core.Assertions.assertThat;

class McpServerTest {

    @Test
    void testToolHandler() {
        McpServer server = createTestServer();
        McpSyncServer syncServer = server.toSyncServer();

        JsonNode args = objectMapper.createObjectNode()
            .put("query", "test");

        ToolResponse response = syncServer.callTool("search", args);

        assertThat(response.isError()).isFalse();
        assertThat(response.getContent()).hasSize(1);
    }
}
```

## Jackson 整合

SDK 使用 Jackson 進行 JSON 序列化。根據需要自訂：

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;

ObjectMapper mapper = new ObjectMapper();
mapper.registerModule(new JavaTimeModule());

// 與伺服器使用自訂對應程式
McpServer server = McpServerBuilder.builder()
    .objectMapper(mapper)
    .build();
```

## 內容型別

支援回應中的多個內容型別：

```java
import io.mcp.server.content.Content;

server.addToolHandler("multi", (args) -> {
    return Mono.just(ToolResponse.success()
        .addTextContent("Plain text response")
        .addImageContent(imageBytes, "image/png")
        .addResourceContent("resource://data", "application/json", jsonData)
        .build());
});
```

## 伺服器生命週期

正確管理伺服器生命週期：

```java
import reactor.core.Disposable;

Disposable serverDisposable = server.start(transport).subscribe();

// 妥善關閉
Runtime.getRuntime().addShutdownHook(new Thread(() -> {
    log.info("Shutting down MCP server");
    serverDisposable.dispose();
    server.stop().block();
}));
```

## 常見模式

### 要求驗證

```java
server.addToolHandler("validate", (args) -> {
    if (!args.has("required_field")) {
        return Mono.just(ToolResponse.error()
            .message("Missing required_field")
            .build());
    }

    return processRequest(args);
});
```

### 非同步作業

```java
server.addToolHandler("async", (args) -> {
    return Mono.fromCallable(() -> callExternalApi(args))
        .timeout(Duration.ofSeconds(30))
        .onErrorResume(TimeoutException.class, e ->
            Mono.just(ToolResponse.error()
                .message("Operation timed out")
                .build()))
        .subscribeOn(Schedulers.boundedElastic());
});
```

### 資源快取

```java
private final Map<String, String> cache = new ConcurrentHashMap<>();

server.addResourceReadHandler((uri) -> {
    return Mono.fromCallable(() ->
        cache.computeIfAbsent(uri, this::loadResource))
        .map(content -> ResourceContent.text(content, uri));
});
```

## 最佳實踐

1. **使用反應式串流**進行非同步作業和背壓處理
2. **利用 Spring Boot** 啟動器用於企業應用程式
3. **實現適當的錯誤處理**包含具體的錯誤訊息
4. **使用 SLF4J** 進行日誌記錄，不是 System.out
5. **驗證輸入**在工具和提示處理器中
6. **支援妥善關閉**進行適當的資源清理
7. **使用有限彈性排程器**用於阻塞作業
8. **傳播內容**用於反應式鏈中的可觀測性
9. **使用同步 API 進行測試**為了簡單起見
10. **遵循 Java 命名慣例**（方法的駝峰大小寫，類別的 PascalCase）
