---
description: 'Clojure 特定的程式碼模式、內聯 def 使用方式、程式碼區塊範本，以及 Clojure 開發的命名空間處理。'
applyTo: '**/*.{clj,cljs,cljc,bb,edn.mdx?}'
---

# Clojure 開發指引

## 程式碼評估工具使用

「使用 repl」代表使用 Calva Backseat Driver 的 **Evaluate Clojure Code** 工具。它會連接到與使用者透過 Calva 連接的相同 REPL。

- 始終保持在 Calva 的 REPL 內，而非從終端機啟動第二個 REPL。
- 如果沒有 REPL 連線，請要求使用者連接 REPL，而非嘗試自行啟動和連接。

### REPL 工具呼叫中的 JSON 字串
在呼叫 REPL 工具時，不要過度跳脫 JSON 參數。

```json
{
  "namespace": "<current-namespace>",
  "replSessionKey": "cljs",
  "code": "(def foo \"something something\")"
}
```

## `defn` 中的文件字串
文件字串應該緊接在函式名稱之後、參數向量之前。

```clojure
(defn my-function
  "This function does something."
  [arg1 arg2]
  ;; function body
  )
```

- 在使用函式之前定義它們——優先使用排序而非 `declare`，除非確實必要。

## 互動式程式設計（又稱 REPL 驅動開發）

### 對齊資料結構元素以進行括號平衡
**始終在所有資料結構中垂直對齊多行元素：向量、對映、列表、集合、所有程式碼（因為 Clojure 程式碼就是資料）。錯位會導致括號平衡器錯誤地閉合括號，建立無效的形式。**

```clojure
;; ❌ 錯誤 - 向量元素錯位
(select-keys m [:key-a
                :key-b
               :key-c])  ; 錯位 → 錯誤的 ] 位置

;; ✅ 正確 - 向量元素對齊
(select-keys m [:key-a
                :key-b
                :key-c])  ; 適當對齊 → 正確的 ] 位置

;; ❌ 錯誤 - 對映條目錯位
{:name "Alice"
 :age 30
:city "Oslo"}  ; 錯位 → 錯誤的 } 位置

;; ✅ 正確 - 對映條目對齊
{:name "Alice"
 :age 30
 :city "Oslo"}  ; 適當對齊 → 正確的 } 位置
```

**關鍵**：括號平衡器依賴一致的縮排來確定結構。

### REPL 依賴管理
在 REPL 會話期間使用 `clojure.repl.deps/add-libs` 進行動態依賴載入。

```clojure
(require '[clojure.repl.deps :refer [add-libs]])
(add-libs '{dk.ative/docjure {:mvn/version "1.15.0"}})
```

- 動態依賴載入需要 Clojure 1.12 或更高版本
- 非常適合程式庫探索和原型設計

### 檢查 Clojure 版本

```clojure
*clojure-version*
;; => {:major 1, :minor 12, :incremental 1, :qualifier nil}
```

### REPL 可用性規範

**當 REPL 不可用時，絕不編輯程式碼檔案。** 當 REPL 評估回傳錯誤表示 REPL 不可用時，立即停止並通知使用者。讓使用者在繼續之前恢復 REPL。

#### 為什麼這很重要
- **互動式程式設計需要可運作的 REPL** - 沒有評估就無法驗證行為
- **猜測會產生錯誤** - 未經測試的程式碼變更會引入錯誤

## 結構化編輯和 REPL 優先習慣
- 在觸及檔案之前，在 REPL 中開發變更。
- 編輯 Clojure 檔案時，始終使用結構化編輯工具，如 **Insert Top Level Form**、**Replace Top Level Form**、**Create Clojure File** 和 **Append Code**，並始終先閱讀其說明。

### 建立新檔案
- 使用 **Create Clojure File** 工具並提供初始內容
- 遵循 Clojure 命名規則：命名空間使用 kebab-case，檔案路徑使用匹配的 snake_case（例如，`my.project.ns` → `my/project/ns.clj`）。

### 重新載入命名空間
編輯檔案後，在 REPL 中重新載入已編輯的命名空間，以便更新的定義生效。

```clojure
(require 'my.namespace :reload)
```

## 評估前的程式碼縮排
一致的縮排對於幫助括號平衡器至關重要。

```clojure
;; ❌
(defn my-function [x]
(+ x 2))

;; ✅
(defn my-function [x]
  (+ x 2))
```

## 縮排偏好

將條件和主體保持在不同行：

```clojure
(when limit
  (println "Limit set to:" limit))
```

將 `and` 和 `or` 參數保持在不同行：

```clojure
(if (and condition-a
         condition-b)
  this
  that)
```

## 內聯 Def 模式

優先使用內聯 def 除錯而非 println/console.log。

### 用於除錯的內聯 `def`
- 內聯 `def` 綁定在 REPL 工作期間保持中間狀態可檢查。
- 當內聯綁定繼續幫助探索時，保留它們。

```clojure
(defn process-instructions [instructions]
  (def instructions instructions)
  (let [grouped (group-by :status instructions)]
    grouped))
```

- 實時檢查保持可用。
- 除錯週期保持快速。
- 迭代開發保持順暢。

您也可以在聊天中向使用者展示程式碼時使用「內聯 def」，使使用者能夠輕鬆地在程式碼區塊中進行實驗。使用者可以使用 Calva 直接在您的程式碼區塊中評估程式碼。（但使用者無法在那裡編輯程式碼。）

## 回傳值 > 列印副作用

優先使用 REPL 和評估的回傳值，而非將內容列印到 stdout。

## 從 `stdin` 讀取
- 當 Clojure 程式碼使用 `(read-line)` 時，它會透過 VS Code 提示使用者。
- 避免在 Babashka 的 nREPL 中使用 stdin 讀取，因為它缺乏 stdin 支援。
- 如果 REPL 阻塞，請要求使用者重新啟動。

## 資料結構偏好

我們試圖保持資料結構盡可能扁平，大量依賴命名空間關鍵字並優化易於解構。通常在應用程式中我們使用命名空間關鍵字，最常見的是「合成」命名空間。

直接在參數列表中解構鍵。

```clojure
(defn handle-user-request
  [{:user/keys [id name email]
    :request/keys [method path headers]
    :config/keys [timeout debug?]}]
  (when debug?
    (println "Processing" method path "for" name)))
```

在許多好處中，這使函式簽名保持透明。

### 避免遮蔽內建函式
必要時重新命名傳入的鍵，以避免隱藏核心函式。

```clojure
(defn create-item
  [{:prompt-sync.file/keys [path uri]
    file-name :prompt-sync.file/name
    file-type :prompt-sync.file/type}]
  #js {:label file-name
       :type file-type})
```

要保持自由的常見符號：
- `class`
- `count`
- `empty?`
- `filter`
- `first`
- `get`
- `key`
- `keyword`
- `map`
- `merge`
- `name`
- `reduce`
- `rest`
- `set`
- `str`
- `symbol`
- `type`
- `update`

## 避免不必要的包裝函式
除非名稱真正釐清組合，否則不要包裝核心函式。

```clojure
(remove (set exclusions) items) ; 包裝函式不會使這更清楚
```

## 用於文件的豐富註解形式 (RCF)

豐富註解形式 `(comment ...)` 與直接 REPL 評估有不同的目的。在檔案編輯中使用 RCF 來 **記錄您已在 REPL 中驗證的函式的使用模式和範例**。

### 何時使用 RCF
- **在 REPL 驗證之後** - 在檔案中記錄有效的範例
- **使用文件** - 展示函式的預期使用方式
- **探索保存** - 在程式碼庫中保留有用的 REPL 發現
- **範例場景** - 演示邊緣情況和典型用法

### RCF 模式
RCF = 豐富註解形式。

當載入檔案時，RCF 中的程式碼不會被評估，這使它們非常適合記錄範例用法，因為人類可以隨意評估其中的程式碼。

```clojure
(defn process-user-data
  "Processes user data with validation"
  [{:user/keys [name email] :as user-data}]
  ;; implementation here
  )

(comment
  ;; Basic usage
  (process-user-data {:user/name "John" :user/email "john@example.com"})

  ;; Edge case - missing email
  (process-user-data {:user/name "Jane"})

  ;; Integration example
  (->> users
       (map process-user-data)
       (filter :valid?))

  :rcf) ; Optional marker for end of comment block
```

### RCF vs REPL 工具使用
```clojure
;; In chat - show direct REPL evaluation:
(in-ns 'my.namespace)
(let [test-data {:user/name "example"}]
  (process-user-data test-data))

;; In files - document with RCF:
(comment
  (process-user-data {:user/name "example"})
  :rcf)
```

## 測試

### 從 REPL 執行測試
重新載入目標命名空間並從 REPL 執行測試以獲得即時回饋。

```clojure
(require '[my.project.some-test] :reload)
(clojure.test/run-tests 'my.project.some-test)
(cljs.test/run-tests 'my.project.some-test)
```

- 更緊密的 REPL 整合。
- 集中執行。
- 更簡單的除錯。
- 直接存取測試資料。

在調查失敗時，優先從測試命名空間內執行個別測試變數。

### 使用 REPL 優先的 TDD 工作流程
在編輯檔案之前，使用真實資料進行迭代。

```clojure
(def sample-text "line 1\nline 2\nline 3\nline 4\nline 5")

(defn format-line-number [n padding marker-len]
  (let [num-str (str n)
        total-padding (- padding marker-len)]
    (str (apply str (repeat (- total-padding (count num-str)) " "))
         num-str)))

(deftest line-number-formatting
  (is (= "  5" (editor-util/format-line-number 5 3 0))
      "Single digit with padding 3, no marker space")
  (is (= " 42" (editor-util/format-line-number 42 3 0))
      "Double digit with padding 3, no marker space"))
```

#### 好處
- 在提交變更之前驗證行為
- 透過即時回饋進行增量開發
- 捕獲已知良好行為的測試
- 從失敗的測試開始新工作以鎖定意圖

### 測試命名和訊息傳遞
保持 `deftest` 名稱具描述性（區域/事物樣式），不使用冗餘的 `-test` 後綴。

### 測試斷言訊息風格
將期望訊息直接附加到 `is`，僅在分組多個相關斷言時使用 `testing` 區塊。

```clojure
(deftest line-marker-formatting
  (is (= "→" (editor-util/format-line-marker true))
      "Target line gets marker")
  (is (= "" (editor-util/format-line-marker false))
      "Non-target gets empty string"))

(deftest context-line-extraction
  (testing "Centered context extraction"
    (let [result (editor-util/get-context-lines "line 1\nline 2\nline 3" 2 3)]
      (is (= 3 (count (str/split-lines result)))
          "Should have 3 lines")
      (is (str/includes? result "→")
          "Should have marker"))))
```

指導方針：
- 保持斷言訊息對期望明確。
- 使用 `testing` 來分組相關檢查。
- 維護 kebab-case 名稱，如 `line-marker-formatting` 或 `context-line-extraction`。

## 快樂的互動式程式設計

記住在工作中優先使用 REPL。請記住，使用者看不到您評估的內容，也看不到結果。在聊天中與使用者溝通您評估的內容和得到的回應。
