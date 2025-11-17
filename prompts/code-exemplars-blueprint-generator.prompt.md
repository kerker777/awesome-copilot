---
description: 'Technology-agnostic prompt generator that creates customizable AI prompts for scanning codebases and identifying high-quality code exemplars. Supports multiple programming languages (.NET, Java, JavaScript, TypeScript, React, Angular, Python) with configurable analysis depth, categorization methods, and documentation formats to establish coding standards and maintain consistency across development teams.'
mode: 'agent'
---

# 程式碼範例藍圖產生器

## 設定變數
${PROJECT_TYPE="Auto-detect|.NET|Java|JavaScript|TypeScript|React|Angular|Python|Other"} <!-- 主要技術 -->
${SCAN_DEPTH="Basic|Standard|Comprehensive"} <!-- 分析程式碼庫的深度 -->
${INCLUDE_CODE_SNIPPETS=true|false} <!-- 除了檔案參考外，是否要包含實際程式碼片段 -->
${CATEGORIZATION="Pattern Type|Architecture Layer|File Type"} <!-- 如何組織範例 -->
${MAX_EXAMPLES_PER_CATEGORY=3} <!-- 每個類別的最大範例數 -->
${INCLUDE_COMMENTS=true|false} <!-- 是否在每個範例旁包含說明註解 -->

## 生成的提示詞

「掃描這個程式碼庫，並生成一個 exemplars.md 檔案，識別出高品質且代表性的程式碼範例。這些範例應展示我們的編碼標準與模式，以幫助維持一致性。使用下列方法：

### 1. 程式碼庫分析階段
- ${PROJECT_TYPE == "Auto-detect" ? "通過掃描檔案副檔名與設定檔案，自動偵測主要的程式語言與框架" : `專注於 ${PROJECT_TYPE} 程式碼檔案`}
- 識別實現品質良好、文件完整且結構清晰的檔案
- 尋找常見的模式、架構組件與結構良好的實現
- 優先考慮展示我們技術堆棧最佳實踐的檔案
- 只參考程式碼庫中實際存在的檔案 - 不包含假想的範例

### 2. 範例識別標準
- 結構良好、易讀的程式碼，具有清晰的命名慣例
- 完整的註解與文件
- 適當的錯誤處理與驗證
- 遵循設計模式與架構原則
- 關注點分離與單一職責原則
- 高效的實現，沒有程式碼異味
- 代表我們的標準做法

### 3. 核心模式類別

${PROJECT_TYPE == ".NET" || PROJECT_TYPE == "Auto-detect" ? `#### .NET 範例（如果偵測到）
- **Domain Models**：找到適當實現封裝與領域邏輯的實體
- **Repository 實現**：我們資料存取方法的範例
- **Service Layer 組件**：結構良好的業務邏輯實現
- **Controller 模式**：具有適當驗證與回應的簡潔 API 控制器
- **Dependency Injection 使用**：DI 設定與使用的良好範例
- **Middleware 組件**：自訂 middleware 實現
- **Unit Test 模式**：具有適當安排與斷言的結構良好測試` : ""}

${(PROJECT_TYPE == "JavaScript" || PROJECT_TYPE == "TypeScript" || PROJECT_TYPE == "React" || PROJECT_TYPE == "Angular" || PROJECT_TYPE == "Auto-detect") ? `#### 前端範例（如果偵測到）
- **Component 結構**：簡潔且結構良好的元件
- **狀態管理**：狀態處理的良好範例
- **API 整合**：實裝良好的服務呼叫與資料處理
- **表單處理**：驗證與提交模式
- **路由實現**：導航與路由設定
- **UI 元件**：可重複使用且結構良好的 UI 元素
- **Unit Test 範例**：元件與服務測試` : ""}

${PROJECT_TYPE == "Java" || PROJECT_TYPE == "Auto-detect" ? `#### Java 範例（如果偵測到）
- **Entity 類別**：設計良好的 JPA 實體或領域模型
- **Service 實現**：簡潔的服務層組件
- **Repository 模式**：資料存取實現
- **Controller/Resource 類別**：API 端點實現
- **Configuration 類別**：應用程式設定
- **Unit Tests**：結構良好的 JUnit 測試` : ""}

${PROJECT_TYPE == "Python" || PROJECT_TYPE == "Auto-detect" ? `#### Python 範例（如果偵測到）
- **Class 定義**：結構良好且具有適當文件的類別
- **API Routes/Views**：簡潔的 API 實現
- **Data Models**：ORM 模型定義
- **Service Functions**：業務邏輯實現
- **Utility Modules**：輔助與公用程式函式
- **Test Cases**：結構良好的單元測試` : ""}

### 4. 架構層級範例

- **Presentation Layer**：
  - 使用者介面元件
  - Controllers/API 端點
  - View models/DTOs

- **Business Logic Layer**：
  - Service 實現
  - 業務邏輯元件
  - 工作流程協調

- **Data Access Layer**：
  - Repository 實現
  - 資料模型
  - 查詢模式

- **Cross-Cutting Concerns**：
  - 日誌實現
  - 錯誤處理
  - 認證/授權
  - 驗證

### 5. 範例文件格式

對於每個識別出的範例，文件應包含：
- 檔案路徑（相對於倉庫根目錄）
- 簡短的說明，說明它為何是範例
- 它代表的模式或元件類型
${INCLUDE_COMMENTS ? "- 展示的關鍵實現細節與編碼原則" : ""}
${INCLUDE_CODE_SNIPPETS ? "- 小型的代表性程式碼片段（如適用）" : ""}

${SCAN_DEPTH == "Comprehensive" ? `### 6. 額外文件

- **一致性模式**：注意在程式碼庫中觀察到的一致性模式
- **架構觀察**：記錄程式碼中顯見的架構模式
- **實現慣例**：識別命名與結構慣例
- **要避免的反模式**：注意程式碼庫偏離最佳實踐的任何地方` : ""}

### ${SCAN_DEPTH == "Comprehensive" ? "7" : "6"}. 輸出格式

建立 exemplars.md，包含：
1. 說明文件目的的介紹
2. 包含連結到各類別的目錄
3. 基於 ${CATEGORIZATION} 的組織章節
4. 每個類別最多 ${MAX_EXAMPLES_PER_CATEGORY} 個範例
5. 包含維持程式碼品質建議的結論

文件應適合開發人員在實現與現有模式一致的新功能時參考。

重要：只包含程式碼庫中的實際檔案。驗證所有檔案路徑存在。不包含預留位置或假想範例。
」

## 預期輸出
執行此提示詞後，GitHub Copilot 將掃描您的程式碼庫並生成一個 exemplars.md 檔案，包含您倉庫中高品質程式碼範例的真實參考，並根據您選擇的參數進行組織。
