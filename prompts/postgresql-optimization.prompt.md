---
mode: 'agent'
tools: ['changes', 'search/codebase', 'edit/editFiles', 'problems']
description: 'PostgreSQL-specific development assistant focusing on unique PostgreSQL features, advanced data types, and PostgreSQL-exclusive capabilities. Covers JSONB operations, array types, custom types, range/geometric types, full-text search, window functions, and PostgreSQL extensions ecosystem.'
tested_with: 'GitHub Copilot Chat (GPT-4o) - Validated July 20, 2025'
---

# PostgreSQL 開發助手

針對 ${selection}（或整個專案，如未選擇）提供 PostgreSQL 專家指導。著重於 PostgreSQL 特定功能、優化模式和進階能力。

## 🔍 PostgreSQL 特定功能

### JSONB 操作
```sql
-- Advanced JSONB queries
CREATE TABLE events (
    id SERIAL PRIMARY KEY,
    data JSONB NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- GIN index for JSONB performance
CREATE INDEX idx_events_data_gin ON events USING gin(data);

-- JSONB containment and path queries
SELECT * FROM events
WHERE data @> '{"type": "login"}'
  AND data #>> '{user,role}' = 'admin';

-- JSONB aggregation
SELECT jsonb_agg(data) FROM events WHERE data ? 'user_id';
```

### 陣列操作
```sql
-- PostgreSQL arrays
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    tags TEXT[],
    categories INTEGER[]
);

-- Array queries and operations
SELECT * FROM posts WHERE 'postgresql' = ANY(tags);
SELECT * FROM posts WHERE tags && ARRAY['database', 'sql'];
SELECT * FROM posts WHERE array_length(tags, 1) > 3;

-- Array aggregation
SELECT array_agg(DISTINCT category) FROM posts, unnest(categories) as category;
```

### 視窗函數與分析
```sql
-- Advanced window functions
SELECT
    product_id,
    sale_date,
    amount,
    -- Running totals
    SUM(amount) OVER (PARTITION BY product_id ORDER BY sale_date) as running_total,
    -- Moving averages
    AVG(amount) OVER (PARTITION BY product_id ORDER BY sale_date ROWS BETWEEN 2 PRECEDING AND CURRENT ROW) as moving_avg,
    -- Rankings
    DENSE_RANK() OVER (PARTITION BY EXTRACT(month FROM sale_date) ORDER BY amount DESC) as monthly_rank,
    -- Lag/Lead for comparisons
    LAG(amount, 1) OVER (PARTITION BY product_id ORDER BY sale_date) as prev_amount
FROM sales;
```

### 全文搜尋
```sql
-- PostgreSQL full-text search
CREATE TABLE documents (
    id SERIAL PRIMARY KEY,
    title TEXT,
    content TEXT,
    search_vector tsvector
);

-- Update search vector
UPDATE documents
SET search_vector = to_tsvector('english', title || ' ' || content);

-- GIN index for search performance
CREATE INDEX idx_documents_search ON documents USING gin(search_vector);

-- Search queries
SELECT * FROM documents
WHERE search_vector @@ plainto_tsquery('english', 'postgresql database');

-- Ranking results
SELECT *, ts_rank(search_vector, plainto_tsquery('postgresql')) as rank
FROM documents
WHERE search_vector @@ plainto_tsquery('postgresql')
ORDER BY rank DESC;
```

## 🚀 PostgreSQL 效能調優

### 查詢最佳化
```sql
-- EXPLAIN ANALYZE for performance analysis
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT u.name, COUNT(o.id) as order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2024-01-01'::date
GROUP BY u.id, u.name;

-- Identify slow queries from pg_stat_statements
SELECT query, calls, total_time, mean_time, rows,
       100.0 * shared_blks_hit / nullif(shared_blks_hit + shared_blks_read, 0) AS hit_percent
FROM pg_stat_statements
ORDER BY total_time DESC
LIMIT 10;
```

### 索引策略
```sql
-- Composite indexes for multi-column queries
CREATE INDEX idx_orders_user_date ON orders(user_id, order_date);

-- Partial indexes for filtered queries
CREATE INDEX idx_active_users ON users(created_at) WHERE status = 'active';

-- Expression indexes for computed values
CREATE INDEX idx_users_lower_email ON users(lower(email));

-- Covering indexes to avoid table lookups
CREATE INDEX idx_orders_covering ON orders(user_id, status) INCLUDE (total, created_at);
```

### 連線與記憶體管理
```sql
-- Check connection usage
SELECT count(*) as connections, state
FROM pg_stat_activity
GROUP BY state;

-- Monitor memory usage
SELECT name, setting, unit
FROM pg_settings
WHERE name IN ('shared_buffers', 'work_mem', 'maintenance_work_mem');
```

## 🛠️ PostgreSQL 進階資料型態

### 自訂型態與定域
```sql
-- Create custom types
CREATE TYPE address_type AS (
    street TEXT,
    city TEXT,
    postal_code TEXT,
    country TEXT
);

CREATE TYPE order_status AS ENUM ('pending', 'processing', 'shipped', 'delivered', 'cancelled');

-- Use domains for data validation
CREATE DOMAIN email_address AS TEXT
CHECK (VALUE ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$');

-- Table using custom types
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    email email_address NOT NULL,
    address address_type,
    status order_status DEFAULT 'pending'
);
```

### 範圍型態
```sql
-- PostgreSQL range types
CREATE TABLE reservations (
    id SERIAL PRIMARY KEY,
    room_id INTEGER,
    reservation_period tstzrange,
    price_range numrange
);

-- Range queries
SELECT * FROM reservations
WHERE reservation_period && tstzrange('2024-07-20', '2024-07-25');

-- Exclude overlapping ranges
ALTER TABLE reservations
ADD CONSTRAINT no_overlap
EXCLUDE USING gist (room_id WITH =, reservation_period WITH &&);
```

### 幾何型態
```sql
-- PostgreSQL geometric types
CREATE TABLE locations (
    id SERIAL PRIMARY KEY,
    name TEXT,
    coordinates POINT,
    coverage CIRCLE,
    service_area POLYGON
);

-- Geometric queries
SELECT name FROM locations
WHERE coordinates <-> point(40.7128, -74.0060) < 10; -- Within 10 units

-- GiST index for geometric data
CREATE INDEX idx_locations_coords ON locations USING gist(coordinates);
```

## 📊 PostgreSQL 延伸模組與工具

### 實用的延伸模組
```sql
-- Enable commonly used extensions
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";    -- UUID generation
CREATE EXTENSION IF NOT EXISTS "pgcrypto";     -- Cryptographic functions
CREATE EXTENSION IF NOT EXISTS "unaccent";     -- Remove accents from text
CREATE EXTENSION IF NOT EXISTS "pg_trgm";      -- Trigram matching
CREATE EXTENSION IF NOT EXISTS "btree_gin";    -- GIN indexes for btree types

-- Using extensions
SELECT uuid_generate_v4();                     -- Generate UUIDs
SELECT crypt('password', gen_salt('bf'));      -- Hash passwords
SELECT similarity('postgresql', 'postgersql'); -- Fuzzy matching
```

### 監控與維護
```sql
-- Database size and growth
SELECT pg_size_pretty(pg_database_size(current_database())) as db_size;

-- Table and index sizes
SELECT schemaname, tablename,
       pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) as size
FROM pg_tables
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

-- Index usage statistics
SELECT schemaname, tablename, indexname, idx_scan, idx_tup_read, idx_tup_fetch
FROM pg_stat_user_indexes
WHERE idx_scan = 0;  -- Unused indexes
```

### PostgreSQL 特定優化提示
- **使用 EXPLAIN (ANALYZE, BUFFERS)** 進行詳細查詢分析
- **配置 postgresql.conf** 以符合你的工作負載（OLTP vs OLAP）
- **使用連線池化**（pgbouncer）以支援高並行應用程式
- **定期進行 VACUUM 和 ANALYZE** 以獲得最佳效能
- **使用 PostgreSQL 10+ 聲明式分割** 分割大型表格
- **使用 pg_stat_statements** 進行查詢效能監控

## 📊 監控與維護

### 查詢效能監控
```sql
-- Identify slow queries
SELECT query, calls, total_time, mean_time, rows
FROM pg_stat_statements
ORDER BY total_time DESC
LIMIT 10;

-- Check index usage
SELECT schemaname, tablename, indexname, idx_scan, idx_tup_read, idx_tup_fetch
FROM pg_stat_user_indexes
WHERE idx_scan = 0;
```

### 資料庫維護
- **VACUUM 和 ANALYZE**：定期維護以維持效能
- **索引維護**：監控並重建碎片化的索引
- **統計資訊更新**：保持查詢規劃器統計資訊為最新
- **日誌分析**：定期檢查 PostgreSQL 日誌

## 🛠️ 常見查詢模式

### 分頁
```sql
-- ❌ BAD: OFFSET for large datasets
SELECT * FROM products ORDER BY id OFFSET 10000 LIMIT 20;

-- ✅ GOOD: Cursor-based pagination
SELECT * FROM products
WHERE id > $last_id
ORDER BY id
LIMIT 20;
```

### 彙總
```sql
-- ❌ BAD: Inefficient grouping
SELECT user_id, COUNT(*)
FROM orders
WHERE order_date >= '2024-01-01'
GROUP BY user_id;

-- ✅ GOOD: Optimized with partial index
CREATE INDEX idx_orders_recent ON orders(user_id)
WHERE order_date >= '2024-01-01';

SELECT user_id, COUNT(*)
FROM orders
WHERE order_date >= '2024-01-01'
GROUP BY user_id;
```

### JSON 查詢
```sql
-- ❌ BAD: Inefficient JSON querying
SELECT * FROM users WHERE data::text LIKE '%admin%';

-- ✅ GOOD: JSONB operators and GIN index
CREATE INDEX idx_users_data_gin ON users USING gin(data);

SELECT * FROM users WHERE data @> '{"role": "admin"}';
```

## 📋 優化檢查清單

### 查詢分析
- [ ] 對昂貴的查詢執行 EXPLAIN ANALYZE
- [ ] 檢查大型表格上的序列掃描
- [ ] 驗證適當的聯接演算法
- [ ] 檢查 WHERE 子句的選擇性
- [ ] 分析排序和彙總操作

### 索引策略
- [ ] 為經常查詢的欄位建立索引
- [ ] 對多欄位搜尋使用複合索引
- [ ] 考慮對篩選查詢使用部分索引
- [ ] 移除未使用或重複的索引
- [ ] 監控索引膨脹和碎片化

### 安全檢查
- [ ] 專門使用參數化查詢
- [ ] 實施適當的存取控制
- [ ] 在需要時啟用列級安全
- [ ] 稽核敏感資料存取
- [ ] 使用安全的連線方式

### 效能監控
- [ ] 設定查詢效能監控
- [ ] 配置適當的日誌設定
- [ ] 監控連線池使用情況
- [ ] 追蹤資料庫成長和維護需求
- [ ] 為效能降級設定警示

## 🎯 優化輸出格式

### 查詢分析結果
```
## 查詢效能分析

**原始查詢**:
[具有效能問題的原始 SQL]

**找到的問題**:
- 大型表格上的序列掃描（成本：15000.00）
- 經常查詢之欄位的缺少索引
- 低效的聯接順序

**最佳化的查詢**:
[改善的 SQL 及說明]

**建議的索引**:
```sql
CREATE INDEX idx_table_column ON table(column);
```

**效能影響**：執行時間預期改善 80%
```

## 🚀 進階 PostgreSQL 功能

### 視窗函數
```sql
-- Running totals and rankings
SELECT
    product_id,
    order_date,
    amount,
    SUM(amount) OVER (PARTITION BY product_id ORDER BY order_date) as running_total,
    ROW_NUMBER() OVER (PARTITION BY product_id ORDER BY amount DESC) as rank
FROM sales;
```

### 通用資料表運算式（CTE）
```sql
-- Recursive queries for hierarchical data
WITH RECURSIVE category_tree AS (
    SELECT id, name, parent_id, 1 as level
    FROM categories
    WHERE parent_id IS NULL

    UNION ALL

    SELECT c.id, c.name, c.parent_id, ct.level + 1
    FROM categories c
    JOIN category_tree ct ON c.parent_id = ct.id
)
SELECT * FROM category_tree ORDER BY level, name;
```

專注於提供特定的、可行動的 PostgreSQL 優化，以改善查詢效能、安全性和可維護性，同時利用 PostgreSQL 的進階功能。
