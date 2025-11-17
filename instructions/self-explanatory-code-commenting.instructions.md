---
description: 'Guidelines for GitHub Copilot to write comments to achieve self-explanatory code with less comments. Examples are in JavaScript but it should work on any language that has comments.'
applyTo: '**'
---

# 自我說明型程式碼註解指引

## 核心原則
**撰寫能夠自我說明的程式碼。只在必要時添加註解，以說明「為什麼」，而非「是什麼」。**
大多數情況下，我們根本不需要註解。

## 註解指引

### ❌ 應避免的註解類型

**過於明顯的註解**
```javascript
// Bad: States the obvious
let counter = 0;  // Initialize counter to zero
counter++;  // Increment counter by one
```

**冗餘的註解**
```javascript
// Bad: Comment repeats the code
function getUserName() {
    return user.name;  // Return the user's name
}
```

**過時的註解**
```javascript
// Bad: Comment doesn't match the code
// Calculate tax at 5% rate
const tax = price * 0.08;  // Actually 8%
```

### ✅ 應撰寫的註解類型

**複雜的業務邏輯**
```javascript
// Good: Explains WHY this specific calculation
// Apply progressive tax brackets: 10% up to 10k, 20% above
const tax = calculateProgressiveTax(income, [0.10, 0.20], [10000]);
```

**非顯而易見的演算法**
```javascript
// Good: Explains the algorithm choice
// Using Floyd-Warshall for all-pairs shortest paths
// because we need distances between all nodes
for (let k = 0; k < vertices; k++) {
    for (let i = 0; i < vertices; i++) {
        for (let j = 0; j < vertices; j++) {
            // ... implementation
        }
    }
}
```

**正規表達式模式**
```javascript
// Good: Explains what the regex matches
// Match email format: username@domain.extension
const emailPattern = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
```

**API 的限制條件或陷阱**
```javascript
// Good: Explains external constraint
// GitHub API rate limit: 5000 requests/hour for authenticated users
await rateLimiter.wait();
const response = await fetch(githubApiUrl);
```

## 決策框架

在撰寫註解之前，先問自己：
1. **程式碼是否能自我說明？** → 不需要註解
2. **使用更好的變數名稱或函式名稱能否消除註解的必要？** → 應該重構程式碼
3. **這個註解是否說明「為什麼」，而非「是什麼」？** → 好的註解
4. **這對未來的維護者有幫助嗎？** → 好的註解

## 特殊的註解情況

### 公共 API
```javascript
/**
 * Calculate compound interest using the standard formula.
 *
 * @param {number} principal - Initial amount invested
 * @param {number} rate - Annual interest rate (as decimal, e.g., 0.05 for 5%)
 * @param {number} time - Time period in years
 * @param {number} compoundFrequency - How many times per year interest compounds (default: 1)
 * @returns {number} Final amount after compound interest
 */
function calculateCompoundInterest(principal, rate, time, compoundFrequency = 1) {
    // ... implementation
}
```

### 設定和常數
```javascript
// Good: Explains the source or reasoning
const MAX_RETRIES = 3;  // Based on network reliability studies
const API_TIMEOUT = 5000;  // AWS Lambda timeout is 15s, leaving buffer
```

### 註記
```javascript
// TODO: Replace with proper user authentication after security review
// FIXME: Memory leak in production - investigate connection pooling
// HACK: Workaround for bug in library v2.1.0 - remove after upgrade
// NOTE: This implementation assumes UTC timezone for all calculations
// WARNING: This function modifies the original array instead of creating a copy
// PERF: Consider caching this result if called frequently in hot path
// SECURITY: Validate input to prevent SQL injection before using in query
// BUG: Edge case failure when array is empty - needs investigation
// REFACTOR: Extract this logic into separate utility function for reusability
// DEPRECATED: Use newApiFunction() instead - this will be removed in v3.0
```

## 應避免的反模式

### 死亡程式碼註解
```javascript
// Bad: Don't comment out code
// const oldFunction = () => { ... };
const newFunction = () => { ... };
```

### 變更日誌註解
```javascript
// Bad: Don't maintain history in comments
// Modified by John on 2023-01-15
// Fixed bug reported by Sarah on 2023-02-03
function processData() {
    // ... implementation
}
```

### 分隔符註解
```javascript
// Bad: Don't use decorative comments
//=====================================
// UTILITY FUNCTIONS
//=====================================
```

## 品質檢查清單

提交程式碼前，請確保您的註解：
- [ ] 說明「為什麼」，而非「是什麼」
- [ ] 文法正確且清晰易懂
- [ ] 隨著程式碼演進仍保持準確
- [ ] 為程式碼理解增加實質價值
- [ ] 位置妥當（位於所說明的程式碼之上）
- [ ] 使用正確的拼寫和專業的語言

## 摘要

記住：**最好的註解就是那些根本不需要寫的註解，因為程式碼已經能夠自我說明。**
