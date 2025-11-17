---

description: '全面的技術無關提示產生器，用於記錄端對端應用程式工作流程。自動偵測專案架構模式、技術堆疊和資料流程模式，以產生詳細的實作藍圖，涵蓋進入點、服務層、資料存取、錯誤處理和測試方法，支援多種技術，包括 .NET、Java/Spring、React 和微服務架構。'

mode: 'agent'
---
# 專案工作流程文件產生器

## 配置變數

```
${PROJECT_TYPE="Auto-detect|.NET|Java|Spring|Node.js|Python|React|Angular|Microservices|Other"}
<!-- 主要技術堆疊 -->

${ENTRY_POINT="API|GraphQL|Frontend|CLI|Message Consumer|Scheduled Job|Custom"}
<!-- 流程的起始點 -->

${PERSISTENCE_TYPE="Auto-detect|SQL Database|NoSQL Database|File System|External API|Message Queue|Cache|None"}
<!-- 資料儲存類型 -->

${ARCHITECTURE_PATTERN="Auto-detect|Layered|Clean|CQRS|Microservices|MVC|MVVM|Serverless|Event-Driven|Other"}
<!-- 主要架構模式 -->

${WORKFLOW_COUNT=1-5}
<!-- 要記錄的工作流程數量 -->

${DETAIL_LEVEL="Standard|Implementation-Ready"}
<!-- 要包含的實作詳細程度 -->

${INCLUDE_SEQUENCE_DIAGRAM=true|false}
<!-- 產生序列圖 -->

${INCLUDE_TEST_PATTERNS=true|false}
<!-- 包含測試方法 -->
```

## 產生的提示

```
"分析程式碼庫並記錄 ${WORKFLOW_COUNT} 個代表性的端對端工作流程，
這些工作流程可作為類似功能的實作範本。使用以下方法：
```

### 初始偵測階段

```
${PROJECT_TYPE == "Auto-detect" ?
  "從檢查程式碼庫結構開始以識別技術：
   - 檢查 .NET 解決方案/專案、Spring 配置、Node.js/Express 檔案等
   - 識別使用的主要程式語言和框架
   - 根據資料夾結構和關鍵元件判斷架構模式"
  : "專注於 ${PROJECT_TYPE} 模式和慣例"}
```

```
${ENTRY_POINT == "Auto-detect" ?
  "透過尋找以下內容來識別典型的進入點：
   - API 控制器或路由定義
   - GraphQL 解析器
   - 發起網路要求的 UI 元件
   - 訊息處理程序或事件訂閱者
   - 排程作業定義"
  : "專注於 ${ENTRY_POINT} 進入點"}
```

```
${PERSISTENCE_TYPE == "Auto-detect" ?
  "透過檢查以下內容來判斷持久化機制：
   - 資料庫內容/連線配置
   - 儲存庫實作
   - ORM 對應
   - 外部 API 用戶端
   - 檔案系統互動"
  : "專注於 ${PERSISTENCE_TYPE} 互動"}
```

### 工作流程文件指示

對於系統中最具代表性的 `${WORKFLOW_COUNT}` 個工作流程：

#### 1. 工作流程概述
   - 提供名稱和工作流程的簡要描述
   - 解釋它所服務的業務目的
   - 識別觸發動作或事件
   - 列出完整工作流程中涉及的所有檔案/類別

#### 2. 進入點實作

**API 進入點：**
```
${ENTRY_POINT == "API" || ENTRY_POINT == "Auto-detect" ?
  "- 記錄接收要求的 API 控制器類別和方法
   - 顯示完整的方法簽章，包括屬性/註解
   - 包含完整的要求 DTO/模型類別定義
   - 記錄驗證屬性和自訂驗證器
   - 顯示驗證/授權屬性和檢查" : ""}
```

**GraphQL 進入點：**
```
${ENTRY_POINT == "GraphQL" || ENTRY_POINT == "Auto-detect" ?
  "- 記錄 GraphQL 解析器類別和方法
   - 顯示查詢/變異的完整結構描述定義
   - 包含輸入類型定義
   - 顯示具有參數處理的解析器方法實作" : ""}
```

**前端進入點：**
```
${ENTRY_POINT == "Frontend" || ENTRY_POINT == "Auto-detect" ?
  "- 記錄發起 API 呼叫的元件
   - 顯示觸發要求的事件處理程序
   - 包含 API 用戶端服務方法
   - 顯示與要求相關的狀態管理程式碼" : ""}
```

**訊息消費者進入點：**
```
${ENTRY_POINT == "Message Consumer" || ENTRY_POINT == "Auto-detect" ?
  "- 記錄訊息處理程序類別和方法
   - 顯示訊息訂閱配置
   - 包含完整的訊息模型定義
   - 顯示反序列化和驗證邏輯" : ""}
```

#### 3. 服務層實作
   - 記錄每個涉及的服務類別及其相依性
   - 顯示具有參數和傳回類型的完整方法簽章
   - 包含具有關鍵業務邏輯的實際方法實作
   - 適用時記錄介面定義
   - 顯示相依性注入註冊模式

**CQRS 模式：**
```
${ARCHITECTURE_PATTERN == "CQRS" || ARCHITECTURE_PATTERN == "Auto-detect" ?
  "- 包含完整的命令/查詢處理程序實作" : ""}
```

**清晰架構模式：**
```
${ARCHITECTURE_PATTERN == "Clean" || ARCHITECTURE_PATTERN == "Auto-detect" ?
  "- 顯示使用案例/互動器實作" : ""}
```

#### 4. 資料對應模式
   - 記錄 DTO 到領域模型的對應程式碼
   - 顯示物件對應器配置或手動對應方法
   - 包含對應期間的驗證邏輯
   - 記錄對應期間建立的任何領域事件

#### 5. 資料存取實作
   - 記錄儲存庫介面及其實作
   - 顯示具有參數和傳回類型的完整方法簽章
   - 包含實際的查詢實作
   - 記錄具有所有屬性的實體/模型類別定義
   - 顯示交易處理模式

**SQL 資料庫模式：**
```
${PERSISTENCE_TYPE == "SQL Database" || PERSISTENCE_TYPE == "Auto-detect" ?
  "- 包含 ORM 配置、註解或 Fluent API 使用
   - 顯示實際的 SQL 查詢或 ORM 陳述式" : ""}
```

**NoSQL 資料庫模式：**
```
${PERSISTENCE_TYPE == "NoSQL Database" || PERSISTENCE_TYPE == "Auto-detect" ?
  "- 顯示文件結構定義
   - 包含文件查詢/更新操作" : ""}
```

#### 6. 回應建構
   - 記錄回應 DTO/模型類別定義
   - 顯示從領域/實體模型到回應模型的對應
   - 包含狀態碼選擇邏輯
   - 記錄錯誤回應結構和產生

#### 7. 錯誤處理模式
   - 記錄工作流程中使用的例外類型
   - 顯示每層的 try/catch 模式
   - 包含全域例外處理程序配置
   - 記錄錯誤記錄實作
   - 顯示重試原則或斷路器模式
   - 包含失敗情境的補償動作

#### 8. 非同步處理模式
   - 記錄背景作業排程程式碼
   - 顯示事件發布實作
   - 包含訊息佇列傳送模式
   - 記錄回呼或 webhook 實作
   - 顯示如何追蹤和監控非同步操作

**測試方法（選用）：**
```
${INCLUDE_TEST_PATTERNS ?
  "9. **測試方法**
     - 記錄每層的單元測試實作
     - 顯示模擬模式和測試裝置設定
     - 包含整合測試實作
     - 記錄測試資料產生方法
     - 顯示 API/控制器測試實作" : ""}
```

**序列圖（選用）：**
```
${INCLUDE_SEQUENCE_DIAGRAM ?
  "10. **序列圖**
      - 產生顯示所有元件的詳細序列圖
      - 包含具有參數類型的方法呼叫
      - 顯示元件之間的傳回值
      - 記錄條件流程和錯誤路徑" : ""}
```

#### 11. 命名慣例
記錄以下的一致模式：
- 控制器命名（例如，`EntityNameController`）
- 服務命名（例如，`EntityNameService`）
- 儲存庫命名（例如，`IEntityNameRepository`）
- DTO 命名（例如，`EntityNameRequest`、`EntityNameResponse`）
- CRUD 操作的方法命名模式
- 變數命名慣例
- 檔案組織模式

#### 12. 實作範本
提供以下的可重複使用程式碼範本：
- 遵循模式建立新的 API 端點
- 實作新的服務方法
- 新增新的儲存庫方法
- 建立新的領域模型類別
- 實作適當的錯誤處理

### 特定技術的實作模式

**.NET 實作模式（如果偵測到）：**
```
${PROJECT_TYPE == ".NET" || PROJECT_TYPE == "Auto-detect" ?
  "- 具有屬性、篩選器和相依性注入的完整控制器類別
   - Startup.cs 或 Program.cs 中的服務註冊
   - Entity Framework DbContext 配置
   - 使用 EF Core 或 Dapper 的儲存庫實作
   - AutoMapper 設定檔配置
   - 跨領域關注點的中介軟體實作
   - 擴充方法模式
   - 配置的選項模式實作
   - 使用 ILogger 的記錄實作
   - 驗證/授權篩選器或原則實作" : ""}
```

**Spring 實作模式（如果偵測到）：**
```
${PROJECT_TYPE == "Java" || PROJECT_TYPE == "Spring" || PROJECT_TYPE == "Auto-detect" ?
  "- 具有註解和相依性注入的完整控制器類別
   - 具有交易邊界的服務實作
   - 儲存庫介面和實作
   - 具有關聯的 JPA 實體定義
   - DTO 類別實作
   - Bean 配置和元件掃描
   - 例外處理程序實作
   - 自訂驗證器實作" : ""}
```

**React 實作模式（如果偵測到）：**
```
${PROJECT_TYPE == "React" || PROJECT_TYPE == "Auto-detect" ?
  "- 具有 props 和 state 的元件結構
   - Hook 實作模式（useState、useEffect、自訂 hooks）
   - API 服務實作
   - 狀態管理模式（Context、Redux）
   - 表單處理實作
   - 路由配置" : ""}
```

### 實作指南

根據記錄的工作流程，提供實作新功能的具體指導：

#### 1. 逐步實作流程
- 新增類似功能時從哪裡開始
- 實作順序（例如，模型 → 儲存庫 → 服務 → 控制器）
- 如何與現有的跨領域關注點整合

#### 2. 要避免的常見陷阱
- 識別目前實作中容易出錯的區域
- 注意效能考量
- 列出遇到的常見錯誤或問題

#### 3. 擴充機制
- 記錄如何插入現有的擴充點
- 顯示如何在不修改現有程式碼的情況下新增新行為
- 解釋配置驅動的功能模式

**結論：**
以最重要模式的摘要作為結論，在實作新功能時應遵循這些模式，以保持與程式碼庫的一致性。"
