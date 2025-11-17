---
applyTo: '*'
description: 'The most comprehensive, practical, and engineer-authored performance optimization instructions for all languages, frameworks, and stacks. Covers frontend, backend, and database best practices with actionable guidance, scenario-based checklists, troubleshooting, and pro tips.'
---

# 效能最佳化實踐指南

## 簡介

效能不只是流行語彙—它是決定用戶喜愛還是放棄某款產品的關鍵差異。我親眼目睹過速度緩慢的應用程式如何激怒用戶、增加雲端費用，甚至導致客戶流失。本指南是一份活躍的實踐集合，匯集了我使用與審視過最有效的真實績效做法，涵蓋前端、後端、資料庫層級以及進階主題。將其作為參考資料、檢查清單，以及打造快速、高效與可擴展軟體的靈感泉源。

---

## 通用原則

- **先測量，再最佳化：** 最佳化前務必進行分析與測量。使用基準測試、分析工具與監控工具來找出真正的瓶頸所在。憑直覺猜測是效能的天敵。
  - *實用貼士：* 使用 Chrome DevTools、Lighthouse、New Relic、Datadog、Py-Spy，或者您使用語言內建的分析工具。
- **為常見情況最佳化：** 著重於最佳化執行頻率最高的程式碼路徑。別浪費時間在罕見的邊界情況上，除非它們非常重要。
- **避免過度最佳化：** 先寫出清楚、可維護的程式碼，只在必要時才最佳化。過度最佳化會讓程式碼變得難以閱讀和維護。
- **最小化資源使用：** 有效率地使用記憶體、CPU、網路與磁碟資源。時時自問：「這能用更少的資源完成嗎？」
- **傾向簡潔設計：** 簡單的演算法與資料結構通常更快，也更容易最佳化。別過度設計。
- **記錄效能假設：** 清楚地註解所有效能關鍵的程式碼或具有非顯而易見的最佳化。未來的維護者（包括你自己）會感謝你。
- **理解你的平台：** 了解你使用的語言、框架與執行時期的效能特性。Python 中快速的東西可能在 JavaScript 中緩慢，反之亦然。
- **將效能測試自動化：** 將效能測試與基準測試整合到你的 CI/CD 流程中。盡早捕捉性能迴歸。
- **設定效能預算：** 為載入時間、記憶體使用量、API 延遲等定義可接受的限制。透過自動化檢查來強制執行。

---

## 前端效能

### 渲染與 DOM
- **最小化 DOM 操作：** 盡可能進行批次更新。頻繁的 DOM 變更成本高昂。
  - *反面模式：* 在迴圈中更新 DOM。應該改為構建文件片段，一次性附加。
- **虛擬 DOM 框架：** 有效率地使用 React、Vue 或類似框架—避免不必要的重新渲染。
  - *React 範例：* 使用 `React.memo`、`useMemo` 與 `useCallback` 來防止不必要的渲染。
- **列表中的鍵：** 始終在列表中使用穩定的鍵以協助虛擬 DOM 比對。避免使用陣列索引作為鍵，除非列表是靜態的。
- **避免內聯樣式：** 內聯樣式可能觸發版面配置抖動。優先使用 CSS 類別。
- **CSS 動畫：** 優先使用 CSS 轉換/動畫而非 JavaScript，以獲得更順暢的 GPU 加速效果。
- **延遲非關鍵渲染：** 使用 `requestIdleCallback` 或類似方法，在瀏覽器空閒時延遲執行工作。

### 資源最佳化
- **影像壓縮：** 使用 ImageOptim、Squoosh 或 TinyPNG 等工具。偏好使用現代格式（WebP、AVIF）進行網頁傳遞。
- **SVG 用於圖標：** SVG 具有良好的縮放性，對於簡單圖形通常比 PNG 更小。
- **最小化與打包：** 使用 Webpack、Rollup 或 esbuild 來打包和最小化 JS/CSS。啟用樹搖動以移除死亡程式碼。
- **快取標頭：** 為靜態資源設定長期快取標頭。使用快取破壞進行更新。
- **延遲載入：** 對影像使用 `loading="lazy"`，對 JS 模組/元件使用動態導入。
- **字型最佳化：** 只使用你需要的字元集。子集化字型並使用 `font-display: swap`。

### 網路最佳化
- **減少 HTTP 請求：** 合併檔案、使用影像雪碧圖，並內聯重要 CSS。
- **HTTP/2 和 HTTP/3：** 啟用這些協定以進行多路複用和降低延遲。
- **客戶端快取：** 使用 Service Worker、IndexedDB 與 localStorage 實現離線與重複訪問。
- **CDN：** 從靠近用戶的 CDN 提供靜態資源。使用多個 CDN 作為冗餘。
- **延遲/非同步指令碼：** 對非關鍵 JS 使用 `defer` 或 `async` 以避免阻擋渲染。
- **預載與預取：** 對關鍵資源使用 `<link rel="preload">` 與 `<link rel="prefetch">`。

### JavaScript 效能
- **避免阻擋主執行緒：** 將重型計算卸載到 Web Worker。
- **防抖/節流事件：** 對滾動、調整大小與輸入事件，使用防抖/節流來限制處理程式頻率。
- **記憶體洩漏：** 清理事件監聽器、時間間隔與 DOM 參考。使用瀏覽器開發工具檢查已分離節點。
- **有效率的資料結構：** 使用 Map/Set 進行查找，TypedArray 用於數值資料。
- **避免全域變數：** 全域變數可能導致記憶體洩漏和不可預測的效能。
- **避免深層物件複製：** 只在必要時使用淺層複製或 lodash 的 `cloneDeep` 等函式庫。

### 無障礙設計與效能
- **無障礙元件：** 確保 ARIA 更新不會過度。使用語意 HTML 同時提升無障礙性與效能。
- **螢幕閱讀器效能：** 避免快速的 DOM 更新，這可能會造成輔助技術超載。

### 框架特定貼士
#### React
- 使用 `React.memo`、`useMemo` 與 `useCallback` 避免不必要的渲染。
- 分割大型元件並使用程式碼分割（`React.lazy`、`Suspense`）。
- 避免在 render 中使用匿名函式；每次渲染時都會建立新的參考。
- 使用 `ErrorBoundary` 優雅地捕捉和處理錯誤。
- 使用 React DevTools Profiler 進行分析。

#### Angular
- 對不需要頻繁更新的元件使用 OnPush 變更檢測。
- 避免在樣板中使用複雜表達式；將邏輯移至元件類別。
- 在 `ngFor` 中使用 `trackBy` 以實現有效率的列表渲染。
- 使用 Angular Router 延遲載入模組和元件。
- 使用 Angular DevTools 進行分析。

#### Vue
- 在樣板中優先使用計算屬性而非方法以進行快取。
- 適當使用 `v-show` 和 `v-if`（`v-show` 更適合頻繁切換可見性）。
- 使用 Vue Router 延遲載入元件和路由。
- 使用 Vue Devtools 進行分析。

### 常見前端陷阱
- 在初始頁面載入時載入大型 JS 套件。
- 未壓縮影像或使用過時格式。
- 未能清理事件監聽器，導致記憶體洩漏。
- 過度使用第三方函式庫進行簡單任務。
- 忽視行動效能（在實體設備上測試！）。

### 前端故障排除
- 使用 Chrome DevTools 的效能標籤來記錄和分析慢幀。
- 使用 Lighthouse 審計效能並取得可行建議。
- 使用 WebPageTest 進行實際負載測試。
- 監控 Core Web Vitals（LCP、FID、CLS）以取得以用戶為中心的指標。

---

## 後端效能

### 演算法與資料結構最佳化
- **選擇適當的資料結構：** 陣列用於順序存取、雜湊表用於快速查找、樹用於階層式資料等。
- **有效率的演算法：** 在適當的地方使用二元搜尋、快速排序或雜湊型演算法。
- **避免 O(n^2) 或更差的情況：** 分析巢狀迴圈和遞迴呼叫。重構以降低複雜度。
- **批次處理：** 以批次方式處理資料以降低額外開銷（例如大量資料庫插入）。
- **串流：** 對大型資料集使用串流 API 以避免將所有內容載入記憶體。

### 並行與平行處理
- **非同步 I/O：** 使用 async/await、回呼或事件迴圈以避免執行緒阻擋。
- **執行緒/工作執行緒集區：** 使用集區來管理並行並避免資源耗盡。
- **避免競態條件：** 在需要時使用鎖定、信號量或原子操作。
- **批次操作：** 批次化網路/資料庫呼叫以減少往返次數。
- **背壓：** 在佇列和管道中實現背壓以避免超載。

### 快取
- **快取昂貴的計算：** 對熱資料使用記憶體內快取（Redis、Memcached）。
- **快取失效：** 使用時間型（TTL）、事件型或手動失效。過時快取比沒有快取更糟。
- **分散式快取：** 對多伺服器設置，使用分散式快取並注意一致性問題。
- **快取雷鳴羊群防護：** 使用鎖定或請求合併以防止雷鳴羊群問題。
- **別快取所有東西：** 有些資料過於易變或敏感而無法快取。

### API 與網路
- **最小化承載量：** 使用 JSON、壓縮回應（gzip、Brotli），避免傳送不必要的資料。
- **分頁：** 始終為大型結果集分頁。對實時資料使用游標。
- **速率限制：** 保護 API 免受濫用和超載。
- **連線池：** 重複使用資料庫與外部服務的連線。
- **協定選擇：** 使用 HTTP/2、gRPC 或 WebSocket 進行高吞吐量、低延遲通訊。

### 日誌記錄與監控
- **最小化關鍵路徑中的日誌記錄：** 過度的日誌記錄會拖慢關鍵程式碼。
- **結構化日誌：** 使用 JSON 或鍵值日誌以便於分析。
- **監控一切：** 延遲、吞吐量、錯誤率、資源使用。使用 Prometheus、Grafana、Datadog 或類似。
- **警示：** 為效能迴歸和資源耗盡設定警示。

### 語言/框架特定貼士
#### Node.js
- 使用非同步 API；避免阻擋事件迴圈（例如在生產環境中千萬不要使用 `fs.readFileSync`）。
- 使用叢集或工作執行緒進行 CPU 綁定任務。
- 限制並行開啟連線以避免資源耗盡。
- 使用串流進行大型檔案或網路資料處理。
- 使用 `clinic.js`、`node --inspect` 或 Chrome DevTools 進行分析。

#### Python
- 使用內建資料結構（`dict`、`set`、`deque`）以提高速度。
- 使用 `cProfile`、`line_profiler` 或 `Py-Spy` 進行分析。
- 使用 `multiprocessing` 或 `asyncio` 進行平行處理。
- 避免 CPU 綁定程式碼中的 GIL 瓶頸；使用 C 延伸或子程式。
- 使用 `lru_cache` 進行記憶化。

#### Java
- 使用有效率的集合（`ArrayList`、`HashMap` 等）。
- 使用 VisualVM、JProfiler 或 YourKit 進行分析。
- 使用執行緒集區（`Executors`）進行並行處理。
- 為堆和垃圾回收調整 JVM 選項（`-Xmx`、`-Xms`、`-XX:+UseG1GC`）。
- 使用 `CompletableFuture` 進行非同步程式設計。

#### .NET
- 對 I/O 綁定操作使用 `async/await`。
- 使用 `Span<T>` 與 `Memory<T>` 以實現有效率的記憶體存取。
- 使用 dotTrace、Visual Studio Profiler 或 PerfView 進行分析。
- 在適當的地方集區物件和連線。
- 使用 `IAsyncEnumerable<T>` 進行資料串流。

### 常見後端陷阱
- 網頁伺服器中的同步/阻擋 I/O。
- 未對資料庫使用連線池。
- 過度快取或快取敏感/易變資料。
- 忽視非同步程式碼中的錯誤處理。
- 未監控或警示效能迴歸。

### 後端故障排除
- 使用火焰圖視覺化 CPU 使用情況。
- 使用分散式追蹤（OpenTelemetry、Jaeger、Zipkin）追蹤跨服務的請求延遲。
- 使用堆傾出和記憶體分析工具尋找洩漏。
- 記錄慢查詢和 API 呼叫以供分析。

---

## 資料庫效能

### 查詢最佳化
- **索引：** 在經常被查詢、過濾或連接的欄位上使用索引。監控索引使用情況並刪除未使用的索引。
- **避免 SELECT *：** 只選擇需要的欄位。減少 I/O 和記憶體使用。
- **參數化查詢：** 防止 SQL 注入並改進查詢計畫快取。
- **查詢計畫：** 分析和最佳化查詢執行計畫。在 SQL 資料庫中使用 `EXPLAIN`。
- **避免 N+1 查詢：** 使用連接或批次查詢以避免迴圈中的重複查詢。
- **限制結果集：** 對大型表使用 `LIMIT`/`OFFSET` 或游標。

### 模式設計
- **正規化：** 正規化以減少冗餘，但如需要可對讀取密集的工作負載進行反正規化。
- **資料型別：** 使用最有效率的資料型別並設定適當的限制。
- **分割：** 對大型表進行分割以提升可擴展性和可管理性。
- **封存：** 定期封存或清除舊資料以保持表格小型且快速。
- **外鍵：** 將其用於資料完整性，但在高寫入情況下要注意效能權衡。

### 交易
- **短交易：** 盡可能保持交易簡短以減少鎖定競爭。
- **隔離層級：** 使用滿足一致性需求的最低隔離層級。
- **避免長時間執行的交易：** 它們可能阻擋其他操作並增加死鎖。

### 快取與複製
- **讀取複本：** 用於擴展讀取密集的工作負載。監控複製延遲。
- **快取查詢結果：** 對經常存取的查詢使用 Redis 或 Memcached。
- **寫入穿透/寫入後延：** 為一致性需求選擇正確的策略。
- **分片：** 跨多個伺服器分散資料以實現可擴展性。

### NoSQL 資料庫
- **為存取模式設計：** 為你需要的查詢建模資料。
- **避免熱點分割：** 均勻分散寫入/讀取。
- **無界成長：** 留意無界陣列或文件。
- **分片和複製：** 用於可擴展性和可用性。
- **一致性模型：** 理解最終一致性與強一致性，並作適當選擇。

### 常見資料庫陷阱
- 遺漏或未使用的索引。
- 生產查詢中的 SELECT *。
- 未監控慢查詢。
- 忽視複製延遲。
- 未封存舊資料。

### 資料庫故障排除
- 使用慢查詢日誌識別瓶頸。
- 使用 `EXPLAIN` 分析查詢計畫。
- 監控快取命中/未命中比率。
- 使用資料庫特定的監控工具（pg_stat_statements、MySQL Performance Schema）。

---

## 效能程式碼審查檢查清單

- [ ] 是否存在明顯的演算法低效率（O(n^2) 或更差）？
- [ ] 資料結構是否適合其用途？
- [ ] 是否存在不必要的計算或重複工作？
- [ ] 快取是否用在適當的地方，失效處理是否正確？
- [ ] 資料庫查詢是否最佳化、索引化，且無 N+1 問題？
- [ ] 大型承載量是否已分頁、串流或分塊？
- [ ] 是否存在記憶體洩漏或無界資源使用？
- [ ] 網路請求是否已最小化、批次化並在失敗時重試？
- [ ] 資源是否已最佳化、壓縮並有效率地提供？
- [ ] 關鍵路徑中是否有任何阻擋操作？
- [ ] 關鍵路徑中的日誌記錄是否已最小化並結構化？
- [ ] 效能關鍵程式碼路徑是否已記錄和測試？
- [ ] 效能敏感程式碼是否有自動化測試或基準測試？
- [ ] 效能迴歸是否有警示？
- [ ] 是否存在反面模式（例如 SELECT *、阻擋 I/O、全域變數）？

---

## 進階主題

### 分析與基準測試
- **分析工具：** 使用語言特定的分析工具（Chrome DevTools、Py-Spy、VisualVM、dotTrace 等）識別瓶頸。
- **微基準測試：** 為關鍵程式碼路徑編寫微基準測試。使用 `benchmark.js`、`pytest-benchmark` 或 Java 的 JMH。
- **A/B 測試：** 使用 A/B 測試或金絲雀發佈來測量最佳化的實際影響。
- **持續效能測試：** 將效能測試整合到 CI/CD。使用 k6、Gatling 或 Locust 等工具。

### 記憶體管理
- **資源清理：** 始終及時釋放資源（檔案、socket、資料庫連線）。
- **物件集區：** 用於經常建立/銷毀的物件（例如資料庫連線、執行緒）。
- **堆監控：** 監控堆使用情況和垃圾回收。為工作負載調整 GC 設定。
- **記憶體洩漏：** 使用洩漏偵測工具（Valgrind、LeakCanary、Chrome DevTools）。

### 可擴展性
- **水平擴展：** 設計無狀態服務、使用分片/分割與負載平衡器。
- **自動擴展：** 使用雲端自動擴展群組並設定合理的閾值。
- **瓶頸分析：** 識別並解決單點故障。
- **分散式系統：** 使用冪等操作、重試與斷路器。

### 安全性與效能
- **有效率的加密：** 使用硬體加速且維護良好的密碼學函式庫。
- **驗證：** 有效率地驗證輸入；避免在關鍵路徑中使用正規表達式。
- **速率限制：** 防止 DoS 攻擊而不傷害合法使用者。

### 行動效能
- **啟動時間：** 延遲載入功能、延遲繁重工作，並最小化初始套件大小。
- **影像/資源最佳化：** 使用回應式影像並為行動頻寬壓縮資源。
- **有效率的儲存：** 使用 SQLite、Realm 或平台最佳化的儲存。
- **分析：** 使用 Android Profiler、Instruments（iOS）或 Firebase 效能監控。

### 雲端與無伺服器
- **冷啟動：** 最小化相依性並保持函式溫暖。
- **資源配置：** 為無伺服器函式調整記憶體/CPU。
- **受管服務：** 使用受管快取、佇列與資料庫以實現可擴展性。
- **成本最佳化：** 監控並最佳化雲端成本作為效能指標。

---

## 實踐範例

### 範例 1：在 JavaScript 中防抖使用者輸入
```javascript
// BAD: Triggers API call on every keystroke
input.addEventListener('input', (e) => {
  fetch(`/search?q=${e.target.value}`);
});

// GOOD: Debounce API calls
let timeout;
input.addEventListener('input', (e) => {
  clearTimeout(timeout);
  timeout = setTimeout(() => {
    fetch(`/search?q=${e.target.value}`);
  }, 300);
});
```

### 範例 2：有效率的 SQL 查詢
```sql
-- BAD: Selects all columns and does not use an index
SELECT * FROM users WHERE email = 'user@example.com';

-- GOOD: Selects only needed columns and uses an index
SELECT id, name FROM users WHERE email = 'user@example.com';
```

### 範例 3：在 Python 中快取昂貴的計算
```python
# BAD: Recomputes result every time
result = expensive_function(x)

# GOOD: Cache result
from functools import lru_cache

@lru_cache(maxsize=128)
def expensive_function(x):
    ...
result = expensive_function(x)
```

### 範例 4：在 HTML 中延遲載入影像
```html
<!-- BAD: Loads all images immediately -->
<img src="large-image.jpg" />

<!-- GOOD: Lazy loads images -->
<img src="large-image.jpg" loading="lazy" />
```

### 範例 5：在 Node.js 中進行非同步 I/O
```javascript
// BAD: Blocking file read
const data = fs.readFileSync('file.txt');

// GOOD: Non-blocking file read
fs.readFile('file.txt', (err, data) => {
  if (err) throw err;
  // process data
});
```

### 範例 6：分析 Python 函式
```python
import cProfile
import pstats

def slow_function():
    ...

cProfile.run('slow_function()', 'profile.stats')
p = pstats.Stats('profile.stats')
p.sort_stats('cumulative').print_stats(10)
```

### 範例 7：在 Node.js 中使用 Redis 進行快取
```javascript
const redis = require('redis');
const client = redis.createClient();

function getCachedData(key, fetchFunction) {
  return new Promise((resolve, reject) => {
    client.get(key, (err, data) => {
      if (data) return resolve(JSON.parse(data));
      fetchFunction().then(result => {
        client.setex(key, 3600, JSON.stringify(result));
        resolve(result);
      });
    });
  });
}
```

---

## 參考資料與進一步閱讀
- [Google Web Fundamentals: Performance](https://web.dev/performance/)
- [MDN Web Docs: Performance](https://developer.mozilla.org/en-US/docs/Web/Performance)
- [OWASP: Performance Testing](https://owasp.org/www-project-performance-testing/)
- [Microsoft Performance Best Practices](https://learn.microsoft.com/en-us/azure/architecture/best-practices/performance)
- [PostgreSQL Performance Optimization](https://wiki.postgresql.org/wiki/Performance_Optimization)
- [MySQL Performance Tuning](https://dev.mysql.com/doc/refman/8.0/en/optimization.html)
- [Node.js Performance Best Practices](https://nodejs.org/en/docs/guides/simple-profiling/)
- [Python Performance Tips](https://docs.python.org/3/library/profile.html)
- [Java Performance Tuning](https://www.oracle.com/java/technologies/javase/performance.html)
- [.NET Performance Guide](https://learn.microsoft.com/en-us/dotnet/standard/performance/)
- [WebPageTest](https://www.webpagetest.org/)
- [Lighthouse](https://developers.google.com/web/tools/lighthouse)
- [Prometheus](https://prometheus.io/)
- [Grafana](https://grafana.com/)
- [k6 Load Testing](https://k6.io/)
- [Gatling](https://gatling.io/)
- [Locust](https://locust.io/)
- [OpenTelemetry](https://opentelemetry.io/)
- [Jaeger](https://www.jaegertracing.io/)
- [Zipkin](https://zipkin.io/)

---

## 結論

效能最佳化是一個持續的過程。始終測量、分析和迭代。使用這些最佳實踐、檢查清單與故障排除貼士來指導你的開發和程式碼審查，以打造高效能、可擴展且有效率的軟體。若你有新的建議或經驗教訓，請加入此處—讓我們持續豐富本指南！

---

<!-- End of Performance Optimization Instructions --> 
