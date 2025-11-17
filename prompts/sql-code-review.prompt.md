---
mode: 'agent'
tools: ['changes', 'search/codebase', 'edit/editFiles', 'problems']
description: 'Universal SQL code review assistant that performs comprehensive security, maintainability, and code quality analysis across all SQL databases (MySQL, PostgreSQL, SQL Server, Oracle). Focuses on SQL injection prevention, access control, code standards, and anti-pattern detection. Complements SQL optimization prompt for complete development coverage.'
tested_with: 'GitHub Copilot Chat (GPT-4o) - Validated July 20, 2025'
---

# SQL 程式碼審查

對 ${selection}（或若無選取則為整個專案）進行徹底的 SQL 程式碼審查，重點關注安全性、效能、可維護性和資料庫最佳實踐。

## 🔒 安全性分析

### SQL 注入預防
```sql
-- ❌ CRITICAL: SQL Injection vulnerability
query = "SELECT * FROM users WHERE id = " + userInput;
query = f"DELETE FROM orders WHERE user_id = {user_id}";

-- ✅ SECURE: Parameterized queries
-- PostgreSQL/MySQL
PREPARE stmt FROM 'SELECT * FROM users WHERE id = ?';
EXECUTE stmt USING @user_id;

-- SQL Server
EXEC sp_executesql N'SELECT * FROM users WHERE id = @id', N'@id INT', @id = @user_id;
```

### 存取控制與權限
- **最小權限原則**：僅授予所需的最小權限
- **角色型存取控制**：使用資料庫角色而非直接指派使用者權限
- **模式安全性**：適當的模式所有權和存取控制
- **函式／預存程序安全性**：檢視定義者（DEFINER）與調用者（INVOKER）權限

### 資料保護
- **敏感資料外洩**：避免在包含敏感欄位的資料表上使用 SELECT *
- **稽核紀錄**：確保敏感操作已被記錄
- **資料遮罩**：使用檢視表或函式來遮罩敏感資料
- **加密**：驗證敏感資料的加密儲存

## ⚡ 效能最佳化

### 查詢結構分析
```sql
-- ❌ BAD: Inefficient query patterns
SELECT DISTINCT u.*
FROM users u, orders o, products p
WHERE u.id = o.user_id
AND o.product_id = p.id
AND YEAR(o.order_date) = 2024;

-- ✅ GOOD: Optimized structure
SELECT u.id, u.name, u.email
FROM users u
INNER JOIN orders o ON u.id = o.user_id
WHERE o.order_date >= '2024-01-01'
AND o.order_date < '2025-01-01';
```

### 索引策略審查
- **缺失的索引**：識別需要建立索引的欄位
- **過度索引**：尋找未使用或冗餘的索引
- **複合索引**：針對複雜查詢的多欄位索引
- **索引維護**：檢查索引碎片化或過時問題

### 聯接最佳化
- **聯接型別**：驗證適當的聯接型別（INNER 與 LEFT 與 EXISTS）
- **聯接順序**：優化以先處理較小的結果集
- **笛卡爾乘積**：識別並修正缺失的聯接條件
- **子查詢與聯接**：選擇最有效率的方法

### 聚合與視窗函式
```sql
-- ❌ BAD: Inefficient aggregation
SELECT user_id,
       (SELECT COUNT(*) FROM orders o2 WHERE o2.user_id = o1.user_id) as order_count
FROM orders o1
GROUP BY user_id;

-- ✅ GOOD: Efficient aggregation
SELECT user_id, COUNT(*) as order_count
FROM orders
GROUP BY user_id;
```

## 🛠️ 程式碼品質與可維護性

### SQL 風格與格式化
```sql
-- ❌ BAD: Poor formatting and style
select u.id,u.name,o.total from users u left join orders o on u.id=o.user_id where u.status='active' and o.order_date>='2024-01-01';

-- ✅ GOOD: Clean, readable formatting
SELECT u.id,
       u.name,
       o.total
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.status = 'active'
  AND o.order_date >= '2024-01-01';
```

### 命名規範
- **一致的命名**：資料表、欄位、條件約束遵循一致的模式
- **具描述性的名稱**：資料庫物件的清晰、有意義的名稱
- **保留字**：避免使用資料庫保留字作為識別碼
- **大小寫一致性**：在整個模式中使用一致的大小寫

### 模式設計審查
- **正規化**：適當的正規化程度（避免過度或不足正規化）
- **資料型別**：最佳的資料型別選擇以支援儲存和效能
- **條件約束**：適當使用 PRIMARY KEY、FOREIGN KEY、CHECK、NOT NULL
- **預設值**：欄位的適當預設值

## 🗄️ 資料庫特定最佳實踐

### PostgreSQL
```sql
-- Use JSONB for JSON data
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    data JSONB NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- GIN index for JSONB queries
CREATE INDEX idx_events_data ON events USING gin(data);

-- Array types for multi-value columns
CREATE TABLE tags (
    post_id INT,
    tag_names TEXT[]
);
```

### MySQL
```sql
-- Use appropriate storage engines
CREATE TABLE sessions (
    id VARCHAR(128) PRIMARY KEY,
    data TEXT,
    expires TIMESTAMP
) ENGINE=InnoDB;

-- Optimize for InnoDB
ALTER TABLE large_table
ADD INDEX idx_covering (status, created_at, id);
```

### SQL Server
```sql
-- Use appropriate data types
CREATE TABLE products (
    id BIGINT IDENTITY(1,1) PRIMARY KEY,
    name NVARCHAR(255) NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    created_at DATETIME2 DEFAULT GETUTCDATE()
);

-- Columnstore indexes for analytics
CREATE COLUMNSTORE INDEX idx_sales_cs ON sales;
```

### Oracle
```sql
-- Use sequences for auto-increment
CREATE SEQUENCE user_id_seq START WITH 1 INCREMENT BY 1;

CREATE TABLE users (
    id NUMBER DEFAULT user_id_seq.NEXTVAL PRIMARY KEY,
    name VARCHAR2(255) NOT NULL
);
```

## 🧪 測試與驗證

### 資料完整性檢查
```sql
-- Verify referential integrity
SELECT o.user_id
FROM orders o
LEFT JOIN users u ON o.user_id = u.id
WHERE u.id IS NULL;

-- Check for data consistency
SELECT COUNT(*) as inconsistent_records
FROM products
WHERE price < 0 OR stock_quantity < 0;
```

### 效能測試
- **執行計畫**：檢視查詢執行計畫
- **負載測試**：使用實際資料量測試查詢
- **壓力測試**：驗證並行負載下的效能
- **迴歸測試**：確保最佳化不會破壞功能

## 📊 常見反模式

### N+1 查詢問題
```sql
-- ❌ BAD: N+1 queries in application code
for user in users:
    orders = query("SELECT * FROM orders WHERE user_id = ?", user.id)

-- ✅ GOOD: Single optimized query
SELECT u.*, o.*
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;
```

### 過度使用 DISTINCT
```sql
-- ❌ BAD: DISTINCT masking join issues
SELECT DISTINCT u.name
FROM users u, orders o
WHERE u.id = o.user_id;

-- ✅ GOOD: Proper join without DISTINCT
SELECT u.name
FROM users u
INNER JOIN orders o ON u.id = o.user_id
GROUP BY u.name;
```

### WHERE 子句中的函式濫用
```sql
-- ❌ BAD: Functions prevent index usage
SELECT * FROM orders
WHERE YEAR(order_date) = 2024;

-- ✅ GOOD: Range conditions use indexes
SELECT * FROM orders
WHERE order_date >= '2024-01-01'
  AND order_date < '2025-01-01';
```

## 📋 SQL 審查檢查清單

### 安全性
- [ ] 所有使用者輸入都已參數化
- [ ] 未使用字串串接來動態建構 SQL
- [ ] 適當的存取控制和權限
- [ ] 敏感資料已妥善保護
- [ ] SQL 注入攻擊向量已排除

### 效能
- [ ] 常查詢的欄位已建立索引
- [ ] 無不必要的 SELECT * 陳述式
- [ ] 聯接已最佳化並使用適當型別
- [ ] WHERE 子句具選擇性且使用索引
- [ ] 子查詢已最佳化或轉換為聯接

### 程式碼品質
- [ ] 一致的命名規範
- [ ] 適當的格式化和縮排
- [ ] 複雜邏輯的有意義註解
- [ ] 使用適當的資料型別
- [ ] 已實現錯誤處理

### 模式設計
- [ ] 資料表已妥善正規化
- [ ] 條件約束強制執行資料完整性
- [ ] 索引支援查詢模式
- [ ] 已定義外鍵關係
- [ ] 預設值適當

## 🎯 審查輸出格式

### 問題範本
```
## [PRIORITY] [CATEGORY]: [Brief Description]

**Location**: [Table/View/Procedure name and line number if applicable]
**Issue**: [Detailed explanation of the problem]
**Security Risk**: [If applicable - injection risk, data exposure, etc.]
**Performance Impact**: [Query cost, execution time impact]
**Recommendation**: [Specific fix with code example]

**Before**:
```sql
-- Problematic SQL
```

**After**:
```sql
-- Improved SQL
```

**Expected Improvement**: [Performance gain, security benefit]
```

### 摘要評估
- **安全性分數**：[1-10] - SQL 注入保護、存取控制
- **效能分數**：[1-10] - 查詢效率、索引使用
- **可維護性分數**：[1-10] - 程式碼品質、文件
- **模式品質分數**：[1-10] - 設計模式、正規化

### 前 3 項優先行動
1. **[關鍵安全修正]**：處理 SQL 注入漏洞
2. **[效能最佳化]**：新增缺失的索引或最佳化查詢
3. **[程式碼品質]**：改進命名規範和文件

著重於提供可實施的、資料庫無關的建議，同時突出平台特定的最佳化和最佳實踐。
