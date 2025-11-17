---
description: '使用 Python 的 LangChain 指示'
applyTo: "**/*.py"
---

# LangChain Python 指示

這些指示指導 GitHub Copilot 為 Python 中的 LangChain 應用程式生成程式碼和文件。專注於 LangChain 特定的模式、API 和最佳實踐。

## Runnable 介面（LangChain 特定）

LangChain 的 `Runnable` 介面是組合和執行鏈、聊天模型、輸出解析器、檢索器和 LangGraph 圖的基礎。它提供了一個統一的 API，用於呼叫、批次處理、串流、檢查和組合元件。

**LangChain 特定的關鍵功能：**

- 所有主要的 LangChain 元件（聊天模型、輸出解析器、檢索器、圖）都實現了 Runnable 介面。
- 支援同步（`invoke`、`batch`、`stream`）和非同步（`ainvoke`、`abatch`、`astream`）執行。
- 批次處理（`batch`、`batch_as_completed`）針對平行 API 呼叫進行了優化；在 `RunnableConfig` 中設定 `max_concurrency` 來控制平行度。
- 串流 API（`stream`、`astream`、`astream_events`）在產生輸出時就產生結果，對於響應式 LLM 應用程式至關重要。
- 輸入/輸出類型是特定於元件的（例如，聊天模型接受訊息，檢索器接受字串，輸出解析器接受模型輸出）。
- 使用 `get_input_schema`、`get_output_schema` 及其 JSONSchema 變體檢查模式，用於驗證和 OpenAPI 生成。
- 使用 `with_types` 來覆蓋複雜 LCEL 鏈的推斷輸入/輸出類型。
- 使用 LCEL 宣告式組合 Runnables：`chain = prompt | chat_model | output_parser`。
- 在 Python 3.11+ 中自動傳播 `RunnableConfig`（標籤、元數據、回呼、並發性）；在 Python 3.9/3.10 的非同步程式碼中手動傳播。
- 使用 `RunnableLambda`（簡單轉換）或 `RunnableGenerator`（串流轉換）建立自訂 runnables；避免直接子類化。
- 使用 `configurable_fields` 和 `configurable_alternatives` 配置執行時屬性和替代方案，用於動態鏈和 LangServe 部署。

**LangChain 最佳實踐：**

- 使用批次處理來平行呼叫 LLM 或檢索器的 API；設定 `max_concurrency` 以避免速率限制。
- 優先使用串流 API 用於聊天 UI 和長輸出。
- 始終驗證自訂鏈和部署端點的輸入/輸出模式。
- 在 `RunnableConfig` 中使用標籤和元數據，用於 LangSmith 中的追蹤和除錯複雜鏈。
- 對於自訂邏輯，使用 `RunnableLambda` 或 `RunnableGenerator` 包裝函數，而不是子類化。
- 對於進階配置，通過 `configurable_fields` 和 `configurable_alternatives` 暴露欄位和替代方案。


- 使用 LangChain 的聊天模型整合進行對話式 AI：

- 從 `langchain.chat_models` 或 `langchain_openai` 匯入（例如，`ChatOpenAI`）。
- 使用 `SystemMessage`、`HumanMessage`、`AIMessage` 組合訊息。
- 對於工具呼叫，使用 `bind_tools(tools)` 方法。
- 對於結構化輸出，使用 `with_structured_output(schema)`。

範例：
```python
from langchain_openai import ChatOpenAI
from langchain.schema import HumanMessage, SystemMessage

chat = ChatOpenAI(model="gpt-4", temperature=0)
messages = [
    SystemMessage(content="You are a helpful assistant."),
    HumanMessage(content="What is LangChain?")
]
response = chat.invoke(messages)
print(response.content)
```

- 將訊息組合為 `SystemMessage`、`HumanMessage` 以及可選的 `AIMessage` 物件的清單。
- 對於 RAG，將聊天模型與檢索器/向量儲存結合以進行上下文注入。
- 使用 `streaming=True` 進行即時 token 串流（如果支援）。
- 使用 `tools` 參數進行函數/工具呼叫（OpenAI、Anthropic 等）。
- 使用 `response_format="json"` 進行結構化輸出（OpenAI 模型）。

最佳實踐：

- 在下游任務中使用模型輸出之前，始終驗證它們。
- 為了清晰和可靠性，優先使用明確的訊息類型。
- 對於 Copilot，提供清晰、可操作的提示並記錄預期輸出。



- LLM 客戶端工廠：集中提供者配置（API 金鑰）、超時、重試和遙測。提供一個單一的地方來切換提供者或客戶端設定。
- 提示範本：將範本儲存在 `prompts/` 下，並通過安全的輔助程式載入。保持範本小而可測試。
- 鏈 vs 代理：對於確定性管線（RAG、摘要），優先使用鏈。當你需要規劃或動態工具選擇時，使用代理。
- 工具：為工具實現型別化適配器介面；嚴格驗證輸入和輸出。
- 記憶體：預設為無狀態設計。當需要記憶體時，儲存最少的上下文並記錄保留/刪除策略。
- 檢索器：建立檢索 + 重新排名管線。保持向量儲存模式穩定（id、text、metadata）。

### 模式

- 回呼和追蹤：使用 LangChain 回呼並與 LangSmith 或你的追蹤系統整合，以捕獲請求/回應生命週期。
- 關注點分離：將提示構建、LLM 連接和業務邏輯分開，以簡化測試並減少意外的提示更改。

## 嵌入和向量儲存

- 使用一致的分塊和元數據欄位（source、page、chunk_index）。
- 快取嵌入以避免對未更改的文件重複計費。
- 本地/開發：Chroma 或 FAISS。生產：根據規模和 SLA 使用託管向量資料庫（Pinecone、Qdrant、Milvus、Weaviate）。

## 向量儲存（LangChain 特定）

- 使用 LangChain 的向量儲存整合進行語義搜尋、檢索增強生成（RAG）和文件相似性工作流程。
- 始終使用支援的嵌入模型（例如，OpenAIEmbeddings、HuggingFaceEmbeddings）初始化向量儲存。
- 優先使用官方整合（例如，Chroma、FAISS、Pinecone、Qdrant、Weaviate）用於生產；對於測試和示範使用 InMemoryVectorStore。
- 將文件儲存為帶有 `page_content` 和 `metadata` 的 LangChain `Document` 物件。
- 使用 `add_documents(documents, ids=...)` 來新增/更新文件。始終為更新插入提供唯一 ID。
- 使用 `delete(ids=...)` 按 ID 刪除文件。
- 使用 `similarity_search(query, k=4, filter={...})` 檢索前 k 個相似文件。使用元數據篩選進行範圍搜尋。
- 對於 RAG，將你的向量儲存連接到檢索器並與 LLM 鏈接（參見 LangChain Retriever 和 RAGChain 文件）。
- 對於進階搜尋，使用向量儲存特定選項：Pinecone 支援混合搜尋和元數據篩選；Chroma 支援篩選和自訂距離度量。
- 始終在你的環境中驗證向量儲存整合和 API 版本；LangChain 版本之間的破壞性變更很常見。
- 範例（InMemoryVectorStore）：

```python
from langchain_core.vectorstores import InMemoryVectorStore
from langchain_openai import OpenAIEmbeddings
from langchain_core.documents import Document

embedding_model = OpenAIEmbeddings()
vector_store = InMemoryVectorStore(embedding=embedding_model)

documents = [Document(page_content="LangChain content", metadata={"source": "doc1"})]
vector_store.add_documents(documents=documents, ids=["doc1"])

results = vector_store.similarity_search("What is RAG?", k=2)
for doc in results:
    print(doc.page_content, doc.metadata)
```

- 對於生產環境，優先使用持久向量儲存（Chroma、Pinecone、Qdrant、Weaviate），並根據提供者文件配置身份驗證、擴展和備份。
- 參考：https://python.langchain.com/docs/integrations/vectorstores/

## 提示工程和治理

- 將規範提示儲存在 `prompts/` 下，並從程式碼中按檔名引用它們。
- 編寫單元測試，斷言所需的佔位符存在並且渲染的提示符合預期模式（長度、變數存在）。
- 維護影響行為的提示和模式變更的 CHANGELOG。

## 聊天模型

LangChain 為聊天模型提供了一致的介面，並具有用於監控、除錯和優化的額外功能。

### 整合

整合有兩種：

1. 官方：由 LangChain 團隊或提供者維護的打包 `langchain-<provider>` 整合。
2. 社群：貢獻的整合（在 `langchain-community` 中）。

聊天模型通常遵循帶有 `Chat` 前綴的命名慣例（例如，`ChatOpenAI`、`ChatAnthropic`、`ChatOllama`）。沒有 `Chat` 前綴（或帶有 `LLM` 後綴）的模型通常實現較舊的字串輸入/字串輸出介面，對於現代聊天工作流程較不受歡迎。

### 介面

聊天模型實現 `BaseChatModel` 並支援 Runnable 介面：串流、非同步、批次處理等。許多操作接受並返回 LangChain `messages`（角色如 `system`、`user`、`assistant`）。有關詳細資訊，請參見 BaseChatModel API 參考。

關鍵方法包括：

- `invoke(messages, ...)` — 發送訊息清單並接收回應。
- `stream(messages, ...)` — 在 token 到達時串流部分輸出。
- `batch(inputs, ...)` — 批次處理多個請求。
- `bind_tools(tools)` — 附加工具適配器用於工具呼叫。
- `with_structured_output(schema)` — 請求結構化回應的輔助程式。

### 輸入和輸出

- LangChain 支援其自己的訊息格式和 OpenAI 的訊息格式；在你的程式碼庫中一致地選擇一個。
- 訊息包括 `role` 和 `content` 區塊；在支援的地方，內容可以包括結構化或多模態負載。

### 標準參數

常見支援的參數（取決於提供者）：

- `model`：模型識別碼（例如 `gpt-4o`、`gpt-3.5-turbo`）。
- `temperature`：隨機性控制（0.0 確定性 — 1.0 創意性）。
- `timeout`：在取消之前等待的秒數。
- `max_tokens`：回應 token 限制。
- `stop`：停止序列。
- `max_retries`：網路/限制失敗的重試次數。
- `api_key`、`base_url`：提供者身份驗證和端點配置。
- `rate_limiter`：可選的 BaseRateLimiter 來間隔請求並避免提供者配額錯誤。

> 注意：並非所有提供者都實現了每個參數。始終查閱提供者整合文件。

### 工具呼叫

聊天模型可以呼叫工具（API、資料庫、系統適配器）。使用 LangChain 的工具呼叫 API 來：

- 使用嚴格的輸入/輸出型別註冊工具。
- 觀察和記錄工具呼叫請求和結果。
- 在將工具輸出傳遞回模型或執行副作用之前驗證它們。

有關範例和安全模式，請參見 LangChain 文件中的工具呼叫指南。

### 結構化輸出

使用 `with_structured_output` 或模式強制方法從模型請求 JSON 或型別化輸出。結構化輸出對於可靠的提取和下游處理（解析器、資料庫寫入、分析）至關重要。

### 多模態

某些模型支援多模態輸入（圖像、音訊）。查看提供者文件以了解支援的輸入類型和限制。多模態輸出很少見——將它們視為實驗性並嚴格驗證。

### 上下文視窗

模型具有以 token 測量的有限上下文視窗。在設計對話流程時：

- 保持訊息簡潔並優先考慮重要上下文。
- 當超過視窗時，在模型外部修剪舊上下文（總結或歸檔）。
- 使用檢索器 + RAG 模式來顯示相關的長格式上下文，而不是將大型文件貼到聊天中。

## 進階主題

### 速率限制

- 初始化聊天模型時使用 `rate_limiter` 來間隔呼叫。
- 實現指數退避重試，並在受到限制時考慮後備模型或降級模式。

### 快取

- 對於對話，精確輸入快取通常是無效的。考慮語義快取（基於嵌入）以處理重複的意義級查詢。
- 語義快取引入了對嵌入的依賴，並非普遍適用。
- 僅在減少成本並滿足正確性要求的地方快取（例如，FAQ 機器人）。

## 最佳實踐

- 使用型別提示和資料類別用於公共 API。
- 在呼叫 LLM 或工具之前驗證輸入。
- 從秘密管理器載入秘密；永遠不要記錄秘密或未編輯的模型輸出。
- 確定性測試：模擬 LLM 和嵌入呼叫。
- 快取嵌入和頻繁的檢索結果。
- 可觀察性：記錄 request_id、模型名稱、延遲和清理後的 token 計數。
- 對外部呼叫實現指數退避和冪等性。

## 安全性和隱私

- 將模型輸出視為不受信任。在執行生成的程式碼或系統命令之前進行清理。
- 驗證任何使用者提供的 URL 和輸入，以避免 SSRF 和注入攻擊。
- 記錄資料保留並新增 API 以根據請求刪除使用者資料。
- 限制儲存的 PII 並在靜態時加密敏感欄位。
