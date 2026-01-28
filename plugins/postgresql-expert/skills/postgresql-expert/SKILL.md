---
name: postgresql-expert
description: "PostgreSQL 進階資料庫專家。專精於進階 SQL、JSON/JSONB、全文搜尋、分區表、窗口函數、CTE、索引類型 (B-tree, GIN, GiST)、效能優化。關鍵字: postgresql, postgres, psql, jsonb, full text search, cte, window function, 全文搜尋, 窗口函數"
---

# PostgreSQL Expert

You are a PostgreSQL Expert specializing in advanced SQL features, JSONB operations, full-text search, performance tuning, and PostgreSQL-specific capabilities.

## Overview

PostgreSQL is a powerful, open-source object-relational database system with advanced features like JSONB, array types, full-text search, and window functions. It's known for its standards compliance and extensibility.

**Core capabilities:**
- Advanced SQL (CTEs, Window Functions, Lateral Joins)
- JSONB and semi-structured data
- Full-text search
- Table partitioning
- Advanced indexing (GIN, GiST, BRIN)
- Procedural languages (PL/pgSQL)
- Performance optimization

## When to use this skill

Activate this skill when users:
- Work with PostgreSQL (關鍵字: "postgresql", "postgres", "psql")
- Use JSON data (關鍵字: "jsonb", "json", "json query")
- Need full-text search (關鍵字: "full text search", "全文搜尋", "tsvector")
- Use window functions (關鍵字: "window function", "窗口函數", "rank", "row_number")
- Implement CTEs (關鍵字: "cte", "with", "recursive query")
- Need advanced queries (關鍵字: "lateral join", "array", "unnest")

## Core Knowledge Areas

### 1. JSONB Operations

**Create and Query JSONB:**
```sql
-- Create table with JSONB
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    attributes JSONB,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Insert JSONB data
INSERT INTO products (name, attributes) VALUES
('Laptop', '{"brand": "Dell", "cpu": "Intel i7", "ram": 16, "price": 1200}'),
('Phone', '{"brand": "Apple", "model": "iPhone 14", "storage": 256, "price": 999}');

-- Query JSONB with operators
-- -> returns JSON, ->> returns text
SELECT name, attributes->'brand' AS brand
FROM products;

SELECT name, attributes->>'brand' AS brand_text
FROM products;

-- Query nested JSON
SELECT name, attributes->'specs'->>'cpu' AS cpu
FROM products;

-- Check if key exists
SELECT * FROM products WHERE attributes ? 'brand';

-- Check if any keys exist
SELECT * FROM products WHERE attributes ?| array['brand', 'model'];

-- Check if all keys exist
SELECT * FROM products WHERE attributes ?& array['brand', 'price'];

-- Query by value
SELECT * FROM products WHERE attributes->>'brand' = 'Apple';

-- Query with numeric comparison
SELECT * FROM products
WHERE (attributes->>'price')::numeric > 1000;

-- Update JSONB
UPDATE products
SET attributes = attributes || '{"warranty": "2 years"}'::jsonb
WHERE id = 1;

-- Remove key
UPDATE products
SET attributes = attributes - 'warranty'
WHERE id = 1;

-- Index JSONB
CREATE INDEX idx_attributes ON products USING GIN (attributes);
CREATE INDEX idx_brand ON products ((attributes->>'brand'));
```

### 2. Full-Text Search

**Basic Full-Text Search:**
```sql
-- Create table with tsvector
CREATE TABLE articles (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200),
    content TEXT,
    search_vector tsvector
);

-- Add full-text search column
ALTER TABLE articles
ADD COLUMN search_vector tsvector
GENERATED ALWAYS AS (
    to_tsvector('english', coalesce(title, '') || ' ' || coalesce(content, ''))
) STORED;

-- Create GIN index
CREATE INDEX idx_search_vector ON articles USING GIN (search_vector);

-- Search
SELECT title, content
FROM articles
WHERE search_vector @@ to_tsquery('english', 'postgresql & database');

-- Search with ranking
SELECT title,
       ts_rank(search_vector, query) AS rank
FROM articles,
     to_tsquery('english', 'postgresql & database') query
WHERE search_vector @@ query
ORDER BY rank DESC;

-- Phrase search
SELECT * FROM articles
WHERE search_vector @@ phraseto_tsquery('english', 'relational database');

-- Highlight search results
SELECT title,
       ts_headline('english', content, to_tsquery('postgresql'), 'MaxWords=20')
FROM articles
WHERE search_vector @@ to_tsquery('postgresql');
```

### 3. Window Functions

**Common Window Functions:**
```sql
-- ROW_NUMBER: Unique sequential number
SELECT
    employee_name,
    department,
    salary,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS row_num
FROM employees;

-- RANK: Ranking with gaps
SELECT
    employee_name,
    department,
    salary,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rank
FROM employees;

-- DENSE_RANK: Ranking without gaps
SELECT
    employee_name,
    salary,
    DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank
FROM employees;

-- LAG/LEAD: Access previous/next row
SELECT
    order_date,
    total_amount,
    LAG(total_amount) OVER (ORDER BY order_date) AS previous_amount,
    LEAD(total_amount) OVER (ORDER BY order_date) AS next_amount
FROM orders;

-- Running total
SELECT
    order_date,
    amount,
    SUM(amount) OVER (ORDER BY order_date) AS running_total
FROM orders;

-- Moving average
SELECT
    date,
    value,
    AVG(value) OVER (
        ORDER BY date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS moving_avg_7days
FROM metrics;

-- NTILE: Divide into buckets
SELECT
    customer_name,
    total_purchases,
    NTILE(4) OVER (ORDER BY total_purchases DESC) AS quartile
FROM customers;
```

### 4. Common Table Expressions (CTEs)

**Basic CTE:**
```sql
-- Simple CTE
WITH high_value_orders AS (
    SELECT customer_id, order_id, total_amount
    FROM orders
    WHERE total_amount > 1000
)
SELECT c.customer_name, h.order_id, h.total_amount
FROM customers c
JOIN high_value_orders h ON c.id = h.customer_id;

-- Multiple CTEs
WITH
monthly_sales AS (
    SELECT
        DATE_TRUNC('month', order_date) AS month,
        SUM(total_amount) AS total
    FROM orders
    GROUP BY month
),
avg_sales AS (
    SELECT AVG(total) AS avg_monthly_sales
    FROM monthly_sales
)
SELECT ms.month, ms.total, a.avg_monthly_sales
FROM monthly_sales ms, avg_sales a
WHERE ms.total > a.avg_monthly_sales;
```

**Recursive CTE:**
```sql
-- Organization hierarchy
WITH RECURSIVE org_tree AS (
    -- Base case: top-level employees
    SELECT id, name, manager_id, 0 AS level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive case
    SELECT e.id, e.name, e.manager_id, ot.level + 1
    FROM employees e
    JOIN org_tree ot ON e.manager_id = ot.id
)
SELECT * FROM org_tree ORDER BY level, name;

-- Generate series
WITH RECURSIVE date_series AS (
    SELECT DATE '2024-01-01' AS date
    UNION ALL
    SELECT date + INTERVAL '1 day'
    FROM date_series
    WHERE date < DATE '2024-12-31'
)
SELECT * FROM date_series;
```

### 5. Array Operations

**Array Queries:**
```sql
-- Create table with array
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    tags TEXT[],
    prices NUMERIC[]
);

-- Insert arrays
INSERT INTO products (name, tags, prices) VALUES
('Laptop', ARRAY['electronics', 'computer', 'portable'], ARRAY[999.99, 1099.99]),
('Desk', ARRAY['furniture', 'office'], ARRAY[299.99]);

-- Query arrays
SELECT * FROM products WHERE 'electronics' = ANY(tags);
SELECT * FROM products WHERE tags @> ARRAY['electronics'];
SELECT * FROM products WHERE tags && ARRAY['computer', 'furniture'];

-- Array functions
SELECT name, array_length(tags, 1) AS tag_count FROM products;
SELECT name, unnest(tags) AS tag FROM products;
SELECT name, array_to_string(tags, ', ') AS tags_str FROM products;

-- Index arrays
CREATE INDEX idx_tags ON products USING GIN (tags);
```

### 6. Table Partitioning

**Range Partitioning:**
```sql
-- Create partitioned table
CREATE TABLE orders (
    id BIGSERIAL,
    order_date DATE NOT NULL,
    customer_id INTEGER,
    total_amount NUMERIC(10,2)
) PARTITION BY RANGE (order_date);

-- Create partitions
CREATE TABLE orders_2024_q1 PARTITION OF orders
    FOR VALUES FROM ('2024-01-01') TO ('2024-04-01');

CREATE TABLE orders_2024_q2 PARTITION OF orders
    FOR VALUES FROM ('2024-04-01') TO ('2024-07-01');

CREATE TABLE orders_2024_q3 PARTITION OF orders
    FOR VALUES FROM ('2024-07-01') TO ('2024-10-01');

-- Create indexes on partitions
CREATE INDEX idx_orders_2024_q1_customer ON orders_2024_q1(customer_id);

-- Insert (automatically routes to correct partition)
INSERT INTO orders (order_date, customer_id, total_amount)
VALUES ('2024-03-15', 123, 500.00);

-- Query (partition pruning)
SELECT * FROM orders WHERE order_date = '2024-03-15';
-- Only scans orders_2024_q1 partition
```

**List Partitioning:**
```sql
CREATE TABLE users (
    id BIGSERIAL,
    username VARCHAR(50),
    country_code CHAR(2)
) PARTITION BY LIST (country_code);

CREATE TABLE users_us PARTITION OF users FOR VALUES IN ('US');
CREATE TABLE users_uk PARTITION OF users FOR VALUES IN ('UK');
CREATE TABLE users_other PARTITION OF users DEFAULT;
```

### 7. Advanced Indexing

**Index Types:**
```sql
-- B-tree (default, good for equality and range)
CREATE INDEX idx_name ON users(name);

-- GIN (good for JSONB, arrays, full-text)
CREATE INDEX idx_tags ON products USING GIN (tags);
CREATE INDEX idx_jsonb ON products USING GIN (attributes);

-- GiST (good for geometric data, full-text)
CREATE INDEX idx_location ON stores USING GIST (location);

-- BRIN (good for large tables with natural ordering)
CREATE INDEX idx_created ON logs USING BRIN (created_at);

-- Partial index
CREATE INDEX idx_active_users ON users(name) WHERE status = 'ACTIVE';

-- Expression index
CREATE INDEX idx_lower_email ON users(LOWER(email));

-- Covering index (include columns)
CREATE INDEX idx_user_orders ON orders(user_id) INCLUDE (total_amount, order_date);
```

### 8. Performance Optimization

**Query Analysis:**
```sql
-- EXPLAIN ANALYZE
EXPLAIN ANALYZE
SELECT u.name, COUNT(o.id) AS order_count
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name;

-- Look for:
-- Seq Scan (bad for large tables)
-- Index Scan (good)
-- Bitmap Index Scan (good for multiple conditions)
-- Nested Loop (can be slow)
-- Hash Join (good for large joins)

-- Check index usage
SELECT
    schemaname,
    tablename,
    indexname,
    idx_scan,
    idx_tup_read,
    idx_tup_fetch
FROM pg_stat_user_indexes
WHERE idx_scan = 0;  -- Unused indexes
```

**VACUUM and ANALYZE:**
```sql
-- Vacuum table
VACUUM users;

-- Vacuum with analyze
VACUUM ANALYZE users;

-- Full vacuum (requires exclusive lock)
VACUUM FULL users;

-- Auto-vacuum settings (postgresql.conf)
autovacuum = on
autovacuum_vacuum_scale_factor = 0.1
autovacuum_analyze_scale_factor = 0.05
```

## Best Practices

### 1. Use JSONB over JSON
```sql
-- JSONB is faster and supports indexing
CREATE TABLE data (info JSONB);  -- Good
CREATE TABLE data (info JSON);   -- Avoid
```

### 2. Proper Indexing
```sql
-- Index foreign keys
CREATE INDEX idx_orders_user_id ON orders(user_id);

-- Partial indexes for filtered queries
CREATE INDEX idx_pending_orders ON orders(created_at)
WHERE status = 'PENDING';

-- Expression indexes
CREATE INDEX idx_email_lower ON users(LOWER(email));
```

### 3. Use CTEs for Complex Queries
```sql
-- More readable than subqueries
WITH filtered_orders AS (
    SELECT * FROM orders WHERE total_amount > 100
),
customer_totals AS (
    SELECT customer_id, SUM(total_amount) AS total
    FROM filtered_orders
    GROUP BY customer_id
)
SELECT * FROM customer_totals WHERE total > 1000;
```

### 4. Connection Pooling
```bash
# Use PgBouncer for connection pooling
# pgbouncer.ini
[databases]
mydb = host=localhost port=5432 dbname=mydb

[pgbouncer]
listen_port = 6432
listen_addr = *
auth_type = md5
pool_mode = transaction
max_client_conn = 1000
default_pool_size = 20
```

## Quick Reference

### Data Types
```sql
-- Numeric
SMALLINT, INTEGER, BIGINT
DECIMAL, NUMERIC
REAL, DOUBLE PRECISION

-- String
CHAR(n), VARCHAR(n), TEXT

-- Binary
BYTEA

-- Date/Time
DATE, TIME, TIMESTAMP, TIMESTAMPTZ, INTERVAL

-- Boolean
BOOLEAN

-- JSON
JSON, JSONB

-- Array
INTEGER[], TEXT[]

-- UUID
UUID

-- Network
INET, CIDR, MACADDR
```

### Essential Commands
```sql
-- Database
CREATE DATABASE mydb;
\c mydb
\l  -- List databases

-- Tables
\dt  -- List tables
\d table_name  -- Describe table
\di  -- List indexes

-- Users
CREATE USER myuser WITH PASSWORD 'password';
GRANT ALL PRIVILEGES ON DATABASE mydb TO myuser;

-- Backup/Restore
pg_dump mydb > backup.sql
psql mydb < backup.sql
```

### Useful Functions
```sql
-- String
CONCAT, LOWER, UPPER, TRIM, SUBSTRING, LENGTH

-- Date
NOW(), CURRENT_DATE, DATE_TRUNC, AGE, EXTRACT

-- JSON
json_build_object, json_agg, jsonb_set, jsonb_each

-- Array
array_agg, unnest, array_length, array_to_string

-- Aggregate
COUNT, SUM, AVG, MAX, MIN, STRING_AGG

-- Window
ROW_NUMBER, RANK, LAG, LEAD, FIRST_VALUE
```

---

**Remember:** PostgreSQL's advanced features like JSONB, CTEs, and window functions make it ideal for complex data modeling and analytics.
