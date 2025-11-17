---
description: 'Svelte 5 和 SvelteKit 開發標準與最佳實踐，用於元件式使用者介面和全端應用程式'
applyTo: '**/*.svelte, **/*.ts, **/*.js, **/*.css, **/*.scss, **/*.json'
---

# Svelte 5 和 SvelteKit 開發指引

使用現代 runes 基礎反應性、TypeScript 和效能優化構建高品質 Svelte 5 和 SvelteKit 應用程式的指引。

## 專案背景
- Svelte 5.x 搭配 runes 系統（$state、$derived、$effect、$props、$bindable）
- SvelteKit 用於具有檔案式路由的全端應用程式
- TypeScript 用於型別安全和更好的開發者體驗
- 元件範圍樣式搭配 CSS 自訂屬性
- 漸進增強和效能優先方法
- 具有優化的現代構建工具（Vite）

## 開發標準

### 架構
- 對所有反應性使用 Svelte 5 runes 系統，而非傳統 stores
- 按功能或領域組織元件以實現可擴展性
- 分離展示元件和邏輯密集型元件
- 將可重用邏輯提取到可組合函式中
- 使用 slots 和 snippets 實作適當的元件組合
- 使用 SvelteKit 的檔案式路由搭配適當的 load 函式

### TypeScript 整合
- 在 `tsconfig.json` 中啟用嚴格模式以獲得最大型別安全
- 使用 `$props()` 語法定義元件 props 的介面
- 為事件處理器、refs 和 SvelteKit 生成的型別加上型別
- 對可重用元件使用泛型型別
- 利用 SvelteKit 生成的 `$types.ts` 檔案
- 使用 `svelte-check` 實作適當的型別檢查

### 元件設計
- 遵循元件的單一職責原則
- 預設使用 `<script lang="ts">` 搭配 runes 語法
- 保持元件小型並專注於單一關注點
- 使用 TypeScript 實作適當的 prop 驗證
- 使用 slots 和 snippets 實現靈活組合
- 設計可測試和可重用的元件

### Svelte 5 Runes 系統
- 使用 `$state()` 進行反應式本地狀態管理
- 對計算值和昂貴計算實作 `$derived()`
- 對具有適當清理的副作用使用 `$effect()`
- 使用 `$props()` 和解構定義元件 props
- 對元件之間的雙向資料綁定使用 `$bindable()`
- 從傳統 stores 遷移到 runes 以獲得更好的效能

### 狀態管理
- 對本地元件狀態使用 `$state()`
- 使用 `setContext`/`getContext` 的 context API 實現共享狀態
- 在需要時使用 SvelteKit stores 進行全域應用程式狀態
- 對複雜資料結構保持狀態正規化
- 對計算值使用 derived 狀態
- 為客戶端資料實作適當的狀態持久化

### SvelteKit 模式
- 使用 `+page.svelte` 作為具有適當 SEO 的頁面元件
- 實作 `+layout.svelte` 用於共享佈局和導航
- 使用 `+page.server.ts` 進行伺服器端資料載入和 API 呼叫
- 在 `+page.server.ts` 中實作表單動作以進行資料變更
- 使用 `+server.ts` 進行 API 端點和伺服器端邏輯
- 使用 SvelteKit 的檔案式系統處理路由

### 樣式
- 使用 `<style>` 區塊的元件範圍樣式
- 為主題和設計系統實作 CSS 自訂屬性
- 使用 `class:` 指令進行條件樣式
- 遵循 BEM 或實用優先的 CSS 慣例
- 實作響應式設計，採用移動優先方法
- 謹慎使用 `:global()` 僅用於真正的全域樣式

### 效能優化
- 對高效列表渲染使用帶鍵的 `{#each}` 區塊
- 使用動態匯入和 `svelte:component` 實作延遲載入
- 使用 `$derived()` 進行昂貴的計算以避免不必要的重新計算
- 利用 SvelteKit 的自動程式碼分割和預載入
- 透過樹搖動和適當的匯入優化套件大小
- 使用 Svelte DevTools 分析以識別效能瓶頸

### 資料獲取
- 使用 SvelteKit 的 load 函式進行伺服器端和通用資料獲取
- 實作適當的載入、錯誤和成功狀態
- 在伺服器 load 函式中使用 promises 處理串流資料
- 使用 `invalidate()` 和 `invalidateAll()` 進行快取管理
- 實作樂觀更新以獲得更好的使用者體驗
- 優雅地處理離線場景和網路錯誤

### 錯誤處理
- 實作 `+error.svelte` 頁面用於路由級錯誤邊界
- 在 load 函式和表單動作中使用 try/catch 區塊
- 提供有意義的錯誤訊息和備用 UI
- 適當記錄錯誤以便除錯和監控
- 使用適當的使用者回饋處理表單中的驗證錯誤
- 使用 SvelteKit 的錯誤和重新導向輔助工具

### 表單和驗證
- 使用 SvelteKit 的表單動作進行伺服器端表單處理
- 使用 `use:enhance` 實作漸進增強
- 對受控表單輸入使用 `bind:value`
- 在客戶端和伺服器端驗證資料
- 處理檔案上傳和複雜表單場景
- 使用標籤和 ARIA 屬性實作適當的可訪問性

### 測試
- 使用 Vitest 和 Testing Library 為元件撰寫單元測試
- 測試元件行為，而非實作細節
- 使用 Playwright 進行使用者工作流程的端對端測試
- 適當模擬 SvelteKit 的 load 函式和 stores
- 徹底測試表單動作和 API 端點
- 使用 axe-core 實作可訪問性測試

### 安全性
- 清理使用者輸入以防止 XSS 攻擊
- 謹慎使用 `@html` 指令並驗證 HTML 內容
- 使用 SvelteKit 實作適當的 CSRF 保護
- 在 load 函式和表單動作中驗證和清理資料
- 對所有外部 API 呼叫和生產部署使用 HTTPS
- 使用適當的會話管理安全地儲存敏感資料

### 可訪問性
- 使用語義 HTML 元素和適當的標題層次結構
- 為所有互動元素實作鍵盤導航
- 提供適當的 ARIA 標籤和描述
- 確保色彩對比度符合 WCAG 指南
- 使用螢幕閱讀器和可訪問性工具進行測試
- 為動態內容實作焦點管理

## 實作流程
1. 使用 TypeScript 和所需適配器初始化 SvelteKit 專案
2. 設定專案結構，採用適當的資料夾組織
3. 定義 TypeScript 介面和元件 props
4. 使用 Svelte 5 runes 實作核心元件
5. 使用 SvelteKit 新增路由、佈局和導航
6. 實作資料載入和表單處理
7. 使用自訂屬性和響應式設計新增樣式系統
8. 實作錯誤處理和載入狀態
9. 新增全面的測試覆蓋率
10. 優化效能和套件大小
11. 確保可訪問性合規性
12. 使用適當的 SvelteKit 適配器部署

## 其他指南
- 遵循 Svelte 的命名慣例（元件使用 PascalCase，函式使用 camelCase）
- 使用 ESLint 搭配 eslint-plugin-svelte 和 Prettier 保持程式碼一致性
- 保持依賴項最新並審查安全漏洞
- 使用 JSDoc 記錄複雜元件和邏輯
- 使用 Svelte DevTools 進行除錯和效能分析
- 使用 SvelteKit 的 meta 標籤和結構化資料實作適當的 SEO
- 使用環境變數進行不同部署階段的配置

## 常見模式
- 使用 slots 的無渲染元件實現靈活的 UI 組合
- 用於跨切面關注點和 DOM 操作的自訂指令
- 基於 Snippet 的組合用於可重用的範本邏輯
- Context providers 用於應用程式範圍的狀態管理
- 表單和互動功能的漸進增強
- 伺服器端渲染搭配客戶端水合以獲得最佳效能
