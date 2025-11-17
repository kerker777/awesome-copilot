---
description: 'Playwright test generation instructions'
applyTo: '**'
---

## 測試撰寫指南

### 程式碼品質標準
- **定位器（Locators）**：優先使用面向使用者的、基於角色的定位器（`getByRole`、`getByLabel`、`getByText` 等）以提高彈性和無障礙性。使用 `test.step()` 來分組互動並提高測試的可讀性和報告。
- **斷言（Assertions）**：使用自動重試的 web-first 斷言。這些斷言以 `await` 關鍵字開頭（例如 `await expect(locator).toHaveText()`）。除非特別測試可見性變化，否則避免使用 `expect(locator).toBeVisible()`。
- **超時（Timeouts）**：依賴 Playwright 的內建自動等待機制。避免硬編碼等待或增加預設超時時間。
- **清晰度**：使用清楚說明意圖的描述性測試和步驟標題。僅在解釋複雜邏輯或非明顯互動時添加註解。


### 測試結構
- **匯入（Imports）**：以 `import { test, expect } from '@playwright/test';` 開頭。
- **組織**：在 `test.describe()` 區塊下將功能的相關測試分組。
- **鉤子（Hooks）**：對於 `describe` 區塊中所有測試通用的設置動作使用 `beforeEach`（例如導航到頁面）。
- **標題**：遵循清晰的命名慣例，例如 `功能 - 特定動作或場景`。


### 檔案組織
- **位置**：將所有測試檔案儲存在 `tests/` 目錄中。
- **命名**：使用慣例 `<feature-or-page>.spec.ts`（例如 `login.spec.ts`、`search.spec.ts`）。
- **範圍**：每個主要應用程式功能或頁面設置一個測試檔案。

### 斷言最佳實踐
- **UI 結構**：使用 `toMatchAriaSnapshot` 來驗證元件的無障礙樹結構。這提供了全面且可存取的快照。
- **元素計數**：使用 `toHaveCount` 來斷言定位器找到的元素數量。
- **文字內容**：對於精確的文字匹配使用 `toHaveText`，對於部分匹配使用 `toContainText`。
- **導航**：在動作後使用 `toHaveURL` 來驗證頁面 URL。


## 範例測試結構

```typescript
import { test, expect } from '@playwright/test';

test.describe('Movie Search Feature', () => {
  test.beforeEach(async ({ page }) => {
    // 在每個測試之前導航到應用程式
    await page.goto('https://debs-obrien.github.io/playwright-movies-app');
  });

  test('Search for a movie by title', async ({ page }) => {
    await test.step('Activate and perform search', async () => {
      await page.getByRole('search').click();
      const searchInput = page.getByRole('textbox', { name: 'Search Input' });
      await searchInput.fill('Garfield');
      await searchInput.press('Enter');
    });

    await test.step('Verify search results', async () => {
      // 驗證搜尋結果的無障礙樹
      await expect(page.getByRole('main')).toMatchAriaSnapshot(`
        - main:
          - heading "Garfield" [level=1]
          - heading "search results" [level=2]
          - list "movies":
            - listitem "movie":
              - link "poster of The Garfield Movie The Garfield Movie rating":
                - /url: /playwright-movies-app/movie?id=tt5779228&page=1
                - img "poster of The Garfield Movie"
                - heading "The Garfield Movie" [level=2]
      `);
    });
  });
});
```

## 測試執行策略

1. **初始執行**：使用 `npx playwright test --project=chromium` 執行測試
2. **偵錯失敗**：分析測試失敗並識別根本原因
3. **迭代**：根據需要精煉定位器、斷言或測試邏輯
4. **驗證**：確保測試穩定通過並涵蓋預期功能
5. **報告**：提供關於測試結果和發現的任何問題的回饋

## 品質檢查清單

在完成測試之前，確保：
- [ ] 所有定位器都是可存取和特定的，並避免嚴格模式違規
- [ ] 測試按邏輯分組並遵循清晰的結構
- [ ] 斷言有意義且反映使用者期望
- [ ] 測試遵循一致的命名慣例
- [ ] 程式碼正確格式化並包含註解
