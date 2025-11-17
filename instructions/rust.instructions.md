---
description: 'Rust 程式語言編碼慣例和最佳實踐'
applyTo: '**/*.rs'
---

# Rust 編碼慣例和最佳實踐

撰寫 Rust 程式碼時遵循慣用的 Rust 實踐和社群標準。

這些指引基於 [The Rust Book](https://doc.rust-lang.org/book/)、[Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)、[RFC 430 命名慣例](https://github.com/rust-lang/rfcs/blob/master/text/0430-finalizing-naming-conventions.md) 以及更廣泛的 Rust 社群 [users.rust-lang.org](https://users.rust-lang.org)。

## 一般指引

- 始終優先考慮可讀性、安全性和可維護性。
- 使用強型別並利用 Rust 的所有權系統實現記憶體安全。
- 將複雜函式分解為更小、更易於管理的函式。
- 對於演算法相關的程式碼，包含所使用方法的說明。
- 撰寫具有良好可維護性實踐的程式碼，包括對某些設計決策原因的註解。
- 使用 `Result<T, E>` 優雅地處理錯誤並提供有意義的錯誤訊息。
- 對於外部依賴項，在文檔中提及其用途和目的。
- 遵循 [RFC 430](https://github.com/rust-lang/rfcs/blob/master/text/0430-finalizing-naming-conventions.md) 使用一致的命名慣例。
- 撰寫符合語言習慣、安全且高效的 Rust 程式碼，遵循借用檢查器的規則。
- 確保程式碼編譯時沒有警告。

## 應遵循的模式

- 使用模組（`mod`）和公開介面（`pub`）來封裝邏輯。
- 使用 `?`、`match` 或 `if let` 適當處理錯誤。
- 使用 `serde` 進行序列化，使用 `thiserror` 或 `anyhow` 處理自訂錯誤。
- 實作 traits 來抽象服務或外部依賴項。
- 使用 `async/await` 和 `tokio` 或 `async-std` 建構非同步程式碼。
- 優先使用列舉而非旗標和狀態以實現型別安全。
- 對複雜物件建立使用建構器。
- 分離二進位和函式庫程式碼（`main.rs` vs `lib.rs`）以實現可測試性和重用性。
- 使用 `rayon` 進行資料並行處理和 CPU 密集型任務。
- 使用迭代器而非基於索引的迴圈，因為它們通常更快且更安全。
- 當不需要所有權時，函式參數使用 `&str` 而非 `String`。
- 優先使用借用和零拷貝操作以避免不必要的配置。

### 所有權、借用和生命週期

- 除非需要轉移所有權，否則優先使用借用（`&T`）而非複製。
- 當需要修改借用的資料時使用 `&mut T`。
- 當編譯器無法推斷時明確註解生命週期。
- 對單執行緒參考計數使用 `Rc<T>`，對執行緒安全的參考計數使用 `Arc<T>`。
- 在單執行緒環境中使用 `RefCell<T>` 實現內部可變性，在多執行緒環境中使用 `Mutex<T>` 或 `RwLock<T>`。

## 應避免的模式

- 除非絕對必要，否則不要使用 `unwrap()` 或 `expect()`—優先進行適當的錯誤處理。
- 避免在函式庫程式碼中發生 panic—改為回傳 `Result`。
- 不要依賴全域可變狀態—使用依賴注入或執行緒安全容器。
- 避免深度巢狀邏輯—使用函式或組合器重構。
- 不要忽略警告—在 CI 中將它們視為錯誤。
- 除非必要且有完整文檔，否則避免使用 `unsafe`。
- 不要過度使用 `clone()`，除非需要轉移所有權，否則使用借用而非複製。
- 避免過早使用 `collect()`，在實際需要集合之前保持迭代器惰性。
- 避免不必要的配置—優先使用借用和零拷貝操作。

## 程式碼風格和格式

- 遵循 Rust Style Guide 並使用 `rustfmt` 進行自動格式化。
- 儘可能保持行數在 100 個字元以下。
- 使用 `///` 在項目之前立即放置函式和結構體文檔。
- 使用 `cargo clippy` 捕捉常見錯誤並執行最佳實踐。

## 錯誤處理

- 對可恢復錯誤使用 `Result<T, E>`，僅對不可恢復錯誤使用 `panic!`。
- 優先使用 `?` 運算子而非 `unwrap()` 或 `expect()` 進行錯誤傳播。
- 使用 `thiserror` 建立自訂錯誤型別或實作 `std::error::Error`。
- 對可能存在或不存在的值使用 `Option<T>`。
- 提供有意義的錯誤訊息和上下文。
- 錯誤型別應該有意義且行為良好（實作標準 traits）。
- 驗證函式參數並為無效輸入回傳適當的錯誤。

## API 設計指南

### 常見 Traits 實作
在適當的地方積極實作常見 traits：
- `Copy`、`Clone`、`Eq`、`PartialEq`、`Ord`、`PartialOrd`、`Hash`、`Debug`、`Display`、`Default`
- 使用標準轉換 traits：`From`、`AsRef`、`AsMut`
- 集合應實作 `FromIterator` 和 `Extend`
- 注意：`Send` 和 `Sync` 在安全時由編譯器自動實作；除非使用 `unsafe` 程式碼，否則避免手動實作

### 型別安全和可預測性
- 使用新型別提供靜態區分
- 參數應透過型別傳達意義；優先使用特定型別而非通用的 `bool` 參數
- 為真正可選的值適當使用 `Option<T>`
- 具有明確接收者的函式應該是方法
- 只有智慧指標應該實作 `Deref` 和 `DerefMut`

### 未來保障
- 使用密封 traits 防止下游實作
- 結構體應該具有私有欄位
- 函式應該驗證其參數
- 所有公開型別必須實作 `Debug`

## 測試和文檔

- 使用 `#[cfg(test)]` 模組和 `#[test]` 註解撰寫全面的單元測試。
- 在測試程式碼旁邊使用測試模組（`mod tests { ... }`）。
- 在 `tests/` 目錄中撰寫整合測試，使用描述性檔案名稱。
- 為每個函式、結構體、列舉和複雜邏輯撰寫清晰簡潔的註解。
- 確保函式具有描述性名稱並包含全面的文檔。
- 使用 rustdoc（`///` 註解）記錄所有公開 API，遵循 [API Guidelines](https://rust-lang.github.io/api-guidelines/)。
- 使用 `#[doc(hidden)]` 從公開文檔中隱藏實作細節。
- 記錄錯誤條件、panic 場景和安全考量。
- 範例應使用 `?` 運算子，而非 `unwrap()` 或已棄用的 `try!` 巨集。

## 專案組織

- 在 `Cargo.toml` 中使用語意版本控制。
- 包含全面的元資料：`description`、`license`、`repository`、`keywords`、`categories`。
- 對可選功能使用功能旗標。
- 使用 `mod.rs` 或命名檔案將程式碼組織到模組中。
- 保持 `main.rs` 或 `lib.rs` 最小化 - 將邏輯移至模組。

## 品質檢查清單

在發布或審查 Rust 程式碼之前，請確保：

### 核心要求
- [ ] **命名**：遵循 RFC 430 命名慣例
- [ ] **Traits**：在適當的地方實作 `Debug`、`Clone`、`PartialEq`
- [ ] **錯誤處理**：使用 `Result<T, E>` 並提供有意義的錯誤型別
- [ ] **文檔**：所有公開項目都有包含範例的 rustdoc 註解
- [ ] **測試**：全面的測試覆蓋率，包括邊緣情況

### 安全性和品質
- [ ] **安全性**：沒有不必要的 `unsafe` 程式碼，適當的錯誤處理
- [ ] **效能**：有效使用迭代器，最小化配置
- [ ] **API 設計**：函式可預測、靈活且型別安全
- [ ] **未來保障**：結構體中的私有欄位，在適當的地方使用密封 traits
- [ ] **工具**：程式碼通過 `cargo fmt`、`cargo clippy` 和 `cargo test`
