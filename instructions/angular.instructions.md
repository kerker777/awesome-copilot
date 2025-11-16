---
description: 'Angular 專用編碼標準和最佳實踐'
applyTo: '**/*.ts, **/*.html, **/*.scss, **/*.css'
---

# Angular 開發指示

使用 TypeScript 產生高品質 Angular 應用程式的指示，使用 Angular Signals 進行狀態管理，遵循 https://angular.dev 概述的 Angular 最佳實踐。

## 專案情境
- 最新的 Angular 版本(預設使用獨立元件)
- TypeScript 以確保型別安全
- Angular CLI 用於專案設定和腳手架
- 遵循 Angular 樣式指南(https://angular.dev/style-guide)
- 使用 Angular Material 或其他現代 UI 函式庫以確保一致的樣式(如果指定)

## 開發標準

### 架構
- 使用獨立元件，除非明確需要模組
- 按獨立功能模組或領域組織程式碼以實現可擴展性
- 為功能模組實作延遲載入以最佳化效能
- 有效使用 Angular 的內建依賴注入系統
- 以明確的關注點分離來結構元件(智慧元件與展示元件)

### TypeScript
- 在 `tsconfig.json` 中啟用嚴格模式以確保型別安全
- 為元件、服務和模型定義清晰的介面和型別
- 使用型別守衛和聯合型別進行穩健的型別檢查
- 使用 RxJS 運算子(例如 `catchError`)實作適當的錯誤處理
- 對反應式表單使用型別化表單(例如 `FormGroup`、`FormControl`)

### 元件設計
- 遵循 Angular 元件生命週期鉤子的最佳實踐
- 當使用 Angular >= 19 時，使用 `input()`、`output()`、`viewChild()`、`viewChildren()`、`contentChild()` 和 `contentChildren()` 函數而不是裝飾器；否則使用裝飾器
- 利用 Angular 的變更偵測策略(預設或 `OnPush` 以提升效能)
- 保持範本清晰，將邏輯放在元件類別或服務中
- 使用 Angular 指令和管道實現可重用功能

### 樣式
- 使用 Angular 的元件級 CSS 封裝(預設：ViewEncapsulation.Emulated)
- 偏好使用 SCSS 進行樣式設定，並保持一致的主題
- 使用 CSS Grid、Flexbox 或 Angular CDK Layout 工具實作響應式設計
- 如果使用，遵循 Angular Material 的主題指南
- 使用 ARIA 屬性和語義化 HTML 維護無障礙性(a11y)

### 狀態管理
- 在元件和服務中使用 Angular Signals 進行反應式狀態管理
- 利用 `signal()`、`computed()` 和 `effect()` 進行反應式狀態更新
- 對可變狀態使用可寫信號，對衍生狀態使用計算信號
- 使用信號和適當的 UI 回饋處理載入和錯誤狀態
- 在將信號與 RxJS 結合使用時，使用 Angular 的 `AsyncPipe` 在範本中處理可觀察物件

### 資料擷取
- 使用 Angular 的 `HttpClient` 進行具有適當型別的 API 呼叫
- 實作 RxJS 運算子進行資料轉換和錯誤處理
- 在獨立元件中使用 Angular 的 `inject()` 函數進行依賴注入
- 實作快取策略(例如，對可觀察物件使用 `shareReplay`)
- 將 API 回應資料儲存在信號中以實現反應式更新
- 使用全域攔截器處理 API 錯誤以確保一致的錯誤處理

### 安全性
- 使用 Angular 的內建清理功能清理使用者輸入
- 實作路由守衛以進行驗證和授權
- 使用 Angular 的 `HttpInterceptor` 進行 CSRF 保護和 API 驗證標頭
- 使用 Angular 的反應式表單和自訂驗證器驗證表單輸入
- 遵循 Angular 的安全最佳實踐(例如，避免直接 DOM 操作)

### 效能
- 使用 `ng build --prod` 啟用生產建置以進行最佳化
- 對路由使用延遲載入以減少初始套件大小
- 使用 `OnPush` 策略和信號最佳化變更偵測以實現細粒度反應性
- 在 `ngFor` 迴圈中使用 trackBy 以提升渲染效能
- 使用 Angular Universal 實作伺服器端渲染(SSR)或靜態網站產生(SSG)(如果指定)

### 測試
- 使用 Jasmine 和 Karma 為元件、服務和管道撰寫單元測試
- 使用 Angular 的 `TestBed` 進行具有模擬依賴的元件測試
- 使用 Angular 的測試工具測試基於信號的狀態更新
- 使用 Cypress 或 Playwright 撰寫端對端測試(如果指定)
- 使用 `provideHttpClientTesting` 模擬 HTTP 請求
- 確保關鍵功能的高測試覆蓋率

## 實作流程
1. 規劃專案結構和功能模組
2. 定義 TypeScript 介面和模型
3. 使用 Angular CLI 腳手架元件、服務和管道
4. 使用基於信號的狀態實作資料服務和 API 整合
5. 建立具有清晰輸入和輸出的可重用元件
6. 新增反應式表單和驗證
7. 使用 SCSS 和響應式設計應用樣式
8. 實作延遲載入的路由和守衛
9. 使用信號新增錯誤處理和載入狀態
10. 撰寫單元和端對端測試
11. 最佳化效能和套件大小

## 附加指南
- 遵循 Angular 樣式指南的檔案命名慣例(參見 https://angular.dev/style-guide)，例如，對元件使用 `feature.ts`，對服務使用 `feature-service.ts`。對於舊版程式碼庫，保持與現有模式的一致性。
- 使用 Angular CLI 命令產生樣板程式碼
- 使用清晰的 JSDoc 註解記錄元件和服務
- 確保無障礙性合規性(WCAG 2.1)(如適用)
- 使用 Angular 的內建 i18n 進行國際化(如果指定)
- 透過建立可重用的工具和共享模組保持程式碼 DRY
- 一致地使用信號進行狀態管理以確保反應式更新
