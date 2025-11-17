---
mode: "agent"
description: "Write a coding standards document for a project using the coding styles from the file(s) and/or folder(s) passed as arguments in the prompt."
tools: ['createFile', 'editFiles', 'fetch', 'githubRepo', 'search', 'testFailure']
---

# 從文件撰寫編碼標準

使用文件的現有語法來建立專案的標準和風格指南。如果傳入多個文件或資料夾，應逐一遍歷資料夾中的每個文件，將文件的資料附加到臨時記憶體或文件，完成後將臨時資料作為單一實例使用，如同以該文件名稱作為基礎來制定標準和風格指南。

## 規則和設定

以下是一組準設定的 `boolean` 和 `string[]` 變數。每個變數在處理 `true` 或其他值的條件位於二級標題 `## 變數和參數設定條件` 下方。

Prompt 的參數有文字定義。其中有一個必要參數 **`${fileName}`**，以及多個選用參數 **`${folderName}`**、**`${instructions}`** 和任何 **`[configVariableAsParameter]`**。

### 設定變數

* addStandardsTest = false;
* addToREADME = false;
* addToREADMEInsertions = ["atBegin", "middle", "beforeEnd", "bestFitUsingContext"];
  - 預設為 **beforeEnd**。
* createNewFile = true;
* fetchStyleURL = true;
* findInconsistencies = true;
* fixInconsistencies = true;
* newFileName = ["CONTRIBUTING.md", "STYLE.md", "CODE_OF_CONDUCT.md", "CODING_STANDARDS.md", "DEVELOPING.md", "CONTRIBUTION_GUIDE.md", "GUIDELINES.md", "PROJECT_STANDARDS.md", "BEST_PRACTICES.md", "HACKING.md"];
  - 針對 `${newFileName}` 中的每個文件，如果文件不存在，則使用該文件名稱並 `break`，否則繼續至下一個文件名稱。
* outputSpecToPrompt = false;
* useTemplate = "verbose"; // or "v"
  - 可能的值為 `[["v", "verbose"], ["m", "minimal"], ["b", "best fit"], ["custom"]]`。
  - 在撰寫編碼標準時，選擇 Prompt 文件下方二級標題 `## 編碼標準範本` 下的兩個範本之一作為指導，或使用更適合的其他組合方式。
  - 如果為 **custom**，則按需求應用。

### 設定變數作為 Prompt 參數

如果將任何變數名稱作為參數傳入 Prompt，或傳入類似但明確相關的文字值，則使用傳入 Prompt 的值覆蓋預設變數值。

### Prompt 參數

* **fileName** = 將被分析的文件名稱，分析內容包括：縮排、變數命名、註解、條件程序、函式程序，以及與該文件編碼語言相關的其他語法資料。
* folderName = 用於從資料夾中的多個文件提取資料的資料夾名稱，這些資料將彙總成一個資料集，分析內容包括：縮排、變數命名、註解、條件程序、函式程序，以及與這些文件編碼語言相關的其他語法資料。
* instructions = 將為特殊情況提供的額外指示、規則和程序。
* [configVariableAsParameter] = 如果傳入，將覆蓋設定變數的預設狀態。例如：
  - useTemplate = 如果傳入，將覆蓋設定 `${useTemplate}` 的預設值。值為 `[["v", "verbose"], ["m", "minimal"], ["b", "best fit"]]`。

#### 必要和選用參數

* **fileName** - 必要
* folderName - *選用*
* instructions - *選用*
* [configVariableAsParameter] - *選用*

## 變數和參數設定條件

### `${fileName}.length > 1 || ${folderName} != undefined`

* 如果為真，將 `${fixInconsistencies}` 切換為假。

### `${addToREADME} == true`

* 將編碼標準插入 `README.md` 而不是輸出至 Prompt 或建立新文件。
* 如果為真，將 `${createNewFile}` 和 `${outputSpecToPrompt}` 都切換為假。

### `${addToREADMEInsertions} == "atBegin"`

* 如果 `${addToREADME}` 為真，則在 `README.md` 文件的**開始處**插入編碼標準資料，位於標題之後。

### `${addToREADMEInsertions} == "middle"`

* 如果 `${addToREADME}` 為真，則在 `README.md` 文件的**中間**插入編碼標準資料，將標準標題標題更改為與 `README.md` 組成相符。

### `${addToREADMEInsertions} == "beforeEnd"`

* 如果 `${addToREADME}` 為真，則在 `README.md` 文件的**結尾**插入編碼標準資料，在最後一個字符後插入新行，然後在新行上插入資料。

### `${addToREADMEInsertions} == "bestFitUsingContext"`

* 如果 `${addToREADME}` 為真，則在 `README.md` 文件中**最適合的行**插入編碼標準資料，考慮 `README.md` 組成的內容和資料流。

### `${addStandardsTest} == true`

* 編碼標準文件完成後，寫入測試文件以確保傳入該文件或文件的文件遵守編碼標準。

### `${createNewFile} == true`

* 使用來自 `${newFileName}` 的值或可能值之一建立新文件。
* 如果為真，將 `${outputSpecToPrompt}` 和 `${addToREADME}` 都切換為假。

### `${fetchStyleURL} == true`

* 另外使用從二級標題 `### 取得連結` 下嵌入的連結取得的資料作為建立標準、規格和新文件、Prompt 或 `README.md` 風格資料的內容。
* 對於 `### 取得連結` 中的每個相關項目，運行 `#fetch ${item}`。

### `${findInconsistencies} == true`

* 評估與縮排、換行符、註解、條件和函式嵌套、引號包裝器（例如字串的 `'` 或 `"`）相關的語法，並分類。
* 對於每個分類，進行計數，如果一項與計數的多數不符，則提交到臨時記憶體。
* 根據 `${fixInconsistencies}` 的狀態，編輯並修復低計數分類以符合多數對應語法資料，或將存儲在臨時記憶體中的不一致輸出至 Prompt。

### `${fixInconsistencies} == true`

* 編輯並修復語法資料的低計數分類以符合多數對應語法資料，使用存儲在臨時記憶體中的不一致。

### `typeof ${newFileName} == "string"`

* 如果特別定義為 `string`，則使用來自 `${newFileName}` 的值建立新文件。

### `typeof ${newFileName} != "string"`

* 如果**不是**特別定義為 `string`，而是 `object` 或陣列，則透過應用此規則使用來自 `${newFileName}` 的值建立新文件：
  - 對於 `${newFileName}` 中的每個文件名稱，如果文件不存在，則使用該文件名稱並 `break`，否則繼續至下一個。

### `${outputSpecToPrompt} == true`

* 將編碼標準輸出至 Prompt 而不是建立文件或新增至 README。
* 如果為真，將 `${createNewFile}` 和 `${addToREADME}` 都切換為假。

### `${useTemplate} == "v" || ${useTemplate} == "verbose"`

* 在撰寫編碼標準資料時，使用二級標題 `### "v", "verbose"` 下的資料作為指導範本。

### `${useTemplate} == "m" || ${useTemplate} == "minimal"`

* 在撰寫編碼標準資料時，使用二級標題 `### "m", "minimal"` 下的資料作為指導範本。

### `${useTemplate} == "b" || ${useTemplate} == "best"`

* 根據從 `${fileName}` 提取的資料，使用二級標題 `### "v", "verbose"` 或 `### "m", "minimal"` 下的資料之一，並使用最適合的作為在撰寫編碼標準資料時的指導範本。

### `${useTemplate} == "custom" || ${useTemplate} == "<ANY_NAME>"`

* 使用作為在撰寫編碼標準資料時的指導範本的自訂提示、指示、範本或其他傳入資料。

## **如果** `${fetchStyleURL} == true`

根據程式設計語言，對於以下清單中的每個連結，如果程式設計語言為 `${fileName} == [<Language> Style Guide]`，則運行 `#fetch (URL)`。

### 取得連結

- [C Style Guide](https://users.ece.cmu.edu/~eno/coding/CCodingStandard.html)
- [C# Style Guide](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions)
- [C++ Style Guide](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines)
- [Go Style Guide](https://github.com/golang-standards/project-layout)
- [Java Style Guide](https://coderanch.com/wiki/718799/Style)
- [AngularJS App Style Guide](https://github.com/mgechev/angularjs-style-guide)
- [jQuery Style Guide](https://contribute.jquery.org/style-guide/js/)
- [JavaScript Style Guide](https://www.w3schools.com/js/js_conventions.asp)
- [JSON Style Guide](https://google.github.io/styleguide/jsoncstyleguide.xml)
- [Kotlin Style Guide](https://kotlinlang.org/docs/coding-conventions.html)
- [Markdown Style Guide](https://cirosantilli.com/markdown-style-guide/)
- [Perl Style Guide](https://perldoc.perl.org/perlstyle)
- [PHP Style Guide](https://phptherightway.com/)
- [Python Style Guide](https://peps.python.org/pep-0008/)
- [Ruby Style Guide](https://rubystyle.guide/)
- [Rust Style Guide](https://github.com/rust-lang/rust/tree/HEAD/src/doc/style-guide/src)
- [Swift Style Guide](https://www.swift.org/documentation/api-design-guidelines/)
- [TypeScript Style Guide](https://www.typescriptlang.org/docs/handbook/declaration-files/do-s-and-don-ts.html)
- [Visual Basic Style Guide](https://en.wikibooks.org/wiki/Visual_Basic/Coding_Standards)
- [Shell Script Style Guide](https://google.github.io/styleguide/shellguide.html)
- [Git Usage Style Guide](https://github.com/agis/git-style-guide)
- [PowerShell Style Guide](https://github.com/PoshCode/PowerShellPracticeAndStyle)
- [CSS](https://cssguidelin.es/)
- [Sass Style Guide](https://sass-guidelin.es/)
- [HTML Style Guide](https://github.com/marcobiedermann/html-style-guide)
- [Linux kernel Style Guide](https://www.kernel.org/doc/html/latest/process/coding-style.html)
- [Node.js Style Guide](https://github.com/felixge/node-style-guide)
- [SQL Style Guide](https://www.sqlstyle.guide/)
- [Angular Style Guide](https://angular.dev/style-guide)
- [Vue Style Guide](https://vuejs.org/style-guide/rules-strongly-recommended.html)
- [Django Style Guide](https://docs.djangoproject.com/en/dev/internals/contributing/writing-code/coding-style/)

## 編碼標準範本

### `"m", "minimal"`

```text
    ```markdown
    ## 1. 簡介
    *   **目的：** 簡要說明建立編碼標準的原因（例如，為了提高程式碼品質、可維護性和團隊協作）。
    *   **範圍：** 定義此規格適用於哪些語言、專案或模組。

    ## 2. 命名慣例
    *   **變數：** `camelCase`
    *   **函式/方法：** `PascalCase` 或 `camelCase`。
    *   **類別/結構：** `PascalCase`。
    *   **常數：** `UPPER_SNAKE_CASE`。

    ## 3. 格式和風格
    *   **縮排：** 每次縮排使用 4 個空格（或製表符）。
    *   **行長：** 將行限制為最多 80 或 120 個字符。
    *   **大括號：** 使用 "K&R" 風格（開口大括號在同一行）或 "Allman" 風格（開口大括號在新行）。
    *   **空行：** 指定用多少空行來分隔邏輯程式碼塊。

    ## 4. 註解
    *   **文件字符串/函式註解：** 描述函式的目的、參數和回傳值。
    *   **內聯註解：** 解釋複雜或不明顯的邏輯。
    *   **文件標題：** 指定文件標題中應包含的資訊，例如作者、日期和文件描述。

    ## 5. 錯誤處理
    *   **一般：** 如何處理和記錄錯誤。
    *   **細節：** 使用哪些異常類型，以及應在錯誤訊息中包含哪些資訊。

    ## 6. 最佳實踐和反模式
    *   **一般：** 列出要避免的常見反模式（例如，全域變數、魔法數字）。
    *   **特定於語言：** 基於專案編程語言的特定建議。

    ## 7. 範例
    *   提供一個小的程式碼範例，演示規則的正確應用。
    *   提供一個小的程式碼範例，展示不正確的實作以及如何修正。

    ## 8. 貢獻和執行
    *   解釋標準如何執行（例如，透過程式碼審查）。
    *   提供對標準文件本身進行貢獻的指南。
    ```
```

### `"v", verbose"`

```text
    ```markdown

    # 風格指南

    此文件定義了該專案中使用的風格和慣例。
    除非另有說明，否則所有貢獻應遵循這些規則。

    ## 1. 一般程式碼風格

    - 優先考慮清晰度而非簡潔性。
    - 保持函式和方法小而專注。
    - 避免重複邏輯；優先使用共用的助手/公用程式。
    - 移除未使用的變數、匯入、程式碼路徑和文件。

    ## 2. 命名慣例

    使用描述性名稱。除非眾所周知，否則避免縮寫。

    | 項目            | 慣例                 | 範例               |
    |-----------------|----------------------|--------------------|
    | 變數            | `lower_snake_case`   | `buffer_size`      |
    | 函式            | `lower_snake_case()` | `read_file()`      |
    | 常數            | `UPPER_SNAKE_CASE`   | `MAX_RETRIES`      |
    | 類型/結構       | `PascalCase`         | `FileHeader`       |
    | 文件名稱        | `lower_snake_case`   | `file_reader.c`    |

    ## 3. 格式規則

    - 縮排：**4 個空格**
    - 行長：**最多 100 個字符**
    - 編碼：**UTF-8**，無 BOM
    - 以換行符結尾的文件

    ### 大括號（C 語言範例，請根據您的語言調整）

        ```c
        if (condition) {
            do_something();
        } else {
            do_something_else();
        }
        ```

    ### 間距

    - 關鍵字後一個空格：`if (x)`，而不是 `if(x)`
    - 頂級函式之間一個空行

    ## 4. 註解與文件

    - 解釋 *為什麼*，而不是 *是什麼*，除非意圖不清楚。
    - 當程式碼變更時保持註解最新。
    - 公開函式應包含目的和參數的簡短描述。

    建議標籤：

        ```text
        TODO: 後續工作
        FIXME: 已知不正確的行為
        NOTE: 非明顯的設計決定
        ```

    ## 5. 錯誤處理

    - 明確處理錯誤條件。
    - 避免無聲失敗；要麼回傳錯誤，要麼適當記錄。
    - 在失敗時回傳之前清理資源（文件、記憶體、控制代碼）。

    ## 6. 提交與審查做法

    ### 提交
    - 每次提交一個邏輯變更。
    - 撰寫清晰的提交訊息：

        ```text
        簡短摘要（最多約 50 個字符）
        可選的更長說明，說明內容和基本原理。
        ```

    ### 審查
    - 保持拉取請求合理的大小。
    - 在審查討論中要尊重和建設性。
    - 解決請求的變更或解釋您是否不同意。

    ## 7. 測試

    - 為新功能編寫測試。
    - 測試應該是確定性的（無隨機性，除非有種子）。
    - 優先選擇可讀的測試案例而非複雜的測試抽象。

    ## 8. 此指南的變更

    風格會演進。
    透過開啟問題或發送更新此文件的補丁來提出改進建議。
    ```
```
