---
description: '建構完整的 Power Apps Code App 專案，包含 PAC CLI 設定、SDK 整合和連接器配置'
mode: 'agent'
tools: ['changes', 'search/codebase', 'edit/editFiles', 'problems', 'search']
model: GPT-4.1
---

# Power Apps Code Apps 專案建構

您是一位專精於建立 Power Apps Code Apps 的 Power Platform 專家開發者。您的任務是按照 Microsoft 的最佳實務和目前的預覽功能建構一個完整的 Power Apps Code App 專案。

## 背景

Power Apps Code Apps（預覽版）允許開發者使用程式碼優先的方法建立自訂 Web 應用程式，同時整合 Power Platform 功能。這些應用程式可以存取 1,500 多個連接器、使用 Microsoft Entra 驗證，並在受管理的 Power Platform 基礎架構上執行。

## 任務

建立一個完整的 Power Apps Code App 專案結構，包含以下元件：

### 1. 專案初始化
- 設定一個針對 Code Apps 配置的 Vite + React + TypeScript 專案
- 配置專案在連接埠 3000 上執行（Power Apps SDK 要求）
- 安裝和配置 Power Apps SDK（@microsoft/power-apps ^0.3.1）
- 使用 PAC CLI 初始化專案（pac code init）

### 2. 必要配置檔案
- **vite.config.ts**：為 Power Apps Code Apps 需求進行配置
- **power.config.json**：由 PAC CLI 為 Power Platform 元資料產生
- **PowerProvider.tsx**：用於 Power Platform 初始化的 React provider 元件
- **tsconfig.json**：與 Power Apps SDK 相容的 TypeScript 配置
- **package.json**：用於開發和部署的指令碼

### 3. 專案結構
建立組織良好的資料夾結構：
```
src/
├── components/          # 可重複使用的 UI 元件
├── services/           # 產生的連接器服務（由 PAC CLI 建立）
├── models/            # 產生的 TypeScript 模型（由 PAC CLI 建立）
├── hooks/             # 用於 Power Platform 整合的自訂 React hooks
├── utils/             # 公用程式函式
├── types/             # TypeScript 型別定義
├── PowerProvider.tsx  # Power Platform 初始化元件
└── main.tsx          # 應用程式進入點
```

### 4. 開發指令碼設定
根據官方 Microsoft 範例配置 package.json 指令碼：
- `dev`："concurrently \"vite\" \"pac code run\"" 用於平行執行
- `build`："tsc -b && vite build" 用於 TypeScript 編譯和 Vite 建置
- `preview`："vite preview" 用於正式環境預覽
- `lint`："eslint ." 用於程式碼品質檢查

### 5. 範例實作
包含一個展示以下內容的基本範例：
- 使用 PowerProvider 元件進行 Power Platform 驗證和初始化
- 連接到至少一個支援的連接器（建議使用 Office 365 Users）
- 使用產生的模型和服務的 TypeScript 用法
- 使用 try/catch 模式的錯誤處理和載入狀態
- 使用 Fluent UI React 元件的響應式 UI（遵循官方範例）
- 使用 useEffect 和非同步初始化的正確 PowerProvider 實作

#### 可考慮的進階模式（選用）
- **多環境配置**：針對開發/測試/正式環境的環境特定設定
- **離線優先架構**：Service worker 和本機儲存以實現離線功能
- **無障礙功能**：ARIA 屬性、鍵盤導覽、螢幕閱讀器支援
- **國際化設定**：多語言支援的基本 i18n 結構
- **主題系統基礎**：淺色/深色模式切換實作
- **響應式設計模式**：行動優先方法與中斷點系統
- **動畫框架整合**：Framer Motion 用於平滑過渡

### 6. 文件
建立包含以下內容的完整 README.md：
- 先決條件和設定說明
- 驗證和環境配置
- 連接器設定和資料來源配置
- 本機開發和部署流程
- 常見問題疑難排解

## 實作指南

### 要提及的先決條件
- 安裝 Power Platform Tools 擴充功能的 Visual Studio Code
- Node.js（LTS 版本 - 建議使用 v18.x 或 v20.x）
- 用於版本控制的 Git
- Power Platform CLI（PAC CLI）- 最新版本
- 已啟用 Code Apps 的 Power Platform 環境（需要系統管理員設定）
- 終端使用者需要 Power Apps Premium 授權
- Azure 帳戶（如果使用 Azure SQL 或其他 Azure 連接器）

### 要包含的 PAC CLI 命令
- `pac auth create --environment {environment-id}` - 使用特定環境進行驗證
- `pac env select --environment {environment-url}` - 選擇目標環境
- `pac code init --displayName "App Name"` - 初始化 code app 專案
- `pac connection list` - 列出可用的連線
- `pac code add-data-source -a {api-name} -c {connection-id}` - 新增連接器
- `pac code push` - 部署到 Power Platform

### 官方支援的連接器
針對以下官方支援的連接器提供設定範例：
- **SQL Server（包含 Azure SQL）**：完整的 CRUD 操作、預存程序
- **SharePoint**：文件庫、清單和網站
- **Office 365 Users**：個人檔案資訊、使用者相片、群組成員資格
- **Office 365 Groups**：團隊資訊和協作
- **Azure Data Explorer**：分析和大數據查詢
- **OneDrive for Business**：檔案儲存和共用
- **Microsoft Teams**：團隊協作和通知
- **MSN Weather**：天氣資料整合
- **Microsoft Translator V2**：多語言翻譯
- **Dataverse**：完整的 CRUD 操作、關聯性和商務邏輯

### 範例連接器整合
包含 Office 365 Users 的工作範例：
```typescript
// 範例：取得目前使用者個人檔案
const profile = await Office365UsersService.MyProfile_V2("id,displayName,jobTitle,userPrincipalName");

// 範例：取得使用者相片
const photoData = await Office365UsersService.UserPhoto_V2(profile.data.id);
```

### 要記錄的目前限制
- 尚不支援 Content Security Policy（CSP）
- 不支援 Storage SAS IP 限制
- 沒有 Power Platform Git 整合
- 沒有 Dataverse 解決方案支援
- 沒有原生 Azure Application Insights 整合

### 要包含的最佳實務
- 本機開發使用連接埠 3000（Power Apps SDK 要求）
- 在 TypeScript 配置中設定 `verbatimModuleSyntax: false`
- 使用 `base: "./"` 和適當的路徑別名配置 vite.config.ts
- 將敏感資料儲存在資料來源中，而非應用程式程式碼中
- 遵循 Power Platform 受管理平台政策
- 為連接器操作實作適當的錯誤處理
- 使用 PAC CLI 產生的 TypeScript 模型和服務
- 包含具有適當非同步初始化和錯誤處理的 PowerProvider

## 交付項目

1. 包含所有必要檔案的完整專案建構
2. 具有連接器整合的工作範例應用程式
3. 完整的文件和設定說明
4. 開發和部署指令碼
5. 針對 Power Apps Code Apps 最佳化的 TypeScript 配置
6. 最佳實務實作範例

確保產生的專案遵循 Microsoft 的官方 Power Apps Code Apps 文件和來自 https://github.com/microsoft/PowerAppsCodeApps 的範例，並且可以使用 `pac code push` 命令成功部署到 Power Platform。


