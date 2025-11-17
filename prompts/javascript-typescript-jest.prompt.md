---
description: '使用 Jest 編寫 JavaScript/TypeScript 測試的最佳實踐，包括模擬策略、測試結構和常見的模式。'
mode: 'agent'
---

### 測試結構
- 測試檔案命名為 `.test.ts` 或 `.test.js` 後綴
- 將測試檔案放在所測試程式碼旁邊，或在專用的 `__tests__` 目錄中
- 使用描述性測試名稱，說明預期的行為
- 使用巢狀 describe 區塊來組織相關的測試
- 遵循模式：`describe('Component/Function/Class', () => { it('should do something', () => {}) })`

### 有效的模擬
- 模擬外部依賴（API、資料庫等）以隔離測試
- 使用 `jest.mock()` 進行模組級別的模擬
- 使用 `jest.spyOn()` 進行特定函式的模擬
- 使用 `mockImplementation()` 或 `mockReturnValue()` 定義模擬行為
- 在 `afterEach` 中使用 `jest.resetAllMocks()` 重設模擬

### 測試非同步程式碼
- 在測試中始終返回 Promise 或使用 async/await 語法
- 使用 `resolves`/`rejects` 匹配器來處理 Promise
- 使用 `jest.setTimeout()` 為較慢的測試設定適當的逾時

### 快照測試
- 對於較少變化的 UI 元件或複雜物件，使用快照測試
- 保持快照小巧且專注
- 在提交之前仔細檢查快照變化

### 測試 React 元件
- 在測試元件時使用 React Testing Library 而非 Enzyme
- 測試使用者行為和元件的無障礙特性
- 透過無障礙角色、標籤或文字內容查詢元素
- 使用 `userEvent` 而非 `fireEvent` 以進行更逼真的使用者互動

## 常見的 Jest 匹配器
- 基礎：`expect(value).toBe(expected)`、`expect(value).toEqual(expected)`
- 真假值：`expect(value).toBeTruthy()`、`expect(value).toBeFalsy()`
- 數字：`expect(value).toBeGreaterThan(3)`、`expect(value).toBeLessThanOrEqual(3)`
- 字串：`expect(value).toMatch(/pattern/)`、`expect(value).toContain('substring')`
- 陣列：`expect(array).toContain(item)`、`expect(array).toHaveLength(3)`
- 物件：`expect(object).toHaveProperty('key', value)`
- 異常：`expect(fn).toThrow()`、`expect(fn).toThrow(Error)`
- 模擬函式：`expect(mockFn).toHaveBeenCalled()`、`expect(mockFn).toHaveBeenCalledWith(arg1, arg2)`
