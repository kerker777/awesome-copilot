---
description: 'Power Apps Code Apps 開發標準與最佳實踐 - 涵蓋 TypeScript、React 及 Power Platform 整合'
applyTo: '**/*.{ts,tsx,js,jsx}, **/vite.config.*, **/package.json, **/tsconfig.json, **/power.config.json'
---

# Power Apps Code Apps 開發指引

以 TypeScript、React 和 Power Platform SDK 產生高品質 Power Apps Code Apps 的指引，遵循 Microsoft 官方最佳實踐和預覽功能。

## 專案背景

- **Power Apps Code Apps (預覽版)**: 以程式碼優先的方式開發網頁應用程式，與 Power Platform 整合
- **TypeScript + React**: 推薦的前端堆棧，使用 Vite 打包工具
- **Power Platform SDK**: @microsoft/power-apps (目前版本 ^0.3.1) 用於連接器整合
- **PAC CLI**: Power Platform 命令列工具，用於專案管理和部署
- **連接埠 3000**: 本機開發時必須使用 Power Platform SDK 所需的連接埠
- **Power Apps 進階版**: 生產環境使用所需的使用者授權要求

## 開發標準

### 專案結構

- 使用組織良好的資料夾結構，清楚區分不同職責：
  ```
  src/
  ├── components/          # 可重複使用的 UI 元件
  ├── hooks/              # Power Platform 自訂 React hooks
  ├── services/           # 產生的連接器服務 (PAC CLI)
  ├── models/            # 產生的 TypeScript 模型 (PAC CLI)
  ├── utils/             # 公用函數與輔助工具
  ├── types/             # TypeScript 類型定義
  ├── PowerProvider.tsx  # Power Platform 初始化
  └── main.tsx          # 應用程式進入點
  ```
- 將產生的檔案 (`services/`、`models/`) 與自訂程式碼分開
- 使用一致的命名慣例（檔案使用 kebab-case，元件使用 PascalCase）

### TypeScript 設定

- 在 tsconfig.json 中設定 `verbatimModuleSyntax: false` 以相容 Power Apps SDK
- 啟用嚴格模式以確保型別安全，建議的 tsconfig.json 設定如下：
  ```json
  {
    "compilerOptions": {
      "target": "ES2020",
      "useDefineForClassFields": true,
      "lib": ["ES2020", "DOM", "DOM.Iterable"],
      "module": "ESNext",
      "skipLibCheck": true,
      "verbatimModuleSyntax": false,
      "moduleResolution": "bundler",
      "allowImportingTsExtensions": true,
      "resolveJsonModule": true,
      "isolatedModules": true,
      "noEmit": true,
      "jsx": "react-jsx",
      "strict": true,
      "noUnusedLocals": true,
      "noUnusedParameters": true,
      "noFallthroughCasesInSwitch": true,
      "baseUrl": ".",
      "paths": {
        "@/*": ["./src/*"]
      }
    }
  }
  ```
- 為 Power Platform 連接器回應使用適當的型別
- 透過 `"@": path.resolve(__dirname, "./src")` 設定路徑別名，以實現更簡潔的匯入
- 定義應用程式特定資料結構的介面
- 實作錯誤邊界並使用適當的錯誤處理型別

### 進階 Power Platform 整合

#### 自訂控制項架構 (PCF 控制項)
- **整合 PCF 控制項**：在 Code Apps 中嵌入 Power Apps 元件架構控制項
  ```typescript
  // 範例：使用自訂 PCF 控制項進行資料視覺化
  import { PCFControlWrapper } from './components/PCFControlWrapper';
  
  const MyComponent = () => {
    return (
      <PCFControlWrapper
        controlName="CustomChartControl"
        dataset={chartData}
        configuration={chartConfig}
      />
    );
  };
  ```
- **PCF 控制項通訊**：處理 PCF 與 React 之間的事件和資料繫結
- **自訂控制項部署**：與 Code Apps 一起封裝與部署 PCF 控制項

#### Power BI 內嵌分析
- **內嵌 Power BI 報表**：整合互動式儀表板與報表
  ```typescript
  import { PowerBIEmbed } from 'powerbi-client-react';
  
  const DashboardComponent = () => {
    return (
      <PowerBIEmbed
        embedConfig={{
          type: 'report',
          id: reportId,
          embedUrl: embedUrl,
          accessToken: accessToken,
          tokenType: models.TokenType.Aad,
          settings: {
            panes: { filters: { expanded: false, visible: false } }
          }
        }}
      />
    );
  };
  ```
- **動態報表篩選**：根據 Code App 內容篩選 Power BI 報表
- **報表匯出功能**：啟用 PDF、Excel 和影像匯出

#### AI Builder 整合
- **認知服務整合**：使用 AI Builder 模型進行表單處理、物件偵測
  ```typescript
  // 範例：使用 AI Builder 進行文件處理
  const processDocument = async (file: File) => {
    const formData = new FormData();
    formData.append('file', file);
    
    const result = await AIBuilderService.ProcessDocument({
      modelId: 'document-processing-model-id',
      document: formData
    });
    
    return result.extractedFields;
  };
  ```
- **預測模型**：整合自訂 AI 模型以進行業務預測
- **情感分析**：使用 AI Builder 分析文本情感
- **物件偵測**：實作影像分析與物件識別

#### Power Virtual Agents 整合
- **聊天機器人嵌入**：在 Code Apps 中整合 Power Virtual Agents Bot
  ```typescript
  import { DirectLine } from 'botframework-directlinejs';
  import { WebChat } from 'botframework-webchat';

  const ChatbotComponent = () => {
    const directLine = new DirectLine({
      token: chatbotToken
    });
    
    return (
      <div style={{ height: '400px', width: '100%' }}>
        <WebChat directLine={directLine} />
      </div>
    );
  };
  ```
- **內容傳遞**：與聊天機器人對話共享 Code App 內容
- **自訂 Bot 操作**：從 Bot 互動觸發 Code App 函數
- 使用 PAC CLI 產生的 TypeScript 服務進行連接器操作
- 使用 Microsoft Entra ID 實作適當的身驗流程
- 處理連接器同意對話框與權限管理
- PowerProvider 實作模式：
  ```typescript
  import { initialize } from "@microsoft/power-apps/app";
  import { useEffect, type ReactNode } from "react";

  export default function PowerProvider({ children }: { children: ReactNode }) {
    useEffect(() => {
      const initApp = async () => {
        try {
          await initialize();
          console.log('Power Platform SDK initialized successfully');
        } catch (error) {
          console.error('Failed to initialize Power Platform SDK:', error);
        }
      };
      initApp();
    }, []);
    return <>{children}</>;
  }
  ```
- 遵循官方支援的連接器模式：
  - SQL Server (including Azure SQL)
  - SharePoint
  - Office 365 Users/Groups
  - Azure Data Explorer
  - OneDrive for Business
  - Microsoft Teams
  - Dataverse (CRUD 操作)

### React 模式

- 所有新開發均使用函式元件搭配 hooks
- 為連接器操作實作適當的載入與錯誤狀態
- 考慮使用 Fluent UI React 元件（如官方範例中所用）
- 在適當時使用 React Query 或 SWR 進行資料獲取與快取
- 遵循 React 最佳實踐進行元件組成
- 使用行動優先的方式實作響應式設計
- 按照官方範例安裝主要相依套件：
  - `@microsoft/power-apps` 用於 Power Platform SDK
  - `@fluentui/react-components` 用於 UI 元件
  - `concurrently` 用於並行執行腳本 (開發相依套件)

### 資料管理

- 將敏感資料存放在資料來源中，絕不在應用程式代碼中儲存
- 使用產生的模型進行型別安全的連接器操作
- 實作適當的資料驗證與淨化
- 盡可能優雅地處理離線情境
- 適當快取經常存取的資料

#### 進階 Dataverse 關係
- **多對多關係**：實作聯接表與關係服務
  ```typescript
  // 範例：使用者與角色的多對多關係
  const userRoles = await UserRoleService.getall();
  const filteredRoles = userRoles.filter(ur => ur.userId === currentUser.id);
  ```
- **多型查閱**：處理可參考多個實體型別的客戶欄位
  ```typescript
  // 處理多型客戶查閱 (帳戶或聯絡人)
  const customerType = record.customerType; // 'account' 或 'contact'
  const customerId = record.customerId;
  const customer = customerType === 'account'
    ? await AccountService.get(customerId)
    : await ContactService.get(customerId);
  ```
- **複雜關係查詢**：使用 $expand 和 $filter 進行有效的資料檢索
- **關係驗證**：為關係限制實作業務規則

### 效能最佳化

- 對於耗時計算使用 React.memo 和 useMemo
- 為大型應用程式實作程式碼分割與延遲載入
- 使用樹搖最佳化束大小
- 使用有效的連接器查詢模式以最小化 API 呼叫
- 為大型資料集實作適當的分頁

#### 離線優先架構與同步模式
- **Service Worker 實作**：啟用離線功能
  ```typescript
  // 範例：Service Worker 註冊
  if ('serviceWorker' in navigator) {
    window.addEventListener('load', () => {
      navigator.serviceWorker.register('/sw.js')
        .then(registration => console.log('SW registered:', registration))
        .catch(error => console.log('SW registration failed:', error));
    });
  }
  ```
- **本機資料儲存**：使用 IndexedDB 進行離線資料持久化
  ```typescript
  // 範例：離線儲存的 IndexedDB 包裝
  class OfflineDataStore {
    async saveData(key: string, data: any) {
      const db = await this.openDB();
      const transaction = db.transaction(['data'], 'readwrite');
      transaction.objectStore('data').put({ id: key, data, timestamp: Date.now() });
    }
    
    async loadData(key: string) {
      const db = await this.openDB();
      const transaction = db.transaction(['data'], 'readonly');
      return transaction.objectStore('data').get(key);
    }
  }
  ```
- **同步衝突解決**：處理回到線上時的資料衝突
- **背景同步**：實作週期性資料同步
- **漸進式網頁應用程式 (PWA)**：啟用應用程式安裝與離線功能

### 安全最佳實踐

- 絕不在代碼中儲存秘密或敏感設定
- 使用 Power Platform 內建的身驗與授權
- 實作適當的輸入驗證與淨化
- 遵循 OWASP 網頁應用程式安全指南
- 尊重 Power Platform 資料遺失防止原則
- 實作僅限 HTTPS 的通訊

### 錯誤處理

- 在 React 中實作全面的錯誤邊界
- 優雅地處理連接器特定的錯誤
- 為使用者提供有意義的錯誤訊息
- 適當記錄錯誤，無須公開敏感資訊
- 為暫時性失敗實作重試邏輯
- 處理網路連線問題

### 測試策略

- 為業務邏輯與公用函數編寫單元測試
- 使用 React Testing Library 測試 React 元件
- 在測試中模擬 Power Platform 連接器
- 為關鍵使用者流程實作整合測試
- 使用 TypeScript 以確保更好的測試安全性
- 測試錯誤情境與邊界案例

### 開發流程

- 使用 PAC CLI 進行專案初始化與連接器管理
- 遵循適合團隊規模的 Git 分支策略
- 實作適當的程式碼檢閱流程
- 使用線性和格式化工具 (ESLint、Prettier)
- 使用 concurrently 配置開發腳本：
  - `"dev": "concurrently \"vite\" \"pac code run\""`
  - `"build": "tsc -b && vite build"`
- 在 CI/CD 管道中實作自動化測試
- 遵循發佈的語義化版本控制

### 部署與 DevOps

- 使用 `npm run build` 後接著 `pac code push` 進行部署
- 實作適當的環境管理 (開發、測試、生產)
- 使用環境特定的設定檔案
- 盡可能實作藍綠或金絲雀部署策略
- 監控生產環境中的應用程式效能與錯誤
- 實作適當的備份與災難復原程序

#### 多環境部署管道
- **環境特定的設定**：管理開發/測試/預備/生產環境
  ```json
  // 範例：環境特定的設定檔案
  // config/development.json
  {
    "powerPlatform": {
      "environmentUrl": "https://dev-env.crm.dynamics.com",
      "apiVersion": "9.2"
    },
    "features": {
      "enableDebugMode": true,
      "enableAnalytics": false
    }
  }
  ```
- **自動部署管道**：使用 Azure DevOps 或 GitHub Actions
  ```yaml
  # 範例 Azure DevOps 管道步驟
  - task: PowerPlatformToolInstaller@2
  - task: PowerPlatformSetConnectionVariables@2
    inputs:
      authenticationType: 'PowerPlatformSPN'
      applicationId: '$(AppId)'
      clientSecret: '$(ClientSecret)'
      tenantId: '$(TenantId)'
  - task: PowerPlatformPublishCustomizations@2
  ```
- **環境推廣**：自動推廣從開發 → 測試 → 預備 → 生產
- **復原策略**：在部署失敗時實作自動復原
- **設定管理**：使用 Azure Key Vault 管理環境特定的秘密

## 程式碼品質指南

### 元件開發

- 使用清楚的 props 介面建立可重複使用的元件
- 使用組成優於繼承
- 使用 TypeScript 實作適當的 props 驗證
- 遵循單一責任原則
- 使用清楚的命名編寫自我記錄的代碼

### 狀態管理

- 在簡單情境中使用 React 內建的狀態管理
- 對於複雜狀態管理考慮 Redux Toolkit
- 實作適當的狀態正規化
- 使用 context 或狀態管理庫避免 props drilling
- 有效使用衍生狀態與計算值

### API 整合

- 為了一致性使用 PAC CLI 產生的服務
- 實作適當的要求/回應攔截器
- 處理身驗令牌管理
- 實作要求去重複化與快取
- 使用適當的 HTTP 狀態碼處理

### 樣式與 UI

- 使用一致的設計系統或元件庫
- 使用 CSS Grid/Flexbox 實作響應式設計
- 遵循無障礙指南 (WCAG 2.1)
- 使用 CSS-in-JS 或 CSS 模組進行元件樣式設定
- 在適當時實作暗色模式支援
- 確保行動友善的使用者介面

#### 進階 UI/UX 模式

##### 使用元件庫的設計系統實作
- **元件庫結構**：建立可重複使用的元件系統
  ```typescript
  // 範例：設計系統按鈕元件
  interface ButtonProps {
    variant: 'primary' | 'secondary' | 'danger';
    size: 'small' | 'medium' | 'large';
    disabled?: boolean;
    onClick: () => void;
    children: React.ReactNode;
  }

  export const Button: React.FC<ButtonProps> = ({
    variant, size, disabled, onClick, children
  }) => {
    const classes = `btn btn-${variant} btn-${size} ${disabled ? 'btn-disabled' : ''}`;
    return <button className={classes} onClick={onClick} disabled={disabled}>{children}</button>;
  };
  ```
- **設計令牌**：實作一致的間距、顏色、字型
- **元件文件**：使用 Storybook 進行元件文件

##### 暗色模式與佈景系統
- **佈景提供者實作**：支援多個佈景
  ```typescript
  // 範例：佈景 context 與提供者
  const ThemeContext = createContext({
    theme: 'light',
    toggleTheme: () => {}
  });
  
  export const ThemeProvider: React.FC<{children: ReactNode}> = ({ children }) => {
    const [theme, setTheme] = useState<'light' | 'dark'>('light');
    
    const toggleTheme = () => {
      setTheme(prev => prev === 'light' ? 'dark' : 'light');
    };
    
    return (
      <ThemeContext.Provider value={{ theme, toggleTheme }}>
        <div className={`theme-${theme}`}>{children}</div>
      </ThemeContext.Provider>
    );
  };
  ```
- **CSS 自訂屬性**：使用 CSS 變數進行動態佈景設定
- **系統偏好偵測**：尊重使用者的作業系統佈景偏好

##### 響應式設計進階模式
- **容器查詢**：使用基於容器的響應式設計
  ```css
  /* 範例：響應式元件的容器查詢 */
  .card-container {
    container-type: inline-size;
  }
  
  @container (min-width: 400px) {
    .card {
      display: grid;
      grid-template-columns: 1fr 1fr;
    }
  }
  ```
- **流體排版**：實作響應式字型大小調整
- **適應性佈局**：根據螢幕大小和內容改變佈局模式

##### 動畫與微互動
- **Framer Motion 整合**：平順的動畫與過渡
  ```typescript
  import { motion, AnimatePresence } from 'framer-motion';

  const AnimatedCard = () => {
    return (
      <motion.div
        initial={{ opacity: 0, y: 20 }}
        animate={{ opacity: 1, y: 0 }}
        exit={{ opacity: 0, y: -20 }}
        transition={{ duration: 0.3 }}
        whileHover={{ scale: 1.02 }}
        className="card"
      >
        Card content
      </motion.div>
    );
  };
  ```
- **載入狀態**：動畫骨架與進度指示器
- **手勢識別**：滑動、縮放及觸碰互動
- **效能最佳化**：使用 CSS transforms 和 will-change 屬性

##### 無障礙自動化與測試
- **ARIA 實作**：適當的語義標記與 ARIA 屬性
  ```typescript
  // 範例：無障礙對話方塊元件
  const Modal: React.FC<{isOpen: boolean, onClose: () => void, children: ReactNode}> = ({ 
    isOpen, onClose, children 
  }) => {
    useEffect(() => {
      if (isOpen) {
        document.body.style.overflow = 'hidden';
        const focusableElement = document.querySelector('[data-autofocus]') as HTMLElement;
        focusableElement?.focus();
      }
      return () => { document.body.style.overflow = 'unset'; };
    }, [isOpen]);
    
    return (
      <div 
        role="dialog" 
        aria-modal="true" 
        aria-labelledby="modal-title"
        className={isOpen ? 'modal-open' : 'modal-hidden'}
      >
        {children}
      </div>
    );
  };
  ```
- **自動無障礙測試**：整合 axe-core 進行無障礙測試
- **鍵盤導覽**：實作完整的鍵盤無障礙存取
- **螢幕閱讀器最佳化**：使用 NVDA、JAWS 和 VoiceOver 測試

##### 國際化 (i18n) 與在地化
- **React-intl 整合**：多語言支援
  ```typescript
  import { FormattedMessage, useIntl } from 'react-intl';

  const WelcomeMessage = ({ userName }: { userName: string }) => {
    const intl = useIntl();
    
    return (
      <h1>
        <FormattedMessage
          id="welcome.title"
          defaultMessage="Welcome, {userName}!"
          values={{ userName }}
        />
      </h1>
    );
  };
  ```
- **語言偵測**：自動語言偵測與切換
- **RTL 支援**：阿拉伯文、希伯來文的從右到左語言支援
- **日期與數字格式化**：地區設定特定的格式化
- **翻譯管理**：與翻譯服務整合

## 目前限制與因應措施

### 已知限制

- 尚不支援內容安全原則 (CSP)
- 不支援儲存空間 SAS IP 限制
- 沒有 Power Platform Git 整合
- 不支援 Dataverse 解決方案
- 沒有原生 Azure Application Insights 整合

### 因應措施

- 如有需要，使用替代的錯誤追蹤解決方案
- 實作手動部署工作流程
- 使用外部工具進行進階分析
- 規劃將來遷移至支援的功能

## 文件標準

- 維護包含設定指示的完整 README.md
- 記錄所有自訂元件和 hooks
- 包含常見問題的疑難排解指南
- 記錄部署程序與需求
- 維護版本更新的變更紀錄
- 包含主要選擇的架構決策記錄

## 疑難排解常見問題

### 開發問題

- **連接埠 3000 衝突**：使用 `netstat -ano | findstr :3000` 殺死現有處理程序，然後使用 `taskkill /PID {PID} /F`
- **身驗失敗**：使用 `pac auth list` 驗證環境設定和使用者權限
- **套件安裝失敗**：使用 `npm cache clean --force` 清除 npm 快取並重新安裝
- **TypeScript 編譯錯誤**：檢查 verbatimModuleSyntax 設定和 SDK 相容性
- **連接器權限錯誤**：確保適當的同意流程和管理員權限
- **PowerProvider 初始化錯誤**：檢查控制台中的 SDK 初始化失敗
- **Vite dev 伺服器問題**：確保主機和連接埠設定符合要求

### 部署問題

- **建置失敗**：使用 `npm audit` 驗證所有相依套件並檢查建置設定
- **身驗錯誤**：使用 `pac auth clear` 重新驗證 PAC CLI，然後使用 `pac auth create`
- **連接器無法使用**：驗證 Power Platform 中的連接器設定和連線狀態
- **效能問題**：使用 `npm run build --report` 最佳化束大小並實作快取
- **環境不相符**：使用 `pac env list` 確認正確的環境選擇
- **應用程式逾時錯誤**：檢查 PowerProvider.tsx 實作與網路連線

### 執行階段問題

- **「應用程式逾時」錯誤**：驗證已執行 npm run build 且 PowerProvider 無錯誤
- **連接器身驗提示**：確保適當的同意流程實作
- **資料載入失敗**：檢查網路要求和連接器權限
- **UI 呈現問題**：驗證 Fluent UI 相容性與響應式設計實作

## 最佳實踐摘要

1. **遵循 Microsoft 官方文件與最佳實踐**
2. **使用 TypeScript 以確保型別安全與更佳的開發人員體驗**
3. **實作適當的錯誤處理與使用者回饋**
4. **最佳化效能與使用者體驗**
5. **遵循安全最佳實踐與 Power Platform 原則**
6. **編寫可維護、可測試且經過充分記錄的代碼**
7. **使用 PAC CLI 產生的服務與模型**
8. **規劃將來的功能更新與遷移**
9. **實作全面的測試策略**

10. **遵循適當的 DevOps 與部署實踐**
