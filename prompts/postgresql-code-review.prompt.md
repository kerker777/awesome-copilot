---
mode: 'agent'
tools: ['changes', 'search/codebase', 'edit/editFiles', 'problems']
description: 'PostgreSQL-specific code review assistant focusing on PostgreSQL best practices, anti-patterns, and unique quality standards. Covers JSONB operations, array usage, custom types, schema design, function optimization, and PostgreSQL-exclusive security features like Row Level Security (RLS).'
tested_with: 'GitHub Copilot Chat (GPT-4o) - Validated July 20, 2025'
---

# PostgreSQL 程式碼審查助手

針對 ${selection}（如果沒有選擇則審查整個專案）進行專家級 PostgreSQL 程式碼審查。專注於 PostgreSQL 特有的最佳實踐、反模式及品質標準。

## 🎯 PostgreSQL 特定審查領域

### JSONB 最佳實踐
```sql
-- ❌ BAD: Inefficient JSONB usage
SELECT * FROM orders WHERE data->>'status' = 'shipped';  -- No index support

-- ✅ GOOD: Indexable JSONB queries
CREATE INDEX idx_orders_status ON orders USING gin((data->'status'));
SELECT * FROM orders WHERE data @> '{"status": "shipped"}';

-- ❌ BAD: Deep nesting without consideration
UPDATE orders SET data = data || '{"shipping":{"tracking":{"number":"123"}}}';

-- ✅ GOOD: Structured JSONB with validation
ALTER TABLE orders ADD CONSTRAINT valid_status 
CHECK (data->>'status' IN ('pending', 'shipped', 'delivered'));
```

### 陣列操作審查
```sql
-- ❌ BAD: Inefficient array operations
SELECT * FROM products WHERE 'electronics' = ANY(categories);  -- No index

-- ✅ GOOD: GIN indexed array queries
CREATE INDEX idx_products_categories ON products USING gin(categories);
SELECT * FROM products WHERE categories @> ARRAY['electronics'];

-- ❌ BAD: Array concatenation in loops
-- This would be inefficient in a function/procedure

-- ✅ GOOD: Bulk array operations
UPDATE products SET categories = categories || ARRAY['new_category']
WHERE id IN (SELECT id FROM products WHERE condition);
```

### PostgreSQL 結構描述設計審查
```sql
-- ❌ BAD: Not using PostgreSQL features
CREATE TABLE users (
    id INTEGER,
    email VARCHAR(255),
    created_at TIMESTAMP
);

-- ✅ GOOD: PostgreSQL-optimized schema
CREATE TABLE users (
    id BIGSERIAL PRIMARY KEY,
    email CITEXT UNIQUE NOT NULL,  -- Case-insensitive email
    created_at TIMESTAMPTZ DEFAULT NOW(),
    metadata JSONB DEFAULT '{}',
    CONSTRAINT valid_email CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$')
);

-- Add JSONB GIN index for metadata queries
CREATE INDEX idx_users_metadata ON users USING gin(metadata);
```

### 自訂類型與域
```sql
-- ❌ BAD: Using generic types for specific data
CREATE TABLE transactions (
    amount DECIMAL(10,2),
    currency VARCHAR(3),
    status VARCHAR(20)
);

-- ✅ GOOD: PostgreSQL custom types
CREATE TYPE currency_code AS ENUM ('USD', 'EUR', 'GBP', 'JPY');
CREATE TYPE transaction_status AS ENUM ('pending', 'completed', 'failed', 'cancelled');
CREATE DOMAIN positive_amount AS DECIMAL(10,2) CHECK (VALUE > 0);

CREATE TABLE transactions (
    amount positive_amount NOT NULL,
    currency currency_code NOT NULL,
    status transaction_status DEFAULT 'pending'
);
```

## 🔍 PostgreSQL 特定反模式

### 效能反模式
- **避免 PostgreSQL 特定索引**：不使用適合資料類型的 GIN/GiST
- **濫用 JSONB**：將 JSONB 當作簡單的字串欄位處理
- **忽視陣列運算子**：使用低效的陣列操作
- **不良的分區鍵選擇**：未能有效利用 PostgreSQL 分區功能

### 結構描述設計問題
- **不使用 ENUM 類型**：對有限值集合使用 VARCHAR
- **忽視限制條件**：缺少用於資料驗證的 CHECK 限制條件
- **錯誤的資料類型**：使用 VARCHAR 而不是 TEXT 或 CITEXT
- **缺失 JSONB 結構**：沒有驗證的非結構化 JSONB

### 函數與觸發器問題
```sql
-- ❌ BAD: Inefficient trigger function
CREATE OR REPLACE FUNCTION update_modified_time()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();  -- Should use TIMESTAMPTZ
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- ✅ GOOD: Optimized trigger function
CREATE OR REPLACE FUNCTION update_modified_time()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Set trigger to fire only when needed
CREATE TRIGGER update_modified_time_trigger
    BEFORE UPDATE ON table_name
    FOR EACH ROW
    WHEN (OLD.* IS DISTINCT FROM NEW.*)
    EXECUTE FUNCTION update_modified_time();
```

## 📊 PostgreSQL 擴充套件使用審查

### 擴充套件最佳實踐
```sql
-- ✅ Check if extension exists before creating
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pgcrypto";
CREATE EXTENSION IF NOT EXISTS "pg_trgm";

-- ✅ Use extensions appropriately
-- UUID generation
SELECT uuid_generate_v4();

-- Password hashing
SELECT crypt('password', gen_salt('bf'));

-- Fuzzy text matching
SELECT word_similarity('postgres', 'postgre');
```

## 🛡️ PostgreSQL 安全性審查

### 列級安全性（Row Level Security, RLS）
```sql
-- ✅ GOOD: Implementing RLS
ALTER TABLE sensitive_data ENABLE ROW LEVEL SECURITY;

CREATE POLICY user_data_policy ON sensitive_data
    FOR ALL TO application_role
    USING (user_id = current_setting('app.current_user_id')::INTEGER);
```

### 權限管理
```sql
-- ❌ BAD: Overly broad permissions
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO app_user;

-- ✅ GOOD: Granular permissions
GRANT SELECT, INSERT, UPDATE ON specific_table TO app_user;
GRANT USAGE ON SEQUENCE specific_table_id_seq TO app_user;
```

## 🎯 PostgreSQL 程式碼品質檢查清單

### 結構描述設計
- [ ] 使用適當的 PostgreSQL 資料類型（CITEXT、JSONB、陣列）
- [ ] 利用 ENUM 類型來限制值
- [ ] 實作適當的 CHECK 限制條件
- [ ] 使用 TIMESTAMPTZ 而不是 TIMESTAMP
- [ ] 定義自訂域以供重複使用的限制條件

### 效能考量
- [ ] 適當的索引類型（GIN 用於 JSONB/陣列，GiST 用於範圍）
- [ ] JSONB 查詢使用包含運算子（@>, ?）
- [ ] 陣列操作使用 PostgreSQL 特定運算子
- [ ] 適當使用視窗函數與 CTE
- [ ] 有效使用 PostgreSQL 特定函數

### PostgreSQL 功能利用
- [ ] 在適當的地方使用擴充套件
- [ ] 在有益時在 PL/pgSQL 中實作儲存程序
- [ ] 利用 PostgreSQL 的進階 SQL 功能
- [ ] 使用 PostgreSQL 特定的最佳化技術
- [ ] 在函數中實作適當的錯誤處理

### 安全性與合規
- [ ] 在需要時實作列級安全性（RLS）
- [ ] 適當的角色與權限管理
- [ ] 使用 PostgreSQL 的內建加密函數
- [ ] 使用 PostgreSQL 功能實作稽核軌跡

## 📝 PostgreSQL 特定審查指南

1. **資料類型最佳化**：確保 PostgreSQL 特定類型被適當地使用
2. **索引策略**：審查索引類型，確保利用 PostgreSQL 特定索引
3. **JSONB 結構**：驗證 JSONB 結構描述設計與查詢模式
4. **函數品質**：審查 PL/pgSQL 函數的效率與最佳實踐
5. **擴充套件使用**：驗證 PostgreSQL 擴充套件的適當使用
6. **效能功能**：檢查 PostgreSQL 進階功能的利用
7. **安全性實作**：審查 PostgreSQL 特定安全功能

專注於 PostgreSQL 的獨特功能，確保程式碼利用 PostgreSQL 的優勢，而不是將其當作一般的 SQL 資料庫。
