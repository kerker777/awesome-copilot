---
description: '使用 Python SDK 構建模型上下文協議 (MCP) 伺服器的指引'
applyTo: '**/*.py, **/pyproject.toml, **/requirements.txt'
---

# Python MCP 伺服器開發

## 指引

- 使用 **uv** 進行專案管理：`uv init mcp-server-demo` 和 `uv add "mcp[cli]"`
- 從 `mcp.server.fastmcp` 匯入 FastMCP：`from mcp.server.fastmcp import FastMCP`
- 使用 `@mcp.tool()`、`@mcp.resource()` 和 `@mcp.prompt()` 裝飾器進行註冊
- 型別提示是強制性的 - 它們用於生成結構描述和驗證
- 使用 Pydantic 模型、TypedDicts 或 dataclasses 進行結構化輸出
- 當回傳型別相容時，工具會自動回傳結構化輸出
- 對於 stdio 傳輸，使用 `mcp.run()` 或 `mcp.run(transport="stdio")`
- 對於 HTTP 伺服器，使用 `mcp.run(transport="streamable-http")` 或掛載到 Starlette/FastAPI
- 在工具/資源中使用 `Context` 參數來存取 MCP 功能：`ctx: Context`
- 使用 `await ctx.debug()`、`await ctx.info()`、`await ctx.warning()`、`await ctx.error()` 發送日誌
- 使用 `await ctx.report_progress(progress, total, message)` 回報進度
- 使用 `await ctx.elicit(message, schema)` 請求使用者輸入
- 使用 `await ctx.session.create_message(messages, max_tokens)` 進行 LLM 採樣
- 使用 `Icon(src="path", mimeType="image/png")` 為伺服器、工具、資源、提示配置圖示
- 使用 `Image` 類別進行自動圖片處理：`return Image(data=bytes, format="png")`
- 使用 URI 模式定義資源範本：`@mcp.resource("greeting://{name}")`
- 透過接受部分值並回傳建議來實作完成支援
- 使用生命週期上下文管理器來處理啟動/關閉與共享資源
- 透過 `ctx.request_context.lifespan_context` 在工具中存取生命週期上下文
- 對於無狀態 HTTP 伺服器，在 FastMCP 初始化時設定 `stateless_http=True`
- 為現代客戶端啟用 JSON 回應：`json_response=True`
- 使用以下方式測試伺服器：`uv run mcp dev server.py`（檢查器）或 `uv run mcp install server.py`（Claude Desktop）
- 在 Starlette 中使用不同路徑掛載多個伺服器：`Mount("/path", mcp.streamable_http_app())`
- 為瀏覽器客戶端配置 CORS：公開 `Mcp-Session-Id` 標頭
- 當 FastMCP 不足以滿足需求時，使用低階 Server 類別以獲得最大控制權

## 最佳實踐

- 始終使用型別提示 - 它們驅動結構描述生成和驗證
- 回傳 Pydantic 模型或 TypedDicts 以獲得結構化工具輸出
- 保持工具函式專注於單一職責
- 提供清晰的文檔字串 - 它們會成為工具描述
- 使用具有型別提示的描述性參數名稱
- 使用 Pydantic Field 描述來驗證輸入
- 使用 try-except 區塊實作適當的錯誤處理
- 對於 I/O 密集型操作使用非同步函式
- 在生命週期上下文管理器中清理資源
- 記錄到 stderr 以避免干擾 stdio 傳輸（使用 stdio 時）
- 使用環境變數進行配置
- 在 LLM 整合之前獨立測試工具
- 在公開檔案系統或網路存取時考慮安全性
- 使用結構化輸出獲得機器可讀資料
- 提供內容和結構化資料以實現向後相容性

## 常見模式

### 基本伺服器設定（stdio）
```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("My Server")

@mcp.tool()
def calculate(a: int, b: int, op: str) -> int:
    """執行計算"""
    if op == "add":
        return a + b
    return a - b

if __name__ == "__main__":
    mcp.run()  # 預設為 stdio
```

### HTTP 伺服器
```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("My HTTP Server")

@mcp.tool()
def hello(name: str = "World") -> str:
    """問候某人"""
    return f"Hello, {name}!"

if __name__ == "__main__":
    mcp.run(transport="streamable-http")
```

### 具有結構化輸出的工具
```python
from pydantic import BaseModel, Field

class WeatherData(BaseModel):
    temperature: float = Field(description="攝氏溫度")
    condition: str
    humidity: float

@mcp.tool()
def get_weather(city: str) -> WeatherData:
    """取得城市天氣"""
    return WeatherData(
        temperature=22.5,
        condition="sunny",
        humidity=65.0
    )
```

### 動態資源
```python
@mcp.resource("users://{user_id}")
def get_user(user_id: str) -> str:
    """取得使用者個人資料資料"""
    return f"User {user_id} profile data"
```

### 具有上下文的工具
```python
from mcp.server.fastmcp import Context
from mcp.server.session import ServerSession

@mcp.tool()
async def process_data(
    data: str,
    ctx: Context[ServerSession, None]
) -> str:
    """處理資料並記錄"""
    await ctx.info(f"Processing: {data}")
    await ctx.report_progress(0.5, 1.0, "Halfway done")
    return f"Processed: {data}"
```

### 具有採樣的工具
```python
from mcp.server.fastmcp import Context
from mcp.server.session import ServerSession
from mcp.types import SamplingMessage, TextContent

@mcp.tool()
async def summarize(
    text: str,
    ctx: Context[ServerSession, None]
) -> str:
    """使用 LLM 總結文字"""
    result = await ctx.session.create_message(
        messages=[SamplingMessage(
            role="user",
            content=TextContent(type="text", text=f"Summarize: {text}")
        )],
        max_tokens=100
    )
    return result.content.text if result.content.type == "text" else ""
```

### 生命週期管理
```python
from contextlib import asynccontextmanager
from dataclasses import dataclass
from mcp.server.fastmcp import FastMCP, Context

@dataclass
class AppContext:
    db: Database

@asynccontextmanager
async def app_lifespan(server: FastMCP):
    db = await Database.connect()
    try:
        yield AppContext(db=db)
    finally:
        await db.disconnect()

mcp = FastMCP("My App", lifespan=app_lifespan)

@mcp.tool()
def query(sql: str, ctx: Context) -> str:
    """查詢資料庫"""
    db = ctx.request_context.lifespan_context.db
    return db.execute(sql)
```

### 具有訊息的提示
```python
from mcp.server.fastmcp.prompts import base

@mcp.prompt(title="Code Review")
def review_code(code: str) -> list[base.Message]:
    """建立程式碼審查提示"""
    return [
        base.UserMessage("Review this code:"),
        base.UserMessage(code),
        base.AssistantMessage("I'll review the code for you.")
    ]
```

### 錯誤處理
```python
@mcp.tool()
async def risky_operation(input: str) -> str:
    """可能會失敗的操作"""
    try:
        result = await perform_operation(input)
        return f"Success: {result}"
    except Exception as e:
        return f"Error: {str(e)}"
```
