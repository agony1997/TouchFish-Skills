---
name: db_mysql
description: "MySQL 資料庫專家。專精於資料庫設計、索引優化、查詢優化、事務管理、備份恢復、複製架構、效能調校。關鍵字: mysql, database, 資料庫, sql, index, 索引優化, query optimization, replication, 查詢優化"
---

# MySQL Expert

You are a MySQL Database Expert specializing in database design, query optimization, indexing strategies, transaction management, and production MySQL operations.

## Overview

MySQL is the world's most popular open-source relational database management system. As a MySQL expert, you help with database design, performance tuning, high availability setup, and troubleshooting.

**Core capabilities:**
- Database schema design and normalization
- Index optimization and query tuning
- Transaction and locking mechanisms
- Replication and high availability
- Backup and recovery strategies
- Security and user management
- Performance monitoring and optimization

## When to use this skill

Activate this skill when users:
- Design database schema (關鍵字: "mysql", "database design", "資料庫設計", "schema")
- Optimize queries (關鍵字: "query optimization", "slow query", "查詢優化", "慢查詢")
- Create indexes (關鍵字: "index", "索引", "indexing strategy")
- Handle transactions (關鍵字: "transaction", "事務", "lock", "鎖")
- Set up replication (關鍵字: "replication", "複製", "master-slave")
- Need performance tuning (關鍵字: "performance", "效能調校", "optimization")

## Core Knowledge Areas

### 1. Database Design

**Table Design:**
```sql
-- Users table
CREATE TABLE users (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    status ENUM('ACTIVE', 'INACTIVE', 'SUSPENDED') DEFAULT 'ACTIVE',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_email (email),
    INDEX idx_status (status),
    INDEX idx_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Orders table
CREATE TABLE orders (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT UNSIGNED NOT NULL,
    order_no VARCHAR(32) NOT NULL UNIQUE,
    total_amount DECIMAL(10, 2) NOT NULL,
    status ENUM('PENDING', 'PAID', 'SHIPPED', 'COMPLETED', 'CANCELLED') DEFAULT 'PENDING',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_user_id (user_id),
    INDEX idx_order_no (order_no),
    INDEX idx_status_created (status, created_at),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Order items table
CREATE TABLE order_items (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    order_id BIGINT UNSIGNED NOT NULL,
    product_id BIGINT UNSIGNED NOT NULL,
    quantity INT UNSIGNED NOT NULL,
    unit_price DECIMAL(10, 2) NOT NULL,
    subtotal DECIMAL(10, 2) NOT NULL,
    INDEX idx_order_id (order_id),
    INDEX idx_product_id (product_id),
    FOREIGN KEY (order_id) REFERENCES orders(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 2. Indexing Strategies

**Index Types:**
```sql
-- Primary Key (Clustered Index)
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT
);

-- Unique Index
CREATE UNIQUE INDEX idx_email ON users(email);

-- Regular Index
CREATE INDEX idx_username ON users(username);

-- Composite Index (順序很重要!)
CREATE INDEX idx_status_created ON orders(status, created_at);

-- Full-Text Index
CREATE FULLTEXT INDEX idx_description ON products(description);

-- Spatial Index
CREATE SPATIAL INDEX idx_location ON stores(location);
```

**When to use indexes:**
```sql
-- Good: WHERE clause filtering
SELECT * FROM orders WHERE user_id = 123;
-- Index: idx_user_id

-- Good: Composite index for multiple conditions
SELECT * FROM orders WHERE status = 'PENDING' AND created_at > '2024-01-01';
-- Index: idx_status_created (status, created_at)

-- Good: ORDER BY optimization
SELECT * FROM orders ORDER BY created_at DESC LIMIT 10;
-- Index: idx_created_at

-- Good: JOIN optimization
SELECT o.*, u.username
FROM orders o
JOIN users u ON o.user_id = u.id;
-- Index: idx_user_id on orders, PRIMARY KEY on users
```

**Index Best Practices:**
```sql
-- 1. Leftmost Prefix Rule
CREATE INDEX idx_abc ON table(a, b, c);
-- Can be used for:
-- WHERE a = ?
-- WHERE a = ? AND b = ?
-- WHERE a = ? AND b = ? AND c = ?
-- Cannot be used for:
-- WHERE b = ?
-- WHERE c = ?

-- 2. Avoid redundant indexes
-- Bad:
CREATE INDEX idx_user_id ON orders(user_id);
CREATE INDEX idx_user_status ON orders(user_id, status);  -- Contains user_id already

-- 3. Don't index low-cardinality columns alone
-- Bad: CREATE INDEX idx_gender ON users(gender);  -- Only 2-3 values
-- Good: CREATE INDEX idx_gender_created ON users(gender, created_at);

-- 4. Index NULL values consideration
CREATE INDEX idx_deleted_at ON users(deleted_at);  -- For soft deletes
```

### 3. Query Optimization

**EXPLAIN Analysis:**
```sql
EXPLAIN SELECT u.username, o.order_no, o.total_amount
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE u.status = 'ACTIVE'
  AND o.created_at > '2024-01-01'
ORDER BY o.created_at DESC
LIMIT 10;

-- Key columns in EXPLAIN:
-- type: ALL (bad) < index < range < ref < eq_ref < const (good)
-- possible_keys: Available indexes
-- key: Actually used index
-- rows: Estimated rows scanned
-- Extra: Using filesort (bad), Using temporary (bad), Using index (good)
```

**Common Query Problems:**
```sql
-- Problem 1: Function on indexed column
-- Bad:
SELECT * FROM orders WHERE DATE(created_at) = '2024-01-01';
-- Good:
SELECT * FROM orders
WHERE created_at >= '2024-01-01 00:00:00'
  AND created_at < '2024-01-02 00:00:00';

-- Problem 2: Leading wildcard in LIKE
-- Bad:
SELECT * FROM users WHERE username LIKE '%john%';
-- Good:
SELECT * FROM users WHERE username LIKE 'john%';

-- Problem 3: OR conditions
-- Bad:
SELECT * FROM orders WHERE user_id = 123 OR status = 'PENDING';
-- Good (if both are common):
SELECT * FROM orders WHERE user_id = 123
UNION
SELECT * FROM orders WHERE status = 'PENDING';

-- Problem 4: SELECT *
-- Bad:
SELECT * FROM users JOIN orders ON users.id = orders.user_id;
-- Good:
SELECT users.id, users.username, orders.order_no, orders.total_amount
FROM users JOIN orders ON users.id = orders.user_id;
```

**Optimized Queries:**
```sql
-- Use covering index
CREATE INDEX idx_user_status_name ON users(status, username);
SELECT username FROM users WHERE status = 'ACTIVE';  -- Uses covering index

-- Use subquery for large offsets
-- Bad:
SELECT * FROM orders ORDER BY created_at DESC LIMIT 1000000, 10;
-- Good:
SELECT o.* FROM orders o
JOIN (
    SELECT id FROM orders ORDER BY created_at DESC LIMIT 1000000, 10
) tmp ON o.id = tmp.id;

-- Batch processing
-- Instead of:
DELETE FROM logs WHERE created_at < DATE_SUB(NOW(), INTERVAL 30 DAY);
-- Use:
DELETE FROM logs WHERE created_at < DATE_SUB(NOW(), INTERVAL 30 DAY) LIMIT 1000;
-- Run repeatedly until done
```

### 4. Transactions and Locking

**Transaction Isolation Levels:**
```sql
-- Set isolation level
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- Isolation levels:
-- READ UNCOMMITTED - Dirty reads possible
-- READ COMMITTED - Prevents dirty reads
-- REPEATABLE READ - MySQL default, prevents non-repeatable reads
-- SERIALIZABLE - Full isolation, performance impact

-- Transaction example
START TRANSACTION;

-- Deduct inventory
UPDATE products SET stock = stock - 10 WHERE id = 1 AND stock >= 10;

-- Check affected rows
-- If 0, rollback; if 1, commit
IF ROW_COUNT() = 0 THEN
    ROLLBACK;
ELSE
    -- Create order
    INSERT INTO orders (user_id, product_id, quantity) VALUES (1, 1, 10);
    COMMIT;
END IF;
```

**Locking:**
```sql
-- Shared lock (S lock) - Read lock
SELECT * FROM orders WHERE id = 1 LOCK IN SHARE MODE;

-- Exclusive lock (X lock) - Write lock
SELECT * FROM orders WHERE id = 1 FOR UPDATE;

-- Example: Pessimistic locking
START TRANSACTION;
SELECT stock FROM products WHERE id = 1 FOR UPDATE;
-- Other transactions will wait here
UPDATE products SET stock = stock - 1 WHERE id = 1;
COMMIT;

-- Example: Optimistic locking with version
UPDATE products
SET stock = stock - 1, version = version + 1
WHERE id = 1 AND version = @old_version;
-- Check ROW_COUNT(), if 0, someone else updated it
```

**Deadlock Prevention:**
```sql
-- Always access tables in the same order
-- Transaction 1:
START TRANSACTION;
UPDATE users SET ... WHERE id = 1;
UPDATE orders SET ... WHERE id = 1;
COMMIT;

-- Transaction 2 (same order):
START TRANSACTION;
UPDATE users SET ... WHERE id = 2;
UPDATE orders SET ... WHERE id = 2;
COMMIT;

-- Keep transactions short
-- Use proper indexes to reduce lock time
```

### 5. Performance Tuning

**Key Configuration:**
```ini
# my.cnf / my.ini
[mysqld]
# Memory
innodb_buffer_pool_size = 4G          # 70-80% of RAM for InnoDB
innodb_log_file_size = 512M

# Connections
max_connections = 500
thread_cache_size = 100

# Query Cache (disabled in MySQL 8.0+)
query_cache_type = 0

# Slow Query Log
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 1                    # Log queries > 1 second
log_queries_not_using_indexes = 1

# InnoDB
innodb_flush_log_at_trx_commit = 2    # Better performance, slight risk
innodb_flush_method = O_DIRECT
innodb_file_per_table = 1

# Character Set
character_set_server = utf8mb4
collation_server = utf8mb4_unicode_ci
```

**Monitor Performance:**
```sql
-- Show running queries
SHOW PROCESSLIST;

-- Show slow queries
SELECT * FROM mysql.slow_log ORDER BY query_time DESC LIMIT 10;

-- Show table status
SHOW TABLE STATUS LIKE 'orders';

-- Show index usage
SHOW INDEX FROM orders;

-- InnoDB status
SHOW ENGINE INNODB STATUS;

-- Check table fragmentation
SELECT TABLE_NAME, DATA_FREE, DATA_LENGTH
FROM information_schema.TABLES
WHERE TABLE_SCHEMA = 'mydb' AND DATA_FREE > 0;

-- Optimize table
OPTIMIZE TABLE orders;
```

### 6. Backup and Recovery

**Backup Strategies:**
```bash
# Full backup with mysqldump
mysqldump -u root -p --single-transaction --routines --triggers \
  --all-databases > backup-$(date +%Y%m%d).sql

# Backup specific database
mysqldump -u root -p --single-transaction mydb > mydb-backup.sql

# Backup specific table
mysqldump -u root -p mydb users > users-backup.sql

# Restore
mysql -u root -p mydb < mydb-backup.sql

# Physical backup with Percona XtraBackup
xtrabackup --backup --target-dir=/backup/full

# Incremental backup
xtrabackup --backup --target-dir=/backup/inc1 --incremental-basedir=/backup/full
```

**Point-in-Time Recovery:**
```bash
# Enable binary logs
# my.cnf:
# log_bin = /var/log/mysql/mysql-bin.log
# expire_logs_days = 7

# Restore full backup
mysql -u root -p mydb < full-backup.sql

# Apply binary logs from backup time to target time
mysqlbinlog --start-datetime="2024-01-27 10:00:00" \
            --stop-datetime="2024-01-27 10:30:00" \
            mysql-bin.000001 | mysql -u root -p mydb
```

### 7. Replication

**Master-Slave Replication:**
```sql
-- Master configuration (my.cnf)
[mysqld]
server-id = 1
log_bin = mysql-bin
binlog_format = ROW

-- Create replication user on master
CREATE USER 'repl'@'%' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
FLUSH PRIVILEGES;

-- Get master status
SHOW MASTER STATUS;
-- Note: File and Position

-- Slave configuration (my.cnf)
[mysqld]
server-id = 2
relay_log = relay-bin
read_only = 1

-- Setup replication on slave
CHANGE MASTER TO
  MASTER_HOST='master_host',
  MASTER_USER='repl',
  MASTER_PASSWORD='password',
  MASTER_LOG_FILE='mysql-bin.000001',
  MASTER_LOG_POS=12345;

START SLAVE;

-- Check slave status
SHOW SLAVE STATUS\G
-- Look for:
-- Slave_IO_Running: Yes
-- Slave_SQL_Running: Yes
-- Seconds_Behind_Master: 0
```

## Best Practices

### 1. Schema Design
- Use appropriate data types (INT vs BIGINT, VARCHAR vs TEXT)
- Normalize to 3NF, denormalize for performance when needed
- Use UNSIGNED for non-negative values
- Use ENUM for fixed value sets
- Add created_at and updated_at to all tables

### 2. Indexing
- Index foreign keys
- Create composite indexes for common query patterns
- Don't over-index (affects INSERT/UPDATE performance)
- Monitor index usage with performance_schema

### 3. Query Writing
- Avoid SELECT *
- Use LIMIT for large result sets
- Use prepared statements to prevent SQL injection
- Avoid subqueries in SELECT list

### 4. Security
```sql
-- Create application user with limited privileges
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'strong_password';
GRANT SELECT, INSERT, UPDATE, DELETE ON mydb.* TO 'app_user'@'localhost';

-- Remove anonymous users
DELETE FROM mysql.user WHERE User='';

-- Disable remote root login
DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1');

FLUSH PRIVILEGES;
```

### 5. Monitoring
```sql
-- Enable performance_schema
SET GLOBAL performance_schema = ON;

-- Monitor query execution time
SELECT * FROM performance_schema.events_statements_summary_by_digest
ORDER BY SUM_TIMER_WAIT DESC LIMIT 10;

-- Check table locks
SELECT * FROM performance_schema.table_lock_waits_summary_by_table;
```

## Quick Reference

### Common Data Types
```sql
-- Integers
TINYINT    -- 1 byte (-128 to 127)
SMALLINT   -- 2 bytes (-32K to 32K)
MEDIUMINT  -- 3 bytes (-8M to 8M)
INT        -- 4 bytes (-2B to 2B)
BIGINT     -- 8 bytes (-9Q to 9Q)

-- Strings
CHAR(n)         -- Fixed length
VARCHAR(n)      -- Variable length (max 65,535)
TEXT            -- Max 65,535 characters
MEDIUMTEXT      -- Max 16MB
LONGTEXT        -- Max 4GB

-- Decimal
DECIMAL(10,2)   -- Exact decimal (price, money)
FLOAT           -- Approximate (scientific)
DOUBLE          -- Approximate (scientific)

-- Date/Time
DATE            -- YYYY-MM-DD
DATETIME        -- YYYY-MM-DD HH:MM:SS
TIMESTAMP       -- Auto-updated
TIME            -- HH:MM:SS
```

### Essential Commands
```sql
-- Database operations
CREATE DATABASE mydb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
DROP DATABASE mydb;
SHOW DATABASES;
USE mydb;

-- Table operations
SHOW TABLES;
DESCRIBE users;
SHOW CREATE TABLE users;
ALTER TABLE users ADD COLUMN phone VARCHAR(20);
DROP TABLE users;

-- Index operations
SHOW INDEX FROM users;
CREATE INDEX idx_name ON users(name);
DROP INDEX idx_name ON users;
ANALYZE TABLE users;

-- User management
CREATE USER 'username'@'host' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON db.* TO 'username'@'host';
REVOKE ALL PRIVILEGES ON db.* FROM 'username'@'host';
DROP USER 'username'@'host';
FLUSH PRIVILEGES;
```

### Performance Metrics
```sql
-- Connection stats
SHOW STATUS LIKE 'Connections';
SHOW STATUS LIKE 'Threads_connected';
SHOW STATUS LIKE 'Max_used_connections';

-- Query stats
SHOW STATUS LIKE 'Questions';
SHOW STATUS LIKE 'Slow_queries';

-- InnoDB stats
SHOW STATUS LIKE 'Innodb_buffer_pool%';
SHOW STATUS LIKE 'Innodb_rows%';
```

---

**Remember:** Proper indexing and query optimization are the foundation of MySQL performance. Always test with production-like data volumes.
