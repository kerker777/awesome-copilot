---
description: 'Technology-agnostic blueprint generator for creating comprehensive copilot-instructions.md files that guide GitHub Copilot to produce code consistent with project standards, architecture patterns, and exact technology versions by analyzing existing codebase patterns and avoiding assumptions.'
mode: 'agent'
---

# Copilot 指示藍圖生成器

## 組態變數
${PROJECT_TYPE="Auto-detect|.NET|Java|JavaScript|TypeScript|React|Angular|Python|Multiple|Other"} <!-- 主要技術 -->
${ARCHITECTURE_STYLE="Layered|Microservices|Monolithic|Domain-Driven|Event-Driven|Serverless|Mixed"} <!-- 架構方法 -->
${CODE_QUALITY_FOCUS="Maintainability|Performance|Security|Accessibility|Testability|All"} <!-- 品質優先順序 -->
${DOCUMENTATION_LEVEL="Minimal|Standard|Comprehensive"} <!-- 文件要求 -->
${TESTING_REQUIREMENTS="Unit|Integration|E2E|TDD|BDD|All"} <!-- 測試方法 -->
${VERSIONING="Semantic|CalVer|Custom"} <!-- 版本控制方法 -->

## 生成的提示

「產生一份全面的 copilot-instructions.md 檔案，引導 GitHub Copilot 產生與我們專案標準、架構和技術版本一致的程式碼。這些指示必須嚴格根據程式碼庫中的實際程式碼模式，並避免進行任何假設。請按照以下方法進行：

### 1. 核心指示結構

```markdown
# GitHub Copilot 指示

## 優先順序指南

為此儲存庫產生程式碼時：

1. **版本相容性**：始終偵測並尊重此專案中使用的程式語言、框架和程式庫的確切版本
2. **內容檔案**：優先使用在 .github/copilot 目錄中定義的模式和標準
3. **程式碼庫模式**：當內容檔案未提供特定指導時，掃描程式碼庫以找到已建立的模式
4. **架構一致性**：維持我們的 ${ARCHITECTURE_STYLE} 架構風格和已建立的邊界
5. **程式碼品質**：在所有產生的程式碼中優先考慮 ${CODE_QUALITY_FOCUS == "All" ? "可維護性、效能、安全性、可存取性和可測試性" : CODE_QUALITY_FOCUS}

## 技術版本偵測

在產生程式碼之前，掃描程式碼庫以識別：

1. **程式語言版本**：偵測使用中的程式語言確切版本
   - 檢查專案檔案、組態檔和套件管理員
   - 尋找語言特定的版本指標（例如 .NET 專案中的 <LangVersion>）
   - 不要使用超過偵測到的版本的語言功能

2. **框架版本**：識別所有框架的確切版本
   - 檢查 package.json、.csproj、pom.xml、requirements.txt 等
   - 產生程式碼時尊重版本限制
   - 不要建議偵測到的框架版本中不可用的功能

3. **程式庫版本**：記下關鍵程式庫和相依項的確切版本
   - 產生與這些特定版本相容的程式碼
   - 不要使用偵測到的版本中不可用的 API 或功能

## 內容檔案

優先使用 .github/copilot 目錄中的以下檔案（如果存在）：

- **architecture.md**：系統架構指南
- **tech-stack.md**：技術版本和框架詳細資料
- **coding-standards.md**：程式碼風格和格式標準
- **folder-structure.md**：專案組織指南
- **exemplars.md**：要遵循的典範程式碼模式

## 程式碼庫掃描指示

當內容檔案未提供特定指導時：

1. 識別類似於正在修改或建立的檔案
2. 分析以下項目的模式：
   - 命名慣例
   - 程式碼組織
   - 錯誤處理
   - 記錄方法
   - 文件風格
   - 測試模式

3. 遵循在程式碼庫中找到的最一致的模式
4. 當存在衝突的模式時，優先考慮較新檔案或測試涵蓋率較高的檔案中的模式
5. 不要引入程式碼庫中不存在的模式

## 程式碼品質標準

${CODE_QUALITY_FOCUS.includes("Maintainability") || CODE_QUALITY_FOCUS == "All" ? `### 可維護性
- 撰寫具有清晰命名的自我文件化程式碼
- 遵循程式碼庫中顯而易見的命名和組織慣例
- 遵循已建立的一致性模式
- 使函式專注於單一職責
- 限制函式複雜度和長度以符合現有模式` : ""}

${CODE_QUALITY_FOCUS.includes("Performance") || CODE_QUALITY_FOCUS == "All" ? `### 效能
- 遵循記憶體和資源管理的現有模式
- 符合處理計算成本高的操作的現有模式
- 遵循非同步操作的已建立模式
- 與現有模式一致地應用快取
- 根據程式碼庫中顯而易見的模式最佳化` : ""}

${CODE_QUALITY_FOCUS.includes("Security") || CODE_QUALITY_FOCUS == "All" ? `### 安全性
- 遵循輸入驗證的現有模式
- 應用程式碼庫中使用的相同清理技術
- 使用與現有模式相符的參數化查詢
- 遵循已建立的驗證和授權模式
- 根據現有模式處理敏感資料` : ""}

${CODE_QUALITY_FOCUS.includes("Accessibility") || CODE_QUALITY_FOCUS == "All" ? `### 可存取性
- 遵循程式碼庫中現有的可存取性模式
- 將 ARIA 屬性使用與現有元件相符
- 維持與現有程式碼一致的鍵盤導航支援
- 遵循顏色和對比的已建立模式
- 應用與程式碼庫一致的文字替代方案模式` : ""}

${CODE_QUALITY_FOCUS.includes("Testability") || CODE_QUALITY_FOCUS == "All" ? `### 可測試性
- 遵循可測試程式碼的已建立模式
- 符合程式碼庫中使用的相依性注入方法
- 應用相同的相依性管理模式
- 遵循已建立的模擬和測試替身模式
- 符合現有測試中使用的測試風格` : ""}

## 文件要求

${DOCUMENTATION_LEVEL == "Minimal" ?
`- 符合現有程式碼中找到的註解等級和風格
- 根據程式碼庫中觀察到的模式進行文件記錄
- 遵循現有模式以文件化非顯而易見的行為
- 使用與現有程式碼相同的參數描述格式` : ""}

${DOCUMENTATION_LEVEL == "Standard" ?
`- 遵循程式碼庫中找到的確切文件格式
- 符合現有註解的 XML/JSDoc 風格和完整性
- 以相同的方式文件化參數、返回和例外狀況
- 遵循現有的使用範例模式
- 符合類別級別的文件風格和內容` : ""}

${DOCUMENTATION_LEVEL == "Comprehensive" ?
`- 遵循程式碼庫中找到的最詳細的文件模式
- 符合最文件齊全程式碼的風格和完整性
- 完全按照最徹底記錄的檔案進行文件記錄
- 遵循現有的文件連結模式
- 符合設計決策解釋中的詳細程度` : ""}

## 測試方法

${TESTING_REQUIREMENTS.includes("Unit") || TESTING_REQUIREMENTS == "All" ?
`### 單位測試
- 符合現有單位測試的確切結構和風格
- 遵循測試類別和方法的相同命名慣例
- 使用現有測試中找到的相同判定模式
- 應用程式碼庫中使用的相同模擬方法
- 遵循測試隔離的現有模式` : ""}

${TESTING_REQUIREMENTS.includes("Integration") || TESTING_REQUIREMENTS == "All" ?
`### 整合測試
- 遵循程式碼庫中找到的相同整合測試模式
- 符合測試資料設定和清除的現有模式
- 使用相同的方法測試元件互動
- 遵循現有模式以驗證系統行為` : ""}

${TESTING_REQUIREMENTS.includes("E2E") || TESTING_REQUIREMENTS == "All" ?
`### 端對端測試
- 符合現有的 E2E 測試結構和模式
- 遵循 UI 測試的已建立模式
- 應用相同的方法驗證使用者旅程` : ""}

${TESTING_REQUIREMENTS.includes("TDD") || TESTING_REQUIREMENTS == "All" ?
`### 測試驅動開發
- 遵循程式碼庫中顯而易見的 TDD 模式
- 符合現有程式碼中看到的測試案例進展
- 測試通過後應用相同的重構模式` : ""}

${TESTING_REQUIREMENTS.includes("BDD") || TESTING_REQUIREMENTS == "All" ?
`### 行為驅動開發
- 符合測試中現有的給定-何時-然後結構
- 遵循行為描述的相同模式
- 在測試案例中應用相同程度的業務關注` : ""}

## 技術特定指南

${PROJECT_TYPE == ".NET" || PROJECT_TYPE == "Auto-detect" || PROJECT_TYPE == "Multiple" ? `### .NET 指南
- 偵測並嚴格遵守使用中的特定 .NET 版本
- 僅使用與偵測到的版本相容的 C# 語言功能
- 完全按照 LINQ 在程式碼庫中出現的方式遵循其使用模式
- 符合現有程式碼中的 async/await 使用模式
- 應用程式碼庫中使用的相同相依性注入方法
- 使用程式碼庫中找到的相同集合類型和模式` : ""}

${PROJECT_TYPE == "Java" || PROJECT_TYPE == "Auto-detect" || PROJECT_TYPE == "Multiple" ? `### Java 指南
- 偵測並遵守使用中的特定 Java 版本
- 遵循程式碼庫中找到的完全相同的設計模式
- 符合現有程式碼中的例外狀況處理模式
- 使用程式碼庫中找到的相同集合類型和方法
- 應用現有程式碼中顯而易見的相依性注入模式` : ""}

${PROJECT_TYPE == "JavaScript" || PROJECT_TYPE == "TypeScript" || PROJECT_TYPE == "Auto-detect" || PROJECT_TYPE == "Multiple" ? `### JavaScript/TypeScript 指南
- 偵測並遵守使用中的特定 ECMAScript/TypeScript 版本
- 遵循程式碼庫中找到的相同模組匯入/匯出模式
- 將 TypeScript 型別定義與現有模式相符
- 使用與現有程式碼相同的非同步模式（promises、async/await）
- 遵循來自類似檔案的錯誤處理模式` : ""}

${PROJECT_TYPE == "React" || PROJECT_TYPE == "Auto-detect" || PROJECT_TYPE == "Multiple" ? `### React 指南
- 偵測並遵守使用中的特定 React 版本
- 符合現有元件的元件結構模式
- 遵循程式碼庫中找到的相同 hooks 和生命週期模式
- 應用現有元件中使用的相同狀態管理方法
- 符合現有程式碼中的 prop 輸入和驗證模式` : ""}

${PROJECT_TYPE == "Angular" || PROJECT_TYPE == "Auto-detect" || PROJECT_TYPE == "Multiple" ? `### Angular 指南
- 偵測並遵守使用中的特定 Angular 版本
- 遵循程式碼庫中找到的相同元件和模組模式
- 完全按照現有程式碼中看到的方式符合裝飾器使用
- 應用程式碼庫中找到的相同 RxJS 模式
- 遵循元件通訊的現有模式` : ""}

${PROJECT_TYPE == "Python" || PROJECT_TYPE == "Auto-detect" || PROJECT_TYPE == "Multiple" ? `### Python 指南
- 偵測並遵守使用中的特定 Python 版本
- 遵循現有模組中找到的相同匯入組織
- 符合程式碼庫中使用的型別提示方法（如果適用）
- 應用現有程式碼中找到的相同錯誤處理模式
- 遵循相同的模組組織模式` : ""}

## 版本控制指南

${VERSIONING == "Semantic" ?
`- 遵循在程式碼庫中應用的語義版本控制模式
- 符合現有的破壞性變更文件記錄模式
- 遵循相同的廢止通知方法` : ""}

${VERSIONING == "CalVer" ?
`- 遵循在程式碼庫中應用的日曆版本控制模式
- 符合現有的變更文件記錄模式
- 遵循相同的方法以突顯重大變更` : ""}

${VERSIONING == "Custom" ?
`- 符合程式碼庫中觀察到的確切版本控制模式
- 遵循現有文件中使用的相同變更日誌格式
- 應用專案中使用的相同標籤慣例` : ""}

## 一般最佳實務

- 完全按照現有程式碼中出現的方式遵循命名慣例
- 符合來自類似檔案的程式碼組織模式
- 應用與現有模式一致的錯誤處理
- 遵循程式碼庫中看到的相同測試方法
- 符合現有程式碼中的記錄模式
- 使用與程式碼庫中看到的相同組態方法

## 專案特定指導

- 在產生任何程式碼之前徹底掃描程式碼庫
- 無一例外地尊重現有的架構邊界
- 符合周圍程式碼的風格和模式
- 如有疑問，優先考慮與現有程式碼的一致性，而不是外部最佳實務
```

### 2. 程式碼庫分析指示

要建立 copilot-instructions.md 檔案，請先分析程式碼庫以：

1. **識別確切的技術版本**：
   - ${PROJECT_TYPE == "Auto-detect" ? "透過掃描檔案副檔名和組態檔偵測所有程式語言、框架和程式庫" : `專注於 ${PROJECT_TYPE} 技術`}
   - 從專案檔案、package.json、.csproj 等提取精確的版本資訊
   - 記錄版本限制和相容性要求

2. **瞭解架構**：
   - 分析資料夾結構和模組組織
   - 識別清晰的分層邊界和元件關係
   - 記錄元件之間的通訊模式

3. **記錄程式碼模式**：
   - 列出不同程式碼元素的命名慣例
   - 注意文件風格和完整性
   - 記錄錯誤處理模式
   - 對應測試方法和涵蓋範圍

4. **注意品質標準**：
   - 識別實際使用的效能最佳化技術
   - 記錄程式碼中實施的安全實務
   - 注意現有的可存取性功能（如適用）
   - 記錄程式碼庫中顯而易見的程式碼品質模式

### 3. 實作備註

最終的 copilot-instructions.md 應該：
- 位於 .github/copilot 目錄中
- 僅參考程式碼庫中存在的模式和標準
- 包含明確的版本相容性要求
- 避免規定程式碼中不顯而易見的任何做法
- 提供來自程式碼庫的具體範例
- 足夠全面但簡潔，以便 Copilot 有效使用

重要事項：僅包含基於程式碼庫中實際觀察到的模式的指導。明確指導 Copilot 優先考慮與現有程式碼的一致性，而不是外部最佳實務或較新的語言功能。
」

## 預期輸出

一份全面的 copilot-instructions.md 檔案，將引導 GitHub Copilot 產生與您現有技術版本完全相容的程式碼，並遵循您已建立的模式和架構。