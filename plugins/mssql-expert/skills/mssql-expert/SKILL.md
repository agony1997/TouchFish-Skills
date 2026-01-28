---
name: mssql-expert
description: "Microsoft SQL Server 資料庫專家,精通 T-SQL 與效能調校。關鍵字: mssql, sql server, t-sql, 資料庫"
---

# Microsoft SQL Server 專家技能

## 角色定位
你是 Microsoft SQL Server 資料庫專家,精通 T-SQL 開發、效能調校、索引優化、以及 Java/Quarkus 整合。專注於資料庫設計、查詢優化、交易管理、以及資料庫維護。

## 核心能力

### 1. T-SQL 查詢語言

#### 基礎查詢
```sql
-- SELECT 語句
SELECT
    column1,
    column2,
    CASE
        WHEN condition THEN result
        ELSE default_result
    END AS alias
FROM table_name
WHERE condition
ORDER BY column1 DESC
OFFSET 10 ROWS FETCH NEXT 20 ROWS ONLY;  -- 分頁

-- JOIN 操作
SELECT a.*, b.column
FROM table_a a
INNER JOIN table_b b ON a.id = b.a_id
LEFT JOIN table_c c ON a.id = c.a_id
WHERE a.status = 'active';

-- 聚合查詢
SELECT
    category,
    COUNT(*) as count,
    SUM(amount) as total,
    AVG(amount) as average,
    MAX(created_at) as latest
FROM orders
GROUP BY category
HAVING COUNT(*) > 10;
```

#### 子查詢與 CTE
```sql
-- Common Table Expression (CTE)
WITH MonthlyStats AS (
    SELECT
        YEAR(order_date) as year,
        MONTH(order_date) as month,
        SUM(amount) as total
    FROM orders
    GROUP BY YEAR(order_date), MONTH(order_date)
)
SELECT * FROM MonthlyStats WHERE total > 10000;

-- 遞迴 CTE (樹狀結構)
WITH EmployeeHierarchy AS (
    SELECT employee_id, name, manager_id, 0 AS level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    SELECT e.employee_id, e.name, e.manager_id, eh.level + 1
    FROM employees e
    INNER JOIN EmployeeHierarchy eh ON e.manager_id = eh.employee_id
)
SELECT * FROM EmployeeHierarchy;
```

#### 視窗函數
```sql
-- ROW_NUMBER, RANK, DENSE_RANK
SELECT
    name,
    department,
    salary,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) as row_num,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) as rank,
    DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) as dense_rank
FROM employees;

-- LAG, LEAD (前後列比較)
SELECT
    date,
    value,
    LAG(value, 1) OVER (ORDER BY date) as prev_value,
    LEAD(value, 1) OVER (ORDER BY date) as next_value,
    value - LAG(value, 1) OVER (ORDER BY date) as diff
FROM metrics;
```

#### 字串與日期函數
```sql
-- 字串處理
SELECT
    CONCAT(first_name, ' ', last_name) as full_name,
    UPPER(email) as email_upper,
    SUBSTRING(phone, 1, 3) as area_code,
    LEN(description) as desc_length,
    REPLACE(text, 'old', 'new') as replaced,
    TRIM(name) as trimmed
FROM users;

-- 日期處理
SELECT
    GETDATE() as now,
    DATEADD(day, 7, GETDATE()) as next_week,
    DATEDIFF(day, start_date, end_date) as days_diff,
    FORMAT(created_at, 'yyyy-MM-dd') as formatted_date,
    YEAR(date_column) as year,
    MONTH(date_column) as month,
    DAY(date_column) as day
FROM events;
```

### 2. Stored Procedures (預存程序)

```sql
-- 基礎 Stored Procedure
CREATE PROCEDURE sp_GetUserOrders
    @userId INT,
    @startDate DATE = NULL,
    @endDate DATE = NULL
AS
BEGIN
    SET NOCOUNT ON;

    SELECT
        o.order_id,
        o.order_date,
        o.total_amount,
        o.status
    FROM orders o
    WHERE o.user_id = @userId
        AND (@startDate IS NULL OR o.order_date >= @startDate)
        AND (@endDate IS NULL OR o.order_date <= @endDate)
    ORDER BY o.order_date DESC;
END;

-- 執行 Stored Procedure
EXEC sp_GetUserOrders @userId = 123, @startDate = '2024-01-01';
```

```sql
-- 帶交易與錯誤處理
CREATE PROCEDURE sp_TransferBalance
    @fromAccountId INT,
    @toAccountId INT,
    @amount DECIMAL(18,2)
AS
BEGIN
    SET NOCOUNT ON;
    BEGIN TRY
        BEGIN TRANSACTION;

        -- 檢查餘額
        DECLARE @balance DECIMAL(18,2);
        SELECT @balance = balance FROM accounts WHERE account_id = @fromAccountId;

        IF @balance < @amount
        BEGIN
            THROW 50001, 'Insufficient balance', 1;
        END

        -- 扣款
        UPDATE accounts
        SET balance = balance - @amount
        WHERE account_id = @fromAccountId;

        -- 入帳
        UPDATE accounts
        SET balance = balance + @amount
        WHERE account_id = @toAccountId;

        COMMIT TRANSACTION;
    END TRY
    BEGIN CATCH
        IF @@TRANCOUNT > 0
            ROLLBACK TRANSACTION;

        THROW;
    END CATCH
END;
```

### 3. 索引優化

#### 索引類型
```sql
-- Clustered Index (叢集索引,每個表只能有一個)
CREATE CLUSTERED INDEX IX_Orders_OrderDate
ON orders(order_date);

-- Non-Clustered Index (非叢集索引)
CREATE NONCLUSTERED INDEX IX_Orders_UserId
ON orders(user_id);

-- Composite Index (複合索引)
CREATE NONCLUSTERED INDEX IX_Orders_UserDate
ON orders(user_id, order_date);

-- Covering Index (涵蓋索引,包含查詢所需所有欄位)
CREATE NONCLUSTERED INDEX IX_Orders_UserDate_Include
ON orders(user_id, order_date)
INCLUDE (total_amount, status);

-- Filtered Index (過濾索引,只索引部分資料)
CREATE NONCLUSTERED INDEX IX_Orders_Active
ON orders(order_date)
WHERE status = 'active';

-- Unique Index (唯一索引)
CREATE UNIQUE NONCLUSTERED INDEX IX_Users_Email
ON users(email);
```

#### 索引維護
```sql
-- 重建索引
ALTER INDEX IX_Orders_UserId ON orders REBUILD;

-- 重組索引
ALTER INDEX IX_Orders_UserId ON orders REORGANIZE;

-- 更新統計資訊
UPDATE STATISTICS orders;

-- 查看索引碎片
SELECT
    object_name(ips.object_id) AS TableName,
    i.name AS IndexName,
    ips.avg_fragmentation_in_percent,
    ips.page_count
FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, 'DETAILED') ips
JOIN sys.indexes i ON ips.object_id = i.object_id AND ips.index_id = i.index_id
WHERE ips.avg_fragmentation_in_percent > 10
ORDER BY ips.avg_fragmentation_in_percent DESC;
```

### 4. 執行計劃分析

#### 啟用執行計劃
```sql
-- 顯示預估執行計劃
SET SHOWPLAN_XML ON;
GO

-- 顯示實際執行計劃
SET STATISTICS XML ON;
GO

-- 查看 I/O 統計
SET STATISTICS IO ON;
SET STATISTICS TIME ON;
GO

SELECT * FROM orders WHERE user_id = 123;
GO
```

#### 常見效能問題識別
- **Table Scan**: 全表掃描,應考慮加索引
- **Index Scan**: 索引掃描,可能需要更精確的索引
- **Key Lookup**: 需要回表查詢,考慮使用 Covering Index
- **Sort 操作**: 排序成本高,考慮索引支援排序
- **Hash Join**: 大量資料連接,檢查統計資訊是否最新

### 5. 交易管理

```sql
-- 顯式交易
BEGIN TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

IF @@ERROR <> 0
    ROLLBACK TRANSACTION;
ELSE
    COMMIT TRANSACTION;
```

```sql
-- 交易隔離等級
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;  -- 最低,可能讀到髒資料
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;    -- 預設
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;   -- 防止不可重複讀
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;      -- 最高,完全隔離
SET TRANSACTION ISOLATION LEVEL SNAPSHOT;          -- 快照隔離

-- 使用 SNAPSHOT 隔離
ALTER DATABASE mydb SET ALLOW_SNAPSHOT_ISOLATION ON;
SET TRANSACTION ISOLATION LEVEL SNAPSHOT;
BEGIN TRANSACTION;
-- 查詢操作
COMMIT;
```

### 6. 鎖定與併發控制

```sql
-- 鎖定提示
SELECT * FROM orders WITH (NOLOCK);           -- 不加鎖,可能讀到髒資料
SELECT * FROM orders WITH (ROWLOCK);          -- 行級鎖
SELECT * FROM orders WITH (UPDLOCK);          -- 更新鎖
SELECT * FROM orders WITH (XLOCK);            -- 排他鎖

-- 查看鎖定狀況
SELECT
    resource_type,
    resource_database_id,
    resource_associated_entity_id,
    request_mode,
    request_status
FROM sys.dm_tran_locks
WHERE resource_type = 'OBJECT';
```

### 7. JDBC 與 Quarkus 整合

#### Quarkus 配置
```properties
# application.properties
quarkus.datasource.db-kind=mssql
quarkus.datasource.username=sa
quarkus.datasource.password=YourPassword123
quarkus.datasource.jdbc.url=jdbc:sqlserver://localhost:1433;databaseName=mydb;trustServerCertificate=true;encrypt=true
quarkus.datasource.jdbc.max-size=20
quarkus.datasource.jdbc.min-size=5

# Hibernate 配置
quarkus.hibernate-orm.database.generation=validate
quarkus.hibernate-orm.log.sql=true
quarkus.hibernate-orm.dialect=org.hibernate.dialect.SQLServerDialect
```

#### Panache Entity 範例
```java
@Entity
@Table(name = "orders", indexes = {
    @Index(name = "ix_orders_user_date", columnList = "user_id, order_date")
})
public class Order extends PanacheEntity {

    @Column(name = "user_id", nullable = false)
    public Long userId;

    @Column(name = "order_date", nullable = false)
    public LocalDateTime orderDate;

    @Column(name = "total_amount", precision = 18, scale = 2)
    public BigDecimal totalAmount;

    @Column(length = 50)
    public String status;

    // 查詢方法
    public static List<Order> findByUserId(Long userId) {
        return list("userId", userId);
    }

    public static List<Order> findByUserAndDateRange(Long userId, LocalDate start, LocalDate end) {
        return list("userId = ?1 and orderDate >= ?2 and orderDate < ?3",
                    userId, start.atStartOfDay(), end.plusDays(1).atStartOfDay());
    }

    // 使用 Named Query
    @NamedQuery(name = "Order.findActiveByUser",
                query = "from Order where userId = ?1 and status = 'active' order by orderDate desc")
    public static List<Order> findActiveByUser(Long userId) {
        return find("#Order.findActiveByUser", userId).list();
    }
}
```

#### Native Query (原生 SQL)
```java
@ApplicationScoped
public class OrderRepository {

    @Inject
    EntityManager em;

    public List<OrderSummary> getOrderSummaryByUser(Long userId) {
        String sql = """
            SELECT
                YEAR(order_date) as year,
                MONTH(order_date) as month,
                COUNT(*) as order_count,
                SUM(total_amount) as total
            FROM orders
            WHERE user_id = :userId
            GROUP BY YEAR(order_date), MONTH(order_date)
            ORDER BY year DESC, month DESC
            """;

        return em.createNativeQuery(sql)
            .setParameter("userId", userId)
            .unwrap(NativeQuery.class)
            .setResultSetMapping("OrderSummaryMapping")
            .getResultList();
    }

    @Transactional
    public int bulkUpdateStatus(String oldStatus, String newStatus) {
        return em.createQuery("UPDATE Order o SET o.status = :newStatus WHERE o.status = :oldStatus")
            .setParameter("oldStatus", oldStatus)
            .setParameter("newStatus", newStatus)
            .executeUpdate();
    }
}
```

### 8. 資料庫維護

#### 備份與還原
```sql
-- 完整備份
BACKUP DATABASE mydb TO DISK = 'C:\Backup\mydb.bak';

-- 差異備份
BACKUP DATABASE mydb TO DISK = 'C:\Backup\mydb_diff.bak' WITH DIFFERENTIAL;

-- 交易記錄備份
BACKUP LOG mydb TO DISK = 'C:\Backup\mydb_log.trn';

-- 還原資料庫
RESTORE DATABASE mydb FROM DISK = 'C:\Backup\mydb.bak' WITH REPLACE;
```

#### 效能監控
```sql
-- 查看耗時查詢
SELECT TOP 10
    qs.execution_count,
    qs.total_elapsed_time / 1000000 AS total_elapsed_time_sec,
    qs.total_elapsed_time / qs.execution_count / 1000000 AS avg_elapsed_time_sec,
    SUBSTRING(qt.text, (qs.statement_start_offset/2)+1,
        ((CASE qs.statement_end_offset
            WHEN -1 THEN DATALENGTH(qt.text)
            ELSE qs.statement_end_offset
        END - qs.statement_start_offset)/2) + 1) AS statement_text
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) qt
ORDER BY qs.total_elapsed_time DESC;

-- 查看等待統計
SELECT
    wait_type,
    wait_time_ms / 1000.0 AS wait_time_sec,
    waiting_tasks_count
FROM sys.dm_os_wait_stats
WHERE wait_time_ms > 0
ORDER BY wait_time_ms DESC;
```

## 效能優化最佳實踐

### 1. 避免 N+1 查詢問題
```java
// ❌ 不好: N+1 查詢
List<Order> orders = Order.listAll();
for (Order order : orders) {
    User user = User.findById(order.userId);  // 每次都查詢一次
}

// ✅ 好: 使用 JOIN 一次查詢
@Entity
public class Order extends PanacheEntity {
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id")
    public User user;
}

List<Order> orders = Order.find("from Order o left join fetch o.user").list();
```

### 2. 使用分頁
```java
// ✅ 使用 Panache 分頁
public List<Order> listOrders(int pageIndex, int pageSize) {
    return Order.findAll()
        .page(Page.of(pageIndex, pageSize))
        .list();
}
```

### 3. 批次操作
```java
@Transactional
public void batchInsert(List<Order> orders) {
    for (int i = 0; i < orders.size(); i++) {
        Order.persist(orders.get(i));
        if (i % 50 == 0) {
            Order.flush();
            Order.getEntityManager().clear();
        }
    }
}
```

### 4. 查詢優化
```sql
-- ❌ 不好: 函數運算導致無法使用索引
SELECT * FROM orders WHERE YEAR(order_date) = 2024;

-- ✅ 好: 直接比較,可使用索引
SELECT * FROM orders
WHERE order_date >= '2024-01-01' AND order_date < '2025-01-01';

-- ❌ 不好: OR 條件可能無法使用索引
SELECT * FROM orders WHERE user_id = 1 OR status = 'pending';

-- ✅ 好: 使用 UNION
SELECT * FROM orders WHERE user_id = 1
UNION
SELECT * FROM orders WHERE status = 'pending';
```

## 程式碼產出原則

1. **索引設計**: 為常用查詢欄位建立索引,特別是 WHERE, JOIN, ORDER BY 的欄位
2. **避免 SELECT ***: 只選取需要的欄位
3. **使用參數化查詢**: 防止 SQL Injection
4. **交易邊界**: 明確定義交易範圍,避免長交易
5. **錯誤處理**: 適當處理資料庫異常
6. **連線池管理**: 正確配置連線池大小
7. **批次處理**: 大量資料操作使用批次
8. **監控與日誌**: 記錄慢查詢,定期分析執行計劃

## 參考資源
- T-SQL 官方文檔: https://learn.microsoft.com/sql/t-sql/
- 執行計劃分析: https://learn.microsoft.com/sql/relational-databases/performance/execution-plans
- Quarkus MSSQL: https://quarkus.io/guides/datasource
