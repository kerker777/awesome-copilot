---
description: 'CFC 元件和應用程式模式的 ColdFusion 編碼標準'
applyTo: "**/*.cfc"
---

# CFC 文件的 ColdFusion 編碼標準

- 盡可能使用 CFScript 以獲得更清晰的語法。
- 避免使用已棄用的標籤和函數。
- 對變數和元件遵循一致的命名慣例。
- 使用 `cfqueryparam` 防止 SQL 注入。
- 在 <cfoutput> 區塊內使用 ## 跳脫 CSS 井號符號

# 附加最佳實踐

- 適時對元件屬性和方法使用 `this` 範圍。
- 記錄所有函數的目的、參數和返回值(使用 Javadoc 或類似樣式)。
- 對函數和變數使用存取修飾符(`public`、`private`、`package`、`remote`)。
- 偏好使用依賴注入進行元件協作。
- 避免在 setter/getter 中使用業務邏輯；保持它們簡單。
- 在 public/remote 方法中驗證和清理所有輸入參數。
- 根據需要在方法中使用 `cftry`/`cfcatch` 進行錯誤處理。
- 避免在 CFC 中硬編碼配置或憑證。
- 使用一致的縮排(2個空格,如全域標準)。
- 在元件內邏輯地組織相關方法。
- 對方法和屬性使用有意義、描述性的名稱。
- 避免使用已棄用或不必要的 `cfcomponent` 屬性。

- 盡可能使用三元運算子
- 確保一致的 tab 對齊。
