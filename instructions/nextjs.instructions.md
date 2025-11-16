---
applyTo: '**'
---

# Next.js 最佳實踐指南（2025）

_最後更新：2025年7月_

本文件總結了建構、架構和維護 Next.js 應用程式的最新權威最佳實踐。本文件旨在供 LLM 和開發人員使用，以確保程式碼品質、可維護性和可擴展性。

---

## 1. 專案結構與組織

- **使用 `app/` 目錄**（App Router）用於所有新專案。優先選擇它而非舊版的 `pages/` 目錄。
- **頂層資料夾：**
  - `app/` — 路由、版面配置、頁面和路由處理器
  - `public/` — 靜態資源（圖片、字型等）
  - `lib/` — 共享工具、API 客戶端和邏輯
  - `components/` — 可重複使用的 UI 元件
  - `contexts/` — React context 提供者
  - `styles/` — 全域和模組化樣式表
  - `hooks/` — 自訂 React hooks
  - `types/` — TypeScript 型別定義
- **就近配置：** 將檔案（元件、樣式、測試）放在使用它們的位置附近，但避免過深的巢狀結構。
- **路由群組：** 使用括號（例如 `(admin)`）來群組路由而不影響 URL 路徑。
- **私有資料夾：** 以 `_` 作為前綴（例如 `_internal`）來選擇退出路由並表示實作細節。

- **功能資料夾：** 對於大型應用程式，按功能分組（例如 `app/dashboard/`、`app/auth/`）。
- **使用 `src/`**（可選）：將所有原始碼放在 `src/` 中以與設定檔分離。

## 2.1. Server 和 Client 元件整合（App Router）

**永遠不要在 Server Component 中使用 `next/dynamic` 搭配 `{ ssr: false }`。** 這不受支援且會導致建置/執行時期錯誤。

**正確方法：**
- 如果您需要在 Server Component 中使用 Client Component（例如使用 hooks、瀏覽器 API 或僅限客戶端函式庫的元件），您必須：
  1. 將所有僅限客戶端的邏輯/UI 移到專用的 Client Component 中（頂部加上 `'use client'`）。
  2. 在 Server Component 中直接匯入並使用該 Client Component（不需要 `next/dynamic`）。
  3. 如果您需要組合多個僅限客戶端的元素（例如帶有個人檔案下拉選單的導覽列），請建立一個包含所有這些元素的單一 Client Component。

**範例：**

```tsx
// Server Component
import DashboardNavbar from '@/components/DashboardNavbar';

export default async function DashboardPage() {
  // ...伺服器邏輯...
  return (
    <>
      <DashboardNavbar /> {/* 這是一個 Client Component */}
      {/* ...頁面其餘的伺服器渲染部分... */}
    </>
  );
}
```

**原因：**
- Server Components 不能使用僅限客戶端的功能或停用 SSR 的動態匯入。
- Client Components 可以在 Server Components 中渲染，但反之則不行。

**總結：**
始終將僅限客戶端的 UI 移到 Client Component 中，並在您的 Server Component 中直接匯入它。永遠不要在 Server Component 中使用 `next/dynamic` 搭配 `{ ssr: false }`。

---

## 2. 元件最佳實踐

- **元件類型：**
  - **Server Components**（預設）：用於資料擷取、繁重邏輯和非互動式 UI。
  - **Client Components：** 在頂部加上 `'use client'`。用於互動性、狀態或瀏覽器 API。
- **何時建立元件：**
  - 如果 UI 模式被重複使用超過一次。
  - 如果頁面的某個區段複雜或自成一體。
  - 如果它提高了可讀性或可測試性。
- **命名慣例：**
  - 元件檔案和匯出使用 `PascalCase`（例如 `UserCard.tsx`）。
  - Hooks 使用 `camelCase`（例如 `useUser.ts`）。
  - 靜態資源使用 `snake_case` 或 `kebab-case`（例如 `logo_dark.svg`）。
  - 將 context 提供者命名為 `XyzProvider`（例如 `ThemeProvider`）。
- **檔案命名：**
  - 將元件名稱與檔案名稱匹配。
  - 對於單一匯出檔案，預設匯出元件。
  - 對於多個相關元件，使用 `index.ts` barrel 檔案。
- **元件位置：**
  - 將共享元件放在 `components/` 中。
  - 將路由特定的元件放在相關的路由資料夾中。
- **Props：**
  - 為 props 使用 TypeScript 介面。
  - 優先使用明確的 prop 型別和預設值。
- **測試：**
  - 將測試與元件放在一起（例如 `UserCard.test.tsx`）。

## 3. 命名慣例（一般）

- **資料夾：** `kebab-case`（例如 `user-profile/`）
- **檔案：** 元件使用 `PascalCase`，工具/hooks 使用 `camelCase`，靜態資源使用 `kebab-case`
- **變數/函式：** `camelCase`
- **型別/介面：** `PascalCase`
- **常數：** `UPPER_SNAKE_CASE`

## 4. API 路由（Route Handlers）

- **優先使用 API Routes 而非 Edge Functions**，除非您需要超低延遲或地理分佈。
- **位置：** 將 API 路由放在 `app/api/` 中（例如 `app/api/users/route.ts`）。
- **HTTP 方法：** 匯出以 HTTP 動詞命名的非同步函式（`GET`、`POST` 等）。
- **請求/回應：** 使用 Web `Request` 和 `Response` API。使用 `NextRequest`/`NextResponse` 以獲得進階功能。
- **動態區段：** 為動態 API 路由使用 `[param]`（例如 `app/api/users/[id]/route.ts`）。
- **驗證：** 始終驗證和清理輸入。使用 `zod` 或 `yup` 等函式庫。
- **錯誤處理：** 回傳適當的 HTTP 狀態碼和錯誤訊息。
- **身份驗證：** 使用中介軟體或伺服器端 session 檢查來保護敏感路由。

## 5. 一般最佳實踐

- **TypeScript：** 為所有程式碼使用 TypeScript。在 `tsconfig.json` 中啟用 `strict` 模式。
- **ESLint & Prettier：** 強制執行程式碼風格和 linting。使用官方的 Next.js ESLint 設定。
- **環境變數：** 將機密資訊儲存在 `.env.local` 中。永遠不要將機密資訊提交到版本控制。
- **測試：** 使用 Jest、React Testing Library 或 Playwright。為所有關鍵邏輯和元件編寫測試。
- **無障礙性：** 使用語義化 HTML 和 ARIA 屬性。使用螢幕閱讀器進行測試。
- **效能：**
  - 使用內建的 Image 和 Font 最佳化。
  - 為非同步資料使用 Suspense 和載入狀態。
  - 避免大型客戶端套件；將大部分邏輯保留在 Server Components 中。
- **安全性：**
  - 清理所有使用者輸入。
  - 在生產環境中使用 HTTPS。
  - 設定安全的 HTTP 標頭。
- **文件：**
  - 編寫清晰的 README 和程式碼註解。
  - 記錄公開 API 和元件。

# 避免不必要的範例檔案

除非使用者特別要求實際範例、Storybook story 或明確的文件元件，否則不要在主要程式碼庫中建立範例/示範檔案（例如 ModalExample.tsx）。預設保持儲存庫整潔且專注於生產。

# 始終使用最新的文件和指南
- 對於每個 nextjs 相關的請求，首先搜尋最新的 nextjs 文件、指南和範例。
- 如果可用，使用以下工具來擷取和搜尋文件：
  - `resolve_library_id` 來解析文件中的套件/函式庫名稱。
  - `get_library_docs` 以取得最新文件。


