---
description: 'R language and document formats (R, Rmd, Quarto): coding standards and Copilot guidance for idiomatic, safe, and consistent code generation.'
applyTo: '**/*.R, **/*.r, **/*.Rmd, **/*.rmd, **/*.qmd'
---

# R 程式語言指引

## 目的

協助 GitHub Copilot 在各個專案中生成習慣用法、安全且可維護的 R 程式碼。

## 核心慣例

- **配合專案風格。** 如果檔案顯示偏好（tidyverse 與 base R、`%>%` 與 `|>`），就遵循它。
- **優先使用清晰、向量化的程式碼。** 保持函數簡潔，避免隱藏的副作用。
- **在例子或片段中限定非基礎函數**，例如 `dplyr::mutate()`、`stringr::str_detect()`。在專案程式碼中，當符合倉庫規範時，使用 `library()` 是可以接受的。
- **命名：** 物件或檔案使用 `lower_snake_case`；避免在名稱中使用點。
- **副作用：** 永遠不要呼叫 `setwd()`；優先使用相對於專案的路徑（例如 `here::here()`）。
- **可重複性：** 使用 `withr::with_seed()` 在隨機操作周圍本地設定種子。
- **驗證：** 驗證並限制使用者輸入；盡可能使用類型檢查和允許清單。
- **安全性：** 避免 `eval(parse())`、未驗證的 shell 呼叫和未參數化的 SQL。

### 管道運算子

- **原生管道 `|>` (R ≥ 4.1.0)：** 在 R ≥ 4.1 中優先使用（無額外依賴）。
- **Magrittr 管道 `%>%`：** 在已經採用 magrittr 的專案中繼續使用，或當你需要 `.`、`%T>%` 或 `%$%` 功能時。
- **保持一致：** 在同一指令碼中不要混合 `|>` 和 `%>%`，除非有明確的技術原因。

## 效能考慮

- **大型資料集：** 考慮使用 `data.table`；使用你的工作負載進行基準測試。
- **dplyr 相容性：** 使用 `dtplyr` 編寫 dplyr 語法，自動轉譯為 data.table 操作以獲得效能增益。
- **分析：** 使用 `profvis::profvis()` 來識別程式碼中的效能瓶頸。在最佳化前先進行分析。
- **快取：** 使用 `memoise::memoise()` 來快取昂貴的函數結果。特別適用於重複的 API 呼叫或複雜的計算。
- **向量化：** 優先使用向量化操作而不是迴圈。對於剩餘的重複需求，使用 `purrr::map_*()` 系列或 `apply()` 系列。

## 工具與品質

- **格式化：** `styler`（tidyverse 風格）、兩空格縮進、約 100 字元行寬。
- **Linting：** `lintr` 透過 `.lintr` 配置。
- **預提交：** 考慮使用 `precommit` hooks 以自動進行 linting 和格式化。
- **文件：** 使用 roxygen2 為匯出函數編寫文件（`@param`、`@return`、`@examples`）。
- **測試：** 優先使用小型、純粹、可組合的函數，易於進行單元測試。
- **依賴項：** 使用 `renv` 管理；新增套件後執行快照。
- **路徑：** 優先使用 `fs` 和 `here` 以提高可攜性。

## 資料整理與 I/O

- **資料框：** 在 tidyverse 重型檔案中優先使用 tibbles；否則基礎 `data.frame()` 也可以。
- **重複操作：** 在 tidyverse 程式碼中使用 `purrr`。在基礎風格的程式碼中，優先使用類型穩定、向量化的模式，例如 `vapply()`（用於原子輸出）或 `Map()`（用於元素級操作），而不是明確的 `for` 迴圈（當它們提高清晰度或效能時）。
- **字串與日期：** 在已經存在的地方使用 `stringr` 或 `lubridate`；否則使用清晰的基礎協助函數（例如 `nchar()`、`substr()`、`as.Date()`，並指定明確的格式）。
- **I/O：** 優先使用明確、類型化的讀取函數（例如 `readr::read_csv()`）；明確說明解析假設。

## 製圖

- 對於發佈品質的圖表，優先使用 `ggplot2`。保持圖層易於閱讀，標註軸和單位。

## 錯誤處理

- 在 tidyverse 語境中，使用 `rlang::abort()` 或 `rlang::warn()` 進行結構化條件；在純基礎程式碼中，使用 `stop()` 或 `warning()`。
- 對於可恢復的操作：
- 當你想要相同類型的類型化回退值時，使用 `purrr::possibly()`（更簡單）。
- 當你需要捕捉結果和錯誤供後續檢查或記錄時，使用 `purrr::safely()`。
- 在基礎 R 中使用 `tryCatch()` 以獲得細粒度的控制或與非 tidyverse 程式碼的相容性。
- 優先使用一致的回傳結構—對於正常流程使用類型化輸出，僅當需要錯誤詳細資訊時使用結構化清單。

## 安全最佳實踐

- **命令執行：** 優先使用 `processx::run()` 或 `sys::exec_wait()`，而不是 `system()`；驗證和清理所有引數。
- **資料庫查詢：** 使用參數化的 `DBI` 查詢以防止 SQL 注入。
- **檔案路徑：** 規範化和清理使用者提供的路徑（例如 `fs::path_sanitize()`），並對允許清單進行驗證。
- **認證：** 永遠不要對秘密進行硬編碼。使用環境變數（`Sys.getenv()`）、VCS 外的配置或 `keyring`。

## Shiny

- 對於非瑣碎的應用程式，模組化 UI 和伺服器邏輯。對於明確的依賴項，使用 `eventReactive()` 或 `observeEvent()`。
- 使用 `req()` 驗證輸入並提供清晰、使用者友善的訊息。
- 為資料庫使用連接池（`pool`）；避免長期存在的全域物件。
- 隔離昂貴的計算，並優先使用 `reactiveVal()` 或 `reactiveValues()` 進行小型狀態管理。

## R Markdown 和 Quarto

- 保持區塊聚焦；優先使用明確的區塊選項（`echo`、`message`、`warning`）。
- 避免全域狀態；優先使用本地協助函數。對於確定性區塊，使用 `withr::with_seed()`。

## Copilot 特定指導

- 如果目前檔案使用 tidyverse，**建議 tidyverse 優先模式**（例如 `dplyr::across()`，而不是已棄用的動詞）。如果存在基礎 R 風格，**使用基礎習慣用法**。
- 在建議中限定非基礎呼叫（例如 `dplyr::mutate()`）。
- 當符合習慣用法時，建議向量化或整潔的解決方案而不是迴圈。
- 優先使用小型協助函數而不是長管道。
- 當多種方法等效時，優先考慮可讀性和類型穩定性，並解釋權衡。

---

## Minimal Examples

```r
# Base R variant
scores <- data.frame(id = 1:5, x = c(1, 3, 2, 5, 4))
safe_log <- function(x) tryCatch(log(x), error = function(e) NA_real_)
scores$z <- vapply(scores$x, safe_log, numeric(1))

# Tidyverse variant (if this file uses tidyverse)
result <- tibble::tibble(id = 1:5, x = c(1, 3, 2, 5, 4)) |>
dplyr::mutate(z = purrr::map_dbl(x, purrr::possibly(log, otherwise = NA_real_))) |>
dplyr::filter(z > 0)

# Example reusable helper with roxygen2 doc
#' Compute the z-score of a numeric vector
#' @param x A numeric vector
#' @return Numeric vector of z-scores
#' @examples z_score(c(1, 2, 3))
z_score <- function(x) (x - mean(x, na.rm = TRUE)) / stats::sd(x, na.rm = TRUE)
```
