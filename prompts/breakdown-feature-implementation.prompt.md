---
mode: 'agent'
description: '用於創建詳細功能實作計劃的提示，遵循 Epoch monorepo 結構。'
---

# 功能實作計劃提示

## 目標

擔任為大型 SaaS 公司打造高品質功能的資深軟體工程師。擅長根據功能 PRD 創建詳細的技術實作計劃。
審查提供的上下文並輸出全面、綜合的實作計劃。
**注意：** 除非是用於技術情況的偽代碼，否則不要在輸出中撰寫程式碼。

## 輸出格式

輸出應該是一個完整的實作計劃，以 Markdown 格式保存到 `/docs/ways-of-work/plan/{epic-name}/{feature-name}/implementation-plan.md`。

### 檔案系統

遵循 Epoch monorepo 結構的前端和後端儲存庫的資料夾和檔案結構：

```
apps/
  [app-name]/
services/
  [service-name]/
packages/
  [package-name]/
```

### 實作計劃

對於每個功能：

#### 目標

描述功能目標（3-5 句）

#### 需求

- 詳細的功能需求（項目符號列表）
- 實作計劃細節

#### 技術考量

##### 系統架構概述

使用 Mermaid 創建一個全面的系統架構圖，展示此功能如何整合到整體系統中。圖表應包括：

- **前端層**：使用者介面元件、狀態管理和客戶端邏輯
- **API 層**：tRPC 端點、驗證中介軟體、輸入驗證和請求路由
- **業務邏輯層**：服務類別、業務規則、工作流程編排和事件處理
- **資料層**：資料庫互動、快取機制和外部 API 整合
- **基礎設施層**：Docker 容器、背景服務和部署元件

使用子圖清楚地組織這些層次。使用標記箭頭顯示層次之間的資料流，指示請求/回應模式、資料轉換和事件流。包括此實作獨有的任何功能特定元件、服務或資料結構。

- **技術堆疊選擇**：記錄每層的選擇理由
```

- **技術堆疊選擇**：記錄每層的選擇理由
- **整合點**：定義清晰的邊界和通訊協定
- **部署架構**：Docker 容器化策略
- **可擴展性考量**：水平和垂直擴展方法

##### 資料庫架構設計

使用 Mermaid 創建實體關係圖，展示功能的資料模型：

- **資料表規格**：詳細的欄位定義，包含類型和約束
- **索引策略**：效能關鍵索引及其理由
- **外鍵關係**：資料完整性和參照約束
- **資料庫遷移策略**：版本控制和部署方法

##### API 設計

- 具有完整規格的端點
- 請求/回應格式與 TypeScript 類型
- 使用 Stack Auth 的驗證和授權
- 錯誤處理策略和狀態碼
- 速率限制和快取策略

##### 前端架構

###### 元件階層文件

元件結構將利用 `shadcn/ui` 函式庫作為一致且易於存取的基礎。

**版面結構：**

```
食譜庫頁面
├── 標題區段（shadcn: Card）
│   ├── 標題（shadcn: Typography `h1`）
│   ├── 新增食譜按鈕（shadcn: Button with DropdownMenu）
│   │   ├── 手動輸入（DropdownMenuItem）
│   │   ├── 從 URL 匯入（DropdownMenuItem）
│   │   └── 從 PDF 匯入（DropdownMenuItem）
│   └── 搜尋輸入（shadcn: Input with icon）
├── 主要內容區域（flex 容器）
│   ├── 篩選側邊欄（aside）
│   │   ├── 篩選標題（shadcn: Typography `h4`）
│   │   ├── 類別篩選（shadcn: Checkbox group）
│   │   ├── 料理篩選（shadcn: Checkbox group）
│   │   └── 難度篩選（shadcn: RadioGroup）
│   └── 食譜網格（main）
│       └── 食譜卡片（shadcn: Card）
│           ├── 食譜圖片（img）
│           ├── 食譜標題（shadcn: Typography `h3`）
│           ├── 食譜標籤（shadcn: Badge）
│           └── 快速操作（shadcn: Button - 檢視、編輯）
```

- **狀態流圖**：使用 Mermaid 的元件狀態管理
- 可重用元件庫規格
- 使用 Zustand/React Query 的狀態管理模式
- TypeScript 介面和類型

##### 安全效能

- 驗證/授權需求
- 資料驗證和清理
- 效能優化策略
- 快取機制

## 上下文模板

- **功能 PRD：** [功能 PRD markdown 檔案的內容]
