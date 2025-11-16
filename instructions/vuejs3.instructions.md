---
description: 'VueJS 3 開發標準和最佳實踐，搭配 Composition API 和 TypeScript'
applyTo: '**/*.vue, **/*.ts, **/*.js, **/*.scss'
---

# VueJS 3 開發指引

使用 Composition API、TypeScript 和現代最佳實踐構建高品質 VueJS 3 應用程式的指引。

## 專案背景
- Vue 3.x，預設使用 Composition API
- TypeScript 用於型別安全
- 單檔案元件（`.vue`），使用 `<script setup>` 語法
- 現代構建工具（建議使用 Vite）
- Pinia 用於應用程式狀態管理
- 官方 Vue 風格指南和最佳實踐

## 開發標準

### 架構
- 優先使用 Composition API（`setup` 函式和 composables）而非 Options API
- 按功能或領域組織元件和 composables 以實現可擴展性
- 分離 UI 導向元件（展示型）和邏輯導向元件（容器型）
- 將可重用邏輯提取到 `composables/` 目錄中的 composable 函式
- 按領域建構 store 模組（Pinia），具有明確定義的 actions、state 和 getters

### TypeScript 整合
- 在 `tsconfig.json` 中啟用 `strict` 模式以獲得最大型別安全
- 使用 `defineComponent` 或 `<script setup lang="ts">`，搭配 `defineProps` 和 `defineEmits`
- 對型別化 props 和預設值利用 `PropType<T>`
- 對複雜 prop 和 state 形狀使用介面或型別別名
- 為事件處理器、refs 和 `useRoute`/`useRouter` hooks 定義型別
- 在適用的地方實作泛型元件和 composables

### 元件設計
- 遵守元件的單一職責原則
- 元件名稱使用 PascalCase，檔案名稱使用 kebab-case
- 保持元件小型並專注於單一關注點
- 使用 `<script setup>` 語法以獲得簡潔性和效能
- 使用 TypeScript 驗證 props；僅在必要時使用執行時檢查
- 優先使用 slots 和 scoped slots 實現靈活組合

### 狀態管理
- 對全域狀態使用 Pinia：使用 `defineStore` 定義 stores
- 對簡單本地狀態，在 `setup` 中使用 `ref` 和 `reactive`
- 對衍生狀態使用 `computed`
- 對複雜結構保持狀態正規化
- 對非同步邏輯在 Pinia stores 中使用 actions
- 利用 store 外掛進行持久化或除錯

### Composition API 模式
- 為共享邏輯建立可重用的 composables，例如 `useFetch`、`useAuth`
- 使用具有精確依賴清單的 `watch` 和 `watchEffect`
- 在 `onUnmounted` 或 `watch` 清理回呼中清理副作用
- 謹慎使用 `provide`/`inject` 進行深度依賴注入
- 使用 `useAsyncData` 或第三方資料實用程式（Vue Query）

### 樣式
- 對元件級樣式使用 `<style scoped>` 或 CSS Modules
- 考慮實用優先框架（Tailwind CSS）以快速設計樣式
- 對類別命名遵循 BEM 或函式式 CSS 慣例
- 利用 CSS 自訂屬性進行主題設定和設計 tokens
- 使用 CSS Grid 和 Flexbox 實作移動優先的響應式設計
- 確保樣式可訪問（對比度、焦點狀態）

### 效能優化
- 使用動態匯入和 `defineAsyncComponent` 延遲載入元件
- 對非同步元件載入備用使用 `<Suspense>`
- 對靜態或不常變化的元素應用 `v-once` 和 `v-memo`
- 使用 Vue DevTools 效能標籤進行分析
- 避免不必要的 watchers；盡可能優先使用 `computed`
- 樹搖動未使用的程式碼並利用 Vite 的優化功能

### 資料獲取
- 使用 composables 如 `useFetch`（Nuxt）或函式庫如 Vue Query
- 明確處理載入、錯誤和成功狀態
- 在元件卸載或參數更改時取消過時請求
- 在失敗時使用回滾實作樂觀更新
- 快取回應並使用背景重新驗證

### 錯誤處理
- 對未捕獲的錯誤使用全域錯誤處理器（`app.config.errorHandler`）
- 將風險邏輯包裝在 `try/catch` 中；提供使用者友善的訊息
- 在元件中使用 `errorCaptured` hook 作為本地邊界
- 優雅地顯示備用 UI 或錯誤警報
- 將錯誤記錄到外部服務（Sentry、LogRocket）

### 表單和驗證
- 使用函式庫如 VeeValidate 或 @vueuse/form 進行宣告式驗證
- 使用受控的 `v-model` 綁定構建表單
- 在 blur 或 input 時使用去抖動進行驗證以提高效能
- 在 composables 中處理檔案上傳和複雜的多步驟表單
- 確保可訪問的標籤、錯誤公告和焦點管理

### 路由
- 使用 Vue Router 4，搭配 `createRouter` 和 `createWebHistory`
- 實作巢狀路由和路由級程式碼分割
- 使用導航守衛（`beforeEnter`、`beforeEach`）保護路由
- 在 `setup` 中使用 `useRoute` 和 `useRouter` 進行程式化導航
- 正確管理查詢參數和動態區段
- 透過路由 meta 欄位實作麵包屑資料

### 測試
- 使用 Vue Test Utils 和 Jest 撰寫單元測試
- 專注於行為，而非實作細節
- 使用 `mount` 和 `shallowMount` 進行元件隔離
- 根據需要模擬全域外掛（router、Pinia）
- 使用 Cypress 或 Playwright 新增端對端測試
- 使用 axe-core 整合測試可訪問性

### 安全性
- 避免使用 `v-html`；嚴格清理任何 HTML 輸入
- 使用 CSP 標頭來緩解 XSS 和注入攻擊
- 在範本和指令中驗證和跳脫資料
- 對所有 API 請求使用 HTTPS
- 將敏感 tokens 儲存在 HTTP-only cookies 中，而非 `localStorage`

### 可訪問性
- 使用語義 HTML 元素和 ARIA 屬性
- 為模態和動態內容管理焦點
- 為互動元件提供鍵盤導航
- 為圖片和圖示新增有意義的 `alt` 文字
- 確保色彩對比度符合 WCAG AA 標準

## 實作流程
1. 規劃元件和 composable 架構
2. 使用 Vue 3 和 TypeScript 初始化 Vite 專案
3. 定義 Pinia stores 和 composables
4. 建立核心 UI 元件和佈局
5. 整合路由和導航
6. 實作資料獲取和狀態邏輯
7. 使用驗證和錯誤狀態構建表單
8. 新增全域錯誤處理和備用 UI
9. 新增單元和 E2E 測試
10. 優化效能和套件大小
11. 確保可訪問性合規性
12. 記錄元件、composables 和 stores

## 其他指南
- 遵循 Vue 的官方風格指南（vuejs.org/style-guide）
- 使用 ESLint（搭配 `plugin:vue/vue3-recommended`）和 Prettier 保持程式碼一致性
- 撰寫有意義的 commit 訊息並保持乾淨的 git 歷史記錄
- 保持依賴項最新並審查漏洞
- 使用 JSDoc/TSDoc 記錄複雜邏輯
- 使用 Vue DevTools 進行除錯和分析

## 常見模式
- 無渲染元件和 scoped slots 實現靈活 UI
- 使用 provide/inject 的複合元件
- 用於跨切面關注點的自訂指令
- Teleport 用於模態和覆蓋層
- 全域實用程式的外掛系統（i18n、analytics）
- 參數化邏輯的 Composable 工廠
