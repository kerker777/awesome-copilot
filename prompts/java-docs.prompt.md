---
mode: 'agent'
tools: ['changes', 'search/codebase', 'edit/editFiles', 'problems']
description: '確保 Java 類型使用 Javadoc 註解進行文件化,並遵循文件化最佳實踐。'
---

# Java 文件（Javadoc）最佳實踐

- 公開和受保護的成員應使用 Javadoc 註解進行文件化。
- 建議也對套件私有和私有成員進行文件化,特別是如果它們複雜或不能自我說明。
- Javadoc 註解的第一句是摘要描述。它應該是方法功能的簡潔概述,並以句點結尾。
- 使用 `@param` 標記方法參數。描述以小寫字母開頭,不以句點結尾。
- 使用 `@return` 標記方法返回值。
- 使用 `@throws` 或 `@exception` 記錄方法拋出的例外。
- 使用 `@see` 參考其他類型或成員。
- 使用 `{@inheritDoc}` 從基底類別或介面繼承文件。
  - 除非有重大行為變更,在這種情況下應記錄差異。
- 使用 `@param <T>` 標記泛型類型或方法中的類型參數。
- 使用 `{@code}` 標記內聯程式碼片段。
- 使用 `<pre>{@code ... }</pre>` 標記程式碼區塊。
- 使用 `@since` 指示功能引入的時間（例如版本號）。
- 使用 `@version` 指定成員的版本。
- 使用 `@author` 指定程式碼的作者。
- 使用 `@deprecated` 將成員標記為已棄用並提供替代方案。
