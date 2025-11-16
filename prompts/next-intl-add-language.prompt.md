---
mode: 'agent'
tools: ['changes','search/codebase', 'edit/editFiles', 'findTestFiles', 'search', 'writeTest']
description: '在 Next.js + next-intl 應用程式中新增語言'
---

這是在使用 next-intl 進行國際化的 Next.js 專案中新增語言的指南。

- 應用程式使用 next-intl 進行國際化。
- 所有翻譯都在 `./messages` 目錄中。
- UI 元件是 `src/components/language-toggle.tsx`。
- 路由和中介軟體設定處理於：
  - `src/i18n/routing.ts`
  - `src/middleware.ts`

新增語言時：

- 將 `en.json` 的所有內容翻譯成新語言。目標是將所有 JSON 條目翻譯成新語言以實現完整翻譯。
- 在 `routing.ts` 和 `middleware.ts` 中新增路徑。
- 在 `language-toggle.tsx` 中新增語言。
