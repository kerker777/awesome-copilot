---
description: 'C# 應用程式開發的代碼編寫規則 by @jgkim999'
applyTo: '**/*.cs'
---

# C# 代碼編寫規則

## 命名規則 (Naming Conventions)

一致的命名規則是代碼可讀性的關鍵。建議遵循 Microsoft 的指導方針。

| 元素 | 命名規則 | 範例 |
|------|-----------|------|
| 介面 | 前綴 'I' + PascalCase | `IAsyncRepository`, `ILogger` |
| 公開(public) 成員 | 帕斯卡命名法 (PascalCase) | `public int MaxCount;`, `public void GetData()` |
| 參數、區域變數 | 駝峰命名法 (camelCase) | `int userCount`, `string customerName` |
| 非公開/內部欄位 | 底線(_) + 駝峰命名法 | `private string _connectionString;` |
| 常數 (const) | 帕斯卡命名法 (PascalCase) | `public const int DefaultTimeout = 5000;` |
| 泛型類型參數 | 前綴 'T' + 描述性名稱 | `TKey`, `TValue`, `TResult` |
| 非同步方法 | 'Async' 後綴 | `GetUserAsync`, `DownloadFileAsync` |

## 代碼格式與可讀性 (Formatting & Readability)

一致的格式使代碼在視覺上容易解析。

| 項目 | 規則 | 說明 |
|------|------|------|
| 縮排 | 使用 4 個空格 | 使用 4 個空格而不是製表符。cs 檔案必須使用 4 個空格。 |
| 括號 | 始終使用大括號 {} | 即使控制語句(if, for, while 等)只有一行，也要始終使用大括號。 |
| 空白行 | 邏輯分離 | 在方法定義、屬性定義、邏輯分離的代碼區塊之間新增空白行。 |
| 陳述式編寫 | 每行一個陳述式 | 每行只編寫一個陳述式。 |
| var 關鍵字 | 僅在類型明確時使用 | 只有當可以從右側清楚地推導出變數的類型時，才使用 var。 |
| 命名空間 | 使用檔案範圍命名空間 | 在 C# 10 及更高版本中，使用檔案範圍命名空間以減少不必要的縮排。 |
| 註解 | 編寫 XML 格式的註解 | 始終為編寫的 class 或函式編寫 XML 格式的註解。 |

## 語言功能使用 (Language Features)

利用最新的 C# 功能使代碼更簡潔、更有效率。

| 功能 | 說明 | 範例/註記 |
|------|------|------|
| 非同步編程 | 對 I/O 受限的工作使用 async/await | `async Task<string> GetDataAsync()` |
| ConfigureAwait | 減少庫代碼中的上下文切換開銷 | `await SomeMethodAsync().ConfigureAwait(false)` |
| LINQ | 查詢和操作集合數據 | `users.Where(u => u.IsActive).ToList()` |
| 表達式型成員 | 簡潔地表達簡單的方法/屬性 | `public string Name => _name;` |
| 可為 Null 的參考類型 | 在編譯時防止 NullReferenceException | `#nullable enable` |
| using 宣告 | 簡潔地處理 IDisposable 物件 | `using var stream = new FileStream(...);` |

## 性能與例外處理 (Performance & Exception Handling)

健全且快速的應用程式的指導方針。

### 例外處理

只捕捉可以處理的具體例外。應避免捕捉一般例外，如 catch (Exception)。

不要將例外用於程式流程控制。例外應僅用於預期外的錯誤情況。

### 性能

重複連接字串時，使用 StringBuilder 而不是 + 運算子。

使用 Entity Framework Core 時，對於唯讀查詢，使用 .AsNoTracking() 以提高性能。

避免不必要的物件配置，特別是在迴圈內。

## 安全性 (Security)

編寫安全代碼的基本原則。

| 安全領域 | 規則 | 說明 |
|------|------|------|
| 輸入驗證 | 驗證所有外部數據 | 不要信任從外部(使用者、API 等)進入的所有數據，始終驗證其有效性。 |
| SQL 注入防止 | 使用參數化查詢 | 始終使用參數化查詢或如 Entity Framework 這樣的 ORM 來防止 SQL 注入攻擊。 |
| 敏感資料保護 | 使用組態管理工具 | 不要在原始碼中硬編碼密碼、連接字串、API 金鑰等，請使用 Secret Manager、Azure Key Vault 等。 |

應將這些規則整合到專案的 .editorconfig 檔案和團隊的代碼審查流程中，以持續維持高品質的代碼。
