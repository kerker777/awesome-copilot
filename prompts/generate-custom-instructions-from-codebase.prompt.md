---
description: 'GitHub Copilot 的遷移和程式碼演進指令產生器。分析兩個專案版本（分支、提交或發行版本）之間的差異，以建立精確的指令，讓 Copilot 在技術遷移、主要重構或框架版本升級期間維護一致性。'
mode: 'agent'
---

# 遷移和程式碼演進指令產生器

## 設定變數

```
${MIGRATION_TYPE="框架版本|架構重構|技術遷移|相依性更新|模式變更"}
<!-- 遷移或演進的類型 -->

${SOURCE_REFERENCE="branch|commit|tag"}
<!-- 來源參考點（之前的狀態） -->

${TARGET_REFERENCE="branch|commit|tag"}
<!-- 目標參考點（之後的狀態） -->

${ANALYSIS_SCOPE="整個專案|特定資料夾|僅修改的檔案"}
<!-- 分析範圍 -->

${CHANGE_FOCUS="重大變更|新慣例|過時模式|API 變更|設定"}
<!-- 變更的主要面向 -->

${AUTOMATION_LEVEL="保守|平衡|積極"}
<!-- Copilot 建議的自動化等級 -->

${GENERATE_EXAMPLES="true|false"}
<!-- 包含轉換範例 -->

${VALIDATION_REQUIRED="true|false"}
<!-- 應用前需要驗證 -->
```

## 產生的提示詞

```
"分析兩個專案狀態之間的程式碼演進，以產生 GitHub Copilot 的精確遷移指令。這些指令將指導 Copilot 在未來的修改中自動應用相同的轉換模式。遵循此方法：

### 階段 1：比較狀態分析

#### 結構變更偵測
- 比較 ${SOURCE_REFERENCE} 和 ${TARGET_REFERENCE} 之間的資料夾結構
- 識別移動、重新命名或刪除的檔案
- 分析設定檔中的變更
- 記錄新增和移除的相依性

#### 程式碼轉換分析
${MIGRATION_TYPE == "框架版本" ?
  "- 識別框架版本之間的 API 變更
   - 分析正在使用的新功能
   - 記錄過時的方法/屬性
   - 記錄語法或慣例變更" : ""}

${MIGRATION_TYPE == "架構重構" ?
  "- 分析架構模式變更
   - 識別引入的新抽象
   - 記錄職責重組
   - 記錄資料流變更" : ""}

${MIGRATION_TYPE == "技術遷移" ?
  "- 分析一種技術替換為另一種技術
   - 識別功能等效性
   - 記錄 API 和語法變更
   - 記錄新的相依性和設定" : ""}

#### 轉換模式提取
- 識別應用的重複轉換
- 分析從舊格式到新格式的轉換規則
- 記錄例外和特殊情況
- 建立前後對應矩陣

### 階段 2：遷移指令產生

建立一個具有此結構的 `.github/copilot-migration-instructions.md` 檔案：

\`\`\`markdown
# GitHub Copilot 遷移指令

## 遷移情境
- **類型**：${MIGRATION_TYPE}
- **從**：${SOURCE_REFERENCE}
- **到**：${TARGET_REFERENCE}
- **日期**：[GENERATION_DATE]
- **範圍**：${ANALYSIS_SCOPE}

## 自動轉換規則

### 1. 強制轉換
${AUTOMATION_LEVEL != "保守" ?
  "[AUTOMATIC_TRANSFORMATION_RULES]
   - **舊模式**：[OLD_CODE]
   - **新模式**：[NEW_CODE]
   - **觸發**：何時偵測此模式
   - **動作**：自動應用的轉換" : ""}

### 2. 需要驗證的轉換
${VALIDATION_REQUIRED == "true" ?
  "[TRANSFORMATIONS_WITH_VALIDATION]
   - **偵測到的模式**：[DESCRIPTION]
   - **建議的轉換**：[NEW_APPROACH]
   - **所需驗證**：[VALIDATION_CRITERIA]
   - **替代方案**：[ALTERNATIVE_OPTIONS]" : ""}

### 3. API 對應
${CHANGE_FOCUS == "API 變更" || MIGRATION_TYPE == "框架版本" ?
  "[API_CORRESPONDENCE_TABLE]
   | 舊 API   | 新 API   | 備註     | 範例        |
   | --------- | --------- | --------- | -------------- |
   | [OLD_API] | [NEW_API] | [CHANGES] | [CODE_EXAMPLE] | " : ""} |

### 4. 要採用的新模式
[DETECTED_EMERGING_PATTERNS]
- **模式**：[PATTERN_NAME]
- **用法**：[WHEN_TO_USE]
- **實作**：[HOW_TO_IMPLEMENT]
- **優勢**：[ADVANTAGES]

### 5. 要避免的過時模式
[DETECTED_OBSOLETE_PATTERNS]
- **過時模式**：[OLD_PATTERN]
- **為何避免**：[REASONS]
- **替代方案**：[NEW_PATTERN]
- **遷移**：[CONVERSION_STEPS]

## 檔案類型特定指令

${GENERATE_EXAMPLES == "true" ?
  "### 設定檔
   [CONFIG_TRANSFORMATION_EXAMPLES]

   ### 主要原始檔案
   [SOURCE_TRANSFORMATION_EXAMPLES]

   ### 測試檔案
   [TEST_TRANSFORMATION_EXAMPLES]" : ""}

## 驗證與安全性

### 自動控制點
- 每次轉換後執行的驗證
- 驗證變更要執行的測試
- 要監控的效能指標
- 要執行的相容性檢查

### 手動升級
需要人工介入的情況：
- [COMPLEX_CASES_LIST]
- [ARCHITECTURAL_DECISIONS]
- [BUSINESS_IMPACTS]

## 遷移監控

### 追蹤指標
- 自動遷移的程式碼百分比
- 所需的手動驗證數量
- 自動轉換的錯誤率
- 每個檔案的平均遷移時間

### 錯誤報告
如何向 Copilot 報告不正確的轉換：
- 改進規則的回饋模式
- 要記錄的例外
- 對指令進行的調整

\`\`\`

### 階段 3：情境範例產生

${GENERATE_EXAMPLES == "true" ?
  "#### 轉換範例
   對於每個識別的模式，產生：

   \`\`\`
   // 之前（${SOURCE_REFERENCE}）
   [OLD_CODE_EXAMPLE]

   // 之後（${TARGET_REFERENCE}）
   [NEW_CODE_EXAMPLE]

   // COPILOT 指令
   當您看到此模式 [TRIGGER]，將其轉換為 [NEW_PATTERN]，遵循以下步驟：[STEPS]
   \`\`\`" : ""}

### 階段 4：驗證與最佳化

#### 指令測試
- 在測試程式碼上應用指令
- 驗證轉換一致性
- 根據結果調整規則
- 記錄例外和邊緣案例

#### 迭代最佳化
${AUTOMATION_LEVEL == "積極" ?
  "- 優化規則以最大化自動化
   - 減少偵測中的誤判
   - 改善轉換準確性
   - 記錄經驗教訓" : ""}

### 最終結果

遷移指令使 GitHub Copilot 能夠：
1. **自動應用**未來修改期間的相同轉換
2. **維護一致性**與新採用的慣例
3. **避免過時模式**透過自動提出替代方案
4. **加速未來遷移**透過利用獲得的經驗
5. **減少錯誤**透過自動化重複轉換

這些指令將 Copilot 轉變為智慧遷移助理，能夠一致且可靠地複製您的技術演進決策。
"
```

## 典型使用案例

### 框架版本遷移
非常適合記錄從 Angular 14 到 Angular 17、React Class Components 到 Hooks，或 .NET Framework 到 .NET Core 的轉換。自動識別重大變更並產生相應的轉換規則。

### 技術堆疊演進
在完全替換技術時至關重要：jQuery 到 React、REST 到 GraphQL、SQL 到 NoSQL。建立包含模式對應的全面遷移指南。

### 架構重構
非常適合大型重構，如 Monolith 到 Microservices、MVC 到 Clean Architecture，或 Component 到 Composable 架構。為未來類似的轉換保留架構知識。

### 設計模式現代化
用於採用新模式：Repository Pattern、Dependency Injection、Observer 到 Reactive Programming。記錄理由和實作差異。

## 獨特優勢

### 🧠 **人工智慧增強**
與傳統遷移文件不同，這些指令「訓練」GitHub Copilot 在未來的程式碼修改期間自動複製您的技術演進決策。

### 🔄 **知識資本化**
將特定專案經驗轉化為可重複使用的規則，避免遷移專業知識的流失並加速未來類似的轉換。

### 🎯 **情境感知精確度**
不是泛泛的建議，而是針對您特定的程式碼庫量身訂製的指令，並包含來自專案演進的真實前後範例。

### ⚡ **自動化一致性**
確保新增的程式碼自動遵循新慣例，防止架構退化並維護程式碼演進的一致性。
