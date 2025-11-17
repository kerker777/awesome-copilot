---
description: 'Comprehensive technology stack blueprint generator that analyzes codebases to create detailed architectural documentation. Automatically detects technology stacks, programming languages, and implementation patterns across multiple platforms (.NET, Java, JavaScript, React, Python). Generates configurable blueprints with version information, licensing details, usage patterns, coding conventions, and visual diagrams. Provides implementation-ready templates and maintains architectural consistency for guided development.'
mode: 'agent'
---

# 全面的技術堆疊藍圖產生器

## 設定變數
${PROJECT_TYPE="Auto-detect|.NET|Java|JavaScript|React.js|React Native|Angular|Python|Other"} <!-- 主要技術 -->
${DEPTH_LEVEL="Basic|Standard|Comprehensive|Implementation-Ready"} <!-- 分析深度 -->
${INCLUDE_VERSIONS=true|false} <!-- 包含版本資訊 -->
${INCLUDE_LICENSES=true|false} <!-- 包含授權資訊 -->
${INCLUDE_DIAGRAMS=true|false} <!-- 產生架構圖 -->
${INCLUDE_USAGE_PATTERNS=true|false} <!-- 包含程式碼使用案例 -->
${INCLUDE_CONVENTIONS=true|false} <!-- 記錄編碼慣例 -->
${OUTPUT_FORMAT="Markdown|JSON|YAML|HTML"} <!-- 選擇輸出格式 -->
${CATEGORIZATION="Technology Type|Layer|Purpose"} <!-- 組織方法 -->

## 產生的提示詞

「分析代碼庫，並產生 ${DEPTH_LEVEL} 技術堆疊藍圖，全面記錄技術與實作模式，以便於一致的程式碼產生。請按照以下方式進行：

### 1. 技術識別階段
- ${PROJECT_TYPE == "Auto-detect" ? "掃描代碼庫的專案檔案、設定檔與相依性，以判斷所使用的所有技術堆疊" : "專注於 ${PROJECT_TYPE} 技術"}
- 透過檢查副檔名與檔案內容來識別所有程式設計語言
- 分析設定檔（package.json、.csproj、pom.xml 等）以提取相依性
- 檢查建置指令稿與管道定義，以取得工具資訊
- ${INCLUDE_VERSIONS ? "從套件檔案與設定中提取精確的版本資訊" : "略過版本詳細資訊"}
- ${INCLUDE_LICENSES ? "記錄所有相依性的授權資訊" : ""}

### 2. 核心技術分析

${PROJECT_TYPE == ".NET" || PROJECT_TYPE == "Auto-detect" ? "#### .NET 堆疊分析（若偵測到）
- 目標架構與語言版本（從專案檔案中偵測）
- 所有 NuGet 套件參考、版本與目的註解
- 專案結構與組織模式
- 設定方式（appsettings.json、IOptions 等）
- 認證機制（Identity、JWT 等）
- API 設計模式（REST、GraphQL、最小化 API 等）
- 資料存取方式（EF Core、Dapper 等）
- 相依性注入模式
- 中介軟體管道元件" : ""}

${PROJECT_TYPE == "Java" || PROJECT_TYPE == "Auto-detect" ? "#### Java 堆疊分析（若偵測到）
- JDK 版本與核心架構
- 所有 Maven/Gradle 相依性、版本與目的
- 套件結構組織
- Spring Boot 使用與設定
- 註解模式
- 相依性注入方式
- 資料存取技術（JPA、JDBC 等）
- API 設計（Spring MVC、JAX-RS 等）" : ""}

${PROJECT_TYPE == "JavaScript" || PROJECT_TYPE == "Auto-detect" ? "#### JavaScript 堆疊分析（若偵測到）
- ECMAScript 版本與轉譯器設定
- 所有 npm 相依性，按目的分類
- 模組系統（ESM、CommonJS）
- 建置工具（webpack、Vite 等）及其設定
- TypeScript 使用與設定
- 測試架構與模式" : ""}

${PROJECT_TYPE == "React.js" || PROJECT_TYPE == "Auto-detect" ? "#### React 分析（若偵測到）
- React 版本與關鍵模式（hooks vs class components）
- 狀態管理方式（Context、Redux、Zustand 等）
- 元件庫使用（Material-UI、Chakra 等）
- 路由實作
- 表單處理策略
- API 整合模式
- 元件測試方式" : ""}

${PROJECT_TYPE == "Python" || PROJECT_TYPE == "Auto-detect" ? "#### Python 分析（若偵測到）
- Python 版本與使用的關鍵語言特性
- 套件相依性與虛擬環境設定
- Web 架構詳細資訊（Django、Flask、FastAPI）
- ORM 使用模式
- 專案結構組織
- API 設計模式" : ""}

### 3. 實作模式與慣例
${INCLUDE_CONVENTIONS ?
"記錄每個技術領域的編碼慣例與模式：

#### 命名慣例
- 類別/類型命名模式
- 方法/函式命名模式
- 變數命名慣例
- 檔案命名與組織慣例
- 介面/抽象類別模式

#### 程式碼組織
- 檔案結構與組織
- 資料夾階層結構模式
- 元件/模組邊界
- 程式碼分離與責任模式

#### 常見模式
- 錯誤處理方式
- 日誌記錄模式
- 設定存取
- 認證/授權實作
- 驗證策略
- 測試模式" : ""}

### 4. 使用案例
${INCLUDE_USAGE_PATTERNS ?
"提取代表性程式碼範例，展示標準實作模式：

#### API 實作範例
- 標準控制器/端點實作
- 請求 DTO 模式
- 回應格式化
- 驗證方式
- 錯誤處理

#### 資料存取範例
- Repository 模式實作
- 實體/模型定義
- 查詢模式
- 交易處理

#### 服務層範例
- 服務類別實作
- 商業邏輯組織
- 跨域關注點整合
- 相依性注入使用

#### UI 元件範例（如適用）
- 元件結構
- 狀態管理模式
- 事件處理
- API 整合模式" : ""}

### 5. 技術堆疊地圖
${DEPTH_LEVEL == "Comprehensive" || DEPTH_LEVEL == "Implementation-Ready" ?
"建立全面的技術地圖，包括：

#### 核心架構使用
- 專案中的主要架構及其具體使用方式
- 架構特定的設定與自訂
- 擴充點與自訂項目

#### 整合點
- 不同技術元件的整合方式
- 元件間的認證流程
- 前端與後端的資料流
- 第三方服務整合模式

#### 開發工具
- IDE 設定與慣例
- 程式碼分析工具
- Linters 與格式化工具及其設定
- 建置與部署管道
- 測試架構與方式

#### 基礎設施
- 部署環境詳細資訊
- 容器技術
- 所使用的雲端服務
- 監控與日誌基礎設施" : ""}

### 6. 技術特定的實作詳細資訊

${PROJECT_TYPE == ".NET" || PROJECT_TYPE == "Auto-detect" ?
"#### .NET 實作詳細資訊（若偵測到）
- **相依性注入模式**：
  - 服務註冊方式（Scoped/Singleton/Transient 模式）
  - 設定綁定模式

- **控制器模式**：
  - 基礎控制器使用
  - Action 結果類型與模式
  - 路由屬性慣例
  - 篩選器使用（授權、驗證等）

- **資料存取模式**：
  - ORM 設定與使用
  - 實體設定方式
  - 關聯性定義
  - 查詢模式與最佳化方式

- **API 設計模式**（如使用）：
  - 端點組織
  - 參數綁定方式
  - 回應類型處理

- **語言特性使用**：
  - 從程式碼偵測特定語言特性
  - 識別常見模式與慣用法
  - 注意任何特定版本相依的特性" : ""}

${PROJECT_TYPE == "React.js" || PROJECT_TYPE == "Auto-detect" ?
"#### React 實作詳細資訊（若偵測到）
- **元件結構**：
  - 函式式 vs class 元件
  - Props 介面定義
  - 元件組合模式

- **Hook 使用模式**：
  - 自訂 Hook 實作風格
  - useState 模式
  - useEffect 清理方式
  - Context 使用模式

- **狀態管理**：
  - 本地 vs 全域狀態決策
  - 狀態管理庫模式
  - Store 設定
  - 選擇器模式

- **樣式方式**：
  - CSS 方法論（CSS modules、styled-components 等）
  - 佈景主題實作
  - 響應式設計模式" : ""}

### 7. 新程式碼實作藍圖
${DEPTH_LEVEL == "Implementation-Ready" ?
"根據分析提供詳細的新功能實作藍圖：

- **檔案/類別範本**：常見元件類型的標準結構
- **程式碼片段**：常見操作的可立即使用程式碼模式
- **實作檢查清單**：端對端實作功能的標準步驟
- **整合點**：如何將新程式碼與現有系統連接
- **測試需求**：不同元件類型的標準測試模式
- **文件需求**：新功能的標準文件模式" : ""}

${INCLUDE_DIAGRAMS ?
"### 8. 技術關係圖
- **堆疊圖**：完整技術堆疊的視覺化表示
- **相依性流**：不同技術的互動方式
- **元件關係**：主要元件的相依性
- **資料流**：資料流經技術堆疊的方式" : ""}

### ${INCLUDE_DIAGRAMS ? "9" : "8"}. 技術決策背景
- 記錄技術選擇的明顯原因
- 注意標記為取代的舊版或已棄用技術
- 識別技術限制與邊界
- 記錄技術升級路徑與相容性考量

將輸出格式化為 ${OUTPUT_FORMAT}，並按 ${CATEGORIZATION} 分類技術。

將輸出保存為 'Technology_Stack_Blueprint.${OUTPUT_FORMAT == "Markdown" ? "md" : OUTPUT_FORMAT.toLowerCase()}'
」