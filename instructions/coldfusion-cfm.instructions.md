---
description: 'ColdFusion cfm 文件和應用程式模式'
applyTo: "**/*.cfm"
---

# ColdFusion 編碼標準

- 盡可能使用 CFScript 以獲得更清晰的語法。
- 避免使用已棄用的標籤和函數。
- 對變數和元件遵循一致的命名慣例。
- 使用 `cfqueryparam` 防止 SQL 注入。
- 在 <cfoutput> 區塊內使用 ## 跳脫 CSS 井號符號
- 在 <cfoutput> 區塊內使用 HTMX 時,透過使用雙井號(##)跳脫井號符號(#)以防止意外的變數插值。
- 如果您在 HTMX 目標文件中,請確保頂行為:<cfsetting showDebugOutput = "false">

# 附加最佳實踐

- 使用 `Application.cfc` 進行應用程式設定和請求處理。
- 將程式碼組織成可重用的 CFC(元件)以提高可維護性。
- 驗證和清理所有使用者輸入。
- 使用 `cftry`/`cfcatch` 進行錯誤處理和記錄。
- 避免在源文件中硬編碼憑證或敏感資料。
- 使用一致的縮排(2個空格,如全域標準)。
- 註解複雜邏輯並記錄函數的目的和參數。
- 偏好使用 `cfinclude` 進行共享範本,但避免循環包含。

- 盡可能使用三元運算子
- 確保一致的 tab 對齊。
