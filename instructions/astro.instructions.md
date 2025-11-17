---
description: '內容驅動網站的 Astro 開發標準和最佳實踐'
applyTo: '**/*.astro, **/*.ts, **/*.js, **/*.md, **/*.mdx'
---

# Astro 開發指示

遵循現代最佳實踐,使用內容驅動、伺服器優先架構建立高品質 Astro 應用程式的指示。

## 專案情境
- Astro 5.x 搭配 Islands 架構和 Content Layer API
- TypeScript 以確保型別安全和更好的 DX 與自動產生的型別
- 內容驅動網站(部落格、行銷、電子商務、文件)
- 伺服器優先渲染搭配選擇性客戶端水合
- 支援多種 UI 框架(React、Vue、Svelte、Solid 等)
- 預設靜態網站產生(SSG)搭配可選的伺服器端渲染(SSR)
- 透過現代內容載入和建置最佳化提升效能

## 開發標準

### 架構
- 擁抱 Islands 架構:預設伺服器渲染,選擇性水合
- 使用 Content Collections 組織內容,以實現型別安全的 Markdown/MDX 管理
- 按功能或內容類型結構專案以實現可擴展性
- 使用基於元件的架構,明確分離關注點
- 實作漸進增強模式
- 遵循多頁應用程式(MPA)方法而非單頁應用程式(SPA)模式

### TypeScript 整合
- 使用建議的 v5.0 設定配置 `tsconfig.json`:
```json
{
  "extends": "astro/tsconfigs/base",
  "include": [".astro/types.d.ts", "**/*"],
  "exclude": ["dist"]
}
```
- 型別在 `.astro/types.d.ts` 中自動產生(取代 `src/env.d.ts`)
- 執行 `astro sync` 以產生/更新型別定義
- 使用 TypeScript 介面定義元件屬性
- 利用 Content Collections 和 Content Layer API 的自動產生型別

### 元件設計
- 對靜態、伺服器渲染內容使用 `.astro` 元件
- 僅在需要互動性時匯入框架元件(React、Vue、Svelte)
- 遵循 Astro 的元件腳本結構:頂部前言,下方範本
- 使用遵循 PascalCase 慣例的有意義元件名稱
- 保持元件專注且可組合
- 實作適當的屬性驗證和預設值

### Content Collections

#### 現代 Content Layer API(v5.0+)
- 使用新的 Content Layer API 在 `src/content.config.ts` 中定義集合
- 使用內建載入器:`glob()` 用於基於文件的內容,`file()` 用於單一文件
- 利用新載入系統的增強效能和可擴展性
- Content Layer API 範例:
```typescript
import { defineCollection, z } from 'astro:content';
import { glob } from 'astro/loaders';

const blog = defineCollection({
  loader: glob({ pattern: '**/*.md', base: './src/content/blog' }),
  schema: z.object({
    title: z.string(),
    pubDate: z.date(),
    tags: z.array(z.string()).optional()
  })
});
```

#### 舊版 Collections(向後相容)
- 透過自動 glob() 實作仍支援舊版 `type: 'content'` 集合
- 透過新增明確的 `loader` 配置遷移現有集合
- 使用 `getCollection()` 和 `getEntry()` 進行型別安全查詢
- 使用前言驗證和自動產生的型別組織內容

### View Transitions 與客戶端路由
- 在佈局頭部使用 `<ClientRouter />` 元件啟用(在 v5.0 中從 `<ViewTransitions />` 重新命名)
- 從 `astro:transitions` 匯入:`import { ClientRouter } from 'astro:transitions'`
- 提供類似 SPA 的導航而無需完整頁面重新載入
- 使用 CSS 和 view-transition-name 自訂過渡動畫
- 使用持久性 islands 維護跨頁面導航的狀態
- 使用 `transition:persist` 指令保留元件狀態

### 效能最佳化
- 預設零 JavaScript - 僅在需要時新增互動性
- 策略性使用客戶端指令(`client:load`、`client:idle`、`client:visible`)
- 對圖片和元件實作延遲載入
- 使用 Astro 的內建最佳化來最佳化靜態資源
- 利用 Content Layer API 實現更快的內容載入和建置
- 透過避免不必要的客戶端 JavaScript 最小化套件大小

### 樣式
- 預設在 `.astro` 元件中使用範疇樣式
- 需要時實作 CSS 前處理器(Sass、Less)
- 使用 CSS 自訂屬性進行主題設定和設計系統
- 遵循行動優先的響應式設計原則
- 使用語義化 HTML 和適當的 ARIA 屬性確保無障礙性
- 考慮使用工具優先框架(Tailwind CSS)進行快速開發

### 客戶端互動性
- 對互動元素使用框架元件(React、Vue、Svelte)
- 根據使用者互動模式選擇正確的水合策略
- 在框架邊界內實作狀態管理
- 謹慎處理客戶端路由以維持 MPA 優勢
- 使用 Web Components 實現框架無關的互動性
- 使用儲存或自訂事件在 islands 之間共享狀態

### API 路由和 SSR
- 在 `src/pages/api/` 中建立 API 路由以實現動態功能
- 使用適當的 HTTP 方法和狀態碼
- 實作請求驗證和錯誤處理
- 為動態內容需求啟用 SSR 模式
- 使用中介軟體進行驗證和請求處理
- 安全處理環境變數

### SEO 和 Meta 管理
- 使用 Astro 的內建 SEO 元件和 meta 標籤管理
- 實作適當的 Open Graph 和 Twitter Card 中繼資料
- 自動產生網站地圖以改善搜尋索引
- 使用語義化 HTML 結構以改善無障礙性和 SEO
- 實作結構化資料(JSON-LD)以獲得豐富摘要
- 為搜尋引擎最佳化頁面標題和描述

### 圖片最佳化
- 使用 Astro 的 `<Image />` 元件進行自動最佳化
- 使用適當的 srcset 產生實作響應式圖片
- 對現代瀏覽器使用 WebP 和 AVIF 格式
- 延遲載入摺疊下方的圖片
- 為無障礙性提供適當的替代文字
- 在建置時最佳化圖片以獲得更好的效能

### 資料擷取
- 在元件前言中在建置時擷取資料
- 對條件資料載入使用動態匯入
- 對外部 API 呼叫實作適當的錯誤處理
- 在建置過程中快取昂貴的操作
- 使用 Astro 的內建 fetch 搭配自動 TypeScript 推斷
- 適當處理載入狀態和後備方案

### 建置與部署
- 使用 Astro 的內建最佳化來最佳化靜態資源
- 配置靜態(SSG)或混合(SSR)渲染的部署
- 使用環境變數進行配置管理
- 為生產建置啟用壓縮和快取

## 主要 Astro v5.0 更新

### 重大變更
- **ClientRouter**:使用 `<ClientRouter />` 而不是 `<ViewTransitions />`
- **TypeScript**:在 `.astro/types.d.ts` 中自動產生型別(執行 `astro sync`)
- **Content Layer API**:新的 `glob()` 和 `file()` 載入器以增強效能

### 遷移範例
```typescript
// 現代 Content Layer API
import { defineCollection, z } from 'astro:content';
import { glob } from 'astro/loaders';

const blog = defineCollection({
  loader: glob({ pattern: '**/*.md', base: './src/content/blog' }),
  schema: z.object({ title: z.string(), pubDate: z.date() })
});
```

## 實作指南

### 開發工作流程
1. 使用 `npm create astro@latest` 搭配 TypeScript 範本
2. 使用適當的載入器配置 Content Layer API
3. 使用 `astro sync` 設定 TypeScript 以產生型別
4. 使用 Islands 架構建立佈局元件
5. 實作具有 SEO 和效能最佳化的內容頁面

### Astro 特定最佳實踐
- **Islands 架構**:伺服器優先搭配使用客戶端指令的選擇性水合
- **Content Layer API**:使用 `glob()` 和 `file()` 載入器進行可擴展的內容管理
- **零 JavaScript**:預設靜態渲染,僅在需要時新增互動性
- **View Transitions**:使用 `<ClientRouter />` 啟用類似 SPA 的導航
- **型別安全**:利用 Content Collections 的自動產生型別
- **效能**:使用內建圖片最佳化和最小客戶端套件進行最佳化
