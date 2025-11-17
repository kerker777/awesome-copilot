---
mode: 'agent'
tools: ['changes', 'search/codebase', 'edit/editFiles', 'problems']
description: '取得 C# 非同步程式設計的最佳實踐'
---

# C# 非同步程式設計最佳實踐

您的目標是協助我遵循 C# 中非同步程式設計的最佳實踐。

## 命名慣例

- 為所有非同步方法使用「Async」後綴
- 在適用的情況下，將方法名稱與其同步對應項匹配（例如，`GetDataAsync()` 對應 `GetData()`）

## 返回類型

- 當方法返回值時返回 `Task<T>`
- 當方法不返回值時返回 `Task`
- 考慮在高效能場景中使用 `ValueTask<T>` 以減少配置
- 避免為非同步方法返回 `void`，除非用於事件處理程序

## 例外處理

- 在 await 表達式周圍使用 try/catch 區塊
- 避免在非同步方法中吞噬例外
- 在適當的情況下使用 `ConfigureAwait(false)` 以防止程式庫程式碼中的死鎖
- 使用 `Task.FromException()` 傳播例外，而不是在返回非同步 Task 的方法中拋出

## 效能

- 使用 `Task.WhenAll()` 平行執行多個任務
- 使用 `Task.WhenAny()` 實現超時或取得第一個完成的任務
- 在僅傳遞任務結果時避免不必要的 async/await
- 考慮為長時間執行的操作使用取消權杖

## 常見陷阱

- 永遠不要在非同步程式碼中使用 `.Wait()`、`.Result` 或 `.GetAwaiter().GetResult()`
- 避免混合阻塞和非同步程式碼
- 不要建立 async void 方法（除了事件處理程序）
- 始終等待返回 Task 的方法

## 實作模式

- 為長時間執行的操作實作非同步命令模式
- 使用非同步串流 (IAsyncEnumerable<T>) 非同步處理序列
- 考慮為公共 API 使用基於任務的非同步模式 (TAP)

在檢視我的 C# 程式碼時，識別這些問題並建議遵循這些最佳實踐的改進。
