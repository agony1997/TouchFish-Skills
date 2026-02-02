---
name: composite_spring-stack
description: "複合技能：Spring Boot 全方位開發 + Spring Data JPA + 資料庫設計 + PostgreSQL 進階。REST API、例外處理、Actuator、Profile、Entity Mapping、Repository、JPQL、效能優化、DDD 聚合→表映射、DDL、Flyway、JSONB、全文搜尋、窗口函數、CTE、索引策略、分區表。關鍵字: spring boot, spring, java, springboot, rest api, jpa, spring data jpa, hibernate, orm, jpql, entity, repository, postgresql, postgres, psql, jsonb, full text search, cte, window function, 全文搜尋, 窗口函數, db design, 資料庫設計, database design, erd, schema, ddl, migration, flyway, 索引, index, 表結構, table design, 聚合邊界"
---

# Spring 全端技術棧 (Composite Spring Stack)

你是 Spring Boot + 資料庫全端技術專家,涵蓋 Spring Boot 核心開發、Spring Data JPA 資料存取、DDD 導向資料庫設計、PostgreSQL 進階功能。

---

## Part 1: Spring Boot 核心

### 1. 專案結構

```
my-spring-boot-app/
├── src/main/java/com.example.myapp/
│   ├── MyApplication.java
│   ├── config/                     # SecurityConfig, DatabaseConfig
│   ├── controller/                 # REST controllers
│   ├── service/impl/               # Business logic
│   ├── repository/                 # Data access
│   ├── model/entity/               # Domain models
│   ├── model/dto/                  # DTOs
│   ├── exception/                  # Exception handling
│   └── util/
├── src/main/resources/
│   ├── application.yml
│   ├── application-dev.yml
│   ├── application-prod.yml
│   └── db/migration/              # Flyway migrations
└── src/test/java/
```

### 2. RESTful API 開發

```java
@RestController
@RequestMapping("/api/v1/users")
@RequiredArgsConstructor
public class UserController {
    private final UserService userService;

    @GetMapping
    public ResponseEntity<Page<UserDTO>> getAll(Pageable pageable) {
        return ResponseEntity.ok(userService.findAll(pageable));
    }

    @GetMapping("/{id}")
    public ResponseEntity<UserDTO> getById(@PathVariable Long id) {
        return ResponseEntity.ok(userService.findById(id));
    }

    @PostMapping
    public ResponseEntity<UserDTO> create(@Valid @RequestBody UserCreateRequest request) {
        return ResponseEntity.status(HttpStatus.CREATED).body(userService.create(request));
    }

    @PutMapping("/{id}")
    public ResponseEntity<UserDTO> update(@PathVariable Long id, @Valid @RequestBody UserCreateRequest request) {
        return ResponseEntity.ok(userService.update(id, request));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        userService.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

### 3. Service Layer

```java
@Service
@RequiredArgsConstructor
@Slf4j
@Transactional(readOnly = true)
public class UserServiceImpl implements UserService {
    private final UserRepository userRepository;

    @Override
    public UserDTO findById(Long id) {
        return userRepository.findById(id)
            .map(this::toDTO)
            .orElseThrow(() -> new ResourceNotFoundException("User not found: " + id));
    }

    @Override
    @Transactional
    public UserDTO create(UserCreateRequest request) {
        if (userRepository.existsByEmail(request.getEmail()))
            throw new IllegalArgumentException("Email already exists");
        User user = User.builder().name(request.getName()).email(request.getEmail()).build();
        return toDTO(userRepository.save(user));
    }
}
```

### 4. 例外處理

```java
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        return ResponseEntity.status(404).body(ErrorResponse.of(404, "Not Found", ex.getMessage()));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors()
            .forEach(e -> errors.put(e.getField(), e.getDefaultMessage()));
        return ResponseEntity.badRequest().body(ErrorResponse.ofValidation(errors));
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGlobal(Exception ex) {
        log.error("Unexpected error: ", ex);
        return ResponseEntity.status(500).body(ErrorResponse.of(500, "Internal Server Error", "An unexpected error occurred"));
    }
}
```

### 5. Configuration Management

```yaml
# application.yml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/mydb
    username: ${DB_USERNAME:postgres}
    password: ${DB_PASSWORD:password}
    hikari:
      maximum-pool-size: 10
      minimum-idle: 5
  jpa:
    hibernate.ddl-auto: validate
    show-sql: false
    properties.hibernate.format_sql: true
  flyway:
    enabled: true
    baseline-on-migrate: true

management:
  endpoints.web.exposure.include: health,info,metrics,prometheus
  endpoint.health.show-details: when-authorized
```

**Profile 差異:**
- **dev**: ddl-auto=update, show-sql=true, log=DEBUG, actuator=全開
- **prod**: ddl-auto=validate, show-sql=false, log=WARN, actuator=限縮

### 6. Actuator

```
/actuator/health        - Health status
/actuator/info          - Application info
/actuator/metrics       - Metrics
/actuator/prometheus    - Prometheus format
```

```java
@Component
@RequiredArgsConstructor
public class DatabaseHealthIndicator implements HealthIndicator {
    private final DataSource dataSource;

    @Override
    public Health health() {
        try (Connection conn = dataSource.getConnection()) {
            return conn.isValid(1) ? Health.up().withDetail("db", "Connected").build()
                                   : Health.down().build();
        } catch (Exception e) {
            return Health.down().withDetail("error", e.getMessage()).build();
        }
    }
}
```

### 7. Best Practices

- **Constructor Injection**: `@RequiredArgsConstructor` + `private final` (不用 `@Autowired`)
- **DTOs**: 永不暴露 Entity 到 REST API
- **Transaction**: Service 類別 `@Transactional(readOnly = true)`,寫入方法覆寫 `@Transactional`
- **Validation**: `@Valid` + Bean Validation on DTOs
- **Logging**: `@Slf4j` + 結構化日誌

---

## Part 2: Spring Data JPA

### 1. Entity Mapping

```java
@Entity
@Table(name = "users")
@Getter @Setter @NoArgsConstructor @AllArgsConstructor @Builder
@EntityListeners(AuditingEntityListener.class)
public class User {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, length = 100)
    private String name;

    @Column(nullable = false, unique = true)
    private String email;

    @Enumerated(EnumType.STRING)
    private UserStatus status;

    @Embedded
    private Address address;

    @CreatedDate @Column(updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;
}
```

### 2. Entity Relationships

**One-to-Many / Many-to-One:**
```java
@Entity
public class Order {
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    private User user;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();

    public void addItem(OrderItem item) {
        items.add(item); item.setOrder(this);
    }
    public void removeItem(OrderItem item) {
        items.remove(item); item.setOrder(null);
    }
}
```

**Many-to-Many:**
```java
@ManyToMany
@JoinTable(name = "student_course",
    joinColumns = @JoinColumn(name = "student_id"),
    inverseJoinColumns = @JoinColumn(name = "course_id"))
private Set<Course> courses = new HashSet<>();
```

### 3. Repository

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
    List<User> findByNameContaining(String name);
    Page<User> findByStatus(UserStatus status, Pageable pageable);
    boolean existsByEmail(String email);

    @Modifying
    @Query("DELETE FROM User u WHERE u.status = :status")
    int deleteByStatus(@Param("status") UserStatus status);
}
```

**Query Method Keywords:**
```
findBy, existsBy, countBy, deleteBy
And, Or, Between, LessThan, GreaterThan, Like, In, NotIn
StartingWith, EndingWith, Containing, IgnoreCase
IsNull, IsNotNull, True, False
OrderBy...Asc/Desc, First, Top, Distinct
```

### 4. Custom Queries

**JPQL:**
```java
@Query("SELECT u FROM User u LEFT JOIN FETCH u.orders WHERE u.id = :id")
Optional<User> findByIdWithOrders(@Param("id") Long id);

@Query("SELECT new com.example.dto.UserDTO(u.id, u.name, u.email) FROM User u WHERE u.status = :status")
List<UserDTO> findDTOsByStatus(@Param("status") UserStatus status);

@Modifying
@Query("UPDATE User u SET u.status = :status WHERE u.id = :id")
int updateStatus(@Param("id") Long id, @Param("status") UserStatus status);
```

**Native SQL:**
```java
@Query(value = "SELECT u.*, COUNT(o.id) as order_count FROM users u LEFT JOIN orders o ON u.id = o.user_id GROUP BY u.id", nativeQuery = true)
List<Object[]> getUsersWithOrderCount();
```

### 5. Specifications (Dynamic Queries)

```java
public interface UserRepository extends JpaRepository<User, Long>, JpaSpecificationExecutor<User> {}

public class UserSpec {
    public static Specification<User> hasName(String name) {
        return (root, query, cb) -> name == null ? null : cb.like(root.get("name"), "%" + name + "%");
    }
    public static Specification<User> hasStatus(UserStatus status) {
        return (root, query, cb) -> status == null ? null : cb.equal(root.get("status"), status);
    }
}

// Usage
Specification<User> spec = Specification.where(UserSpec.hasName(name)).and(UserSpec.hasStatus(status));
List<User> users = userRepository.findAll(spec);
```

### 6. Performance Optimization

**N+1 Problem:**
```java
// Bad: N+1
List<User> users = repo.findAll();
users.forEach(u -> u.getOrders().size());  // extra query per user

// Good: Fetch Join
@Query("SELECT u FROM User u LEFT JOIN FETCH u.orders")
List<User> findAllWithOrders();

// Good: EntityGraph
@EntityGraph(attributePaths = {"orders", "profile"})
List<User> findAll();
```

**DTO Projections:**
```java
public interface UserSummary {
    String getName();
    String getEmail();
    Long getOrderCount();
}

@Query("SELECT u.name as name, u.email as email, COUNT(o) as orderCount FROM User u LEFT JOIN u.orders o GROUP BY u.id")
List<UserSummary> findUserSummaries();
```

### 7. JPA Best Practices

- **Lazy Loading by Default**: `@ManyToOne(fetch = FetchType.LAZY)`
- **Bidirectional Sync**: Helper methods `addItem()`/`removeItem()`
- **equals/hashCode**: Entity 用 ID,且處理 null ID: `return id != null && id.equals(other.id)`
- **Avoid toString() on relations**: `@ToString(exclude = {"orders"})`
- **@Modifying for updates**: Always with `@Transactional`

---

## Part 3: 資料庫設計 (DDD 導向)

### 1. 聚合→表結構映射

| DDD 概念 | PostgreSQL 映射 |
|----------|----------------|
| Aggregate Root | 主表 |
| Internal Entity | 子表 (FK + CASCADE) |
| Value Object (單一) | 嵌入欄位 (前綴區分: `address_city`) |
| Value Object (集合) | 獨立子表 |
| Enum | PostgreSQL ENUM 或 VARCHAR |
| 聚合間引用 | ID 引用 (不建 FK) |

**關鍵原則:**
- 聚合內: FK + ON DELETE CASCADE
- 聚合間: 只存 ID,不建外鍵 (各自管理生命週期)

### 2. DDL 設計範本

```sql
-- PostgreSQL Enum Type
CREATE TYPE order_status AS ENUM ('DRAFT', 'SUBMITTED', 'PAID', 'SHIPPED', 'COMPLETED', 'CANCELLED');

-- 聚合根表
CREATE TABLE orders (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_no    VARCHAR(20) NOT NULL,
    customer_id UUID NOT NULL,           -- 聚合間引用,只存 ID
    status      order_status NOT NULL DEFAULT 'DRAFT',
    -- 嵌入 Value Object
    shipping_address_street  VARCHAR(200),
    shipping_address_city    VARCHAR(100),
    shipping_address_zip     VARCHAR(10),
    -- Money VO
    total_amount  NUMERIC(18, 2) NOT NULL DEFAULT 0,
    currency      VARCHAR(3) NOT NULL DEFAULT 'TWD',
    -- 審計 + 樂觀鎖
    created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    version     INTEGER NOT NULL DEFAULT 0,
    CONSTRAINT uq_orders_order_no UNIQUE (order_no),
    CONSTRAINT ck_orders_total CHECK (total_amount >= 0)
);

-- 聚合內子表
CREATE TABLE order_items (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id    UUID NOT NULL,
    product_id  UUID NOT NULL,           -- 跨聚合引用
    quantity    INTEGER NOT NULL,
    unit_price  NUMERIC(18, 2) NOT NULL,
    subtotal    NUMERIC(18, 2) NOT NULL,
    CONSTRAINT fk_order_items_order FOREIGN KEY (order_id) REFERENCES orders(id) ON DELETE CASCADE,
    CONSTRAINT ck_quantity CHECK (quantity > 0)
);
```

### 3. 索引策略

```markdown
## 索引決策矩陣
1. 主鍵: 自動建立
2. 唯一約束: 自動建立索引
3. 外鍵欄位: 必須建立 (PostgreSQL 不自動建)
4. WHERE 常用欄位: 建立索引
5. JOIN 條件: 建立索引
6. 排序欄位: 建立或複合索引
7. 高選擇性: 優先索引
8. 低選擇性: 用 Partial Index 或不索引
```

```sql
-- 外鍵索引 (必要)
CREATE INDEX idx_order_items_order_id ON order_items(order_id);

-- 查詢條件索引
CREATE INDEX idx_orders_customer_id ON orders(customer_id);
CREATE INDEX idx_orders_status ON orders(status);

-- 複合索引
CREATE INDEX idx_orders_customer_status ON orders(customer_id, status);

-- 涵蓋索引 (避免回表)
CREATE INDEX idx_orders_list ON orders(customer_id, created_at DESC) INCLUDE (order_no, status, total_amount);

-- Partial Index (只索引活躍資料)
CREATE INDEX idx_orders_active ON orders(created_at DESC) WHERE status NOT IN ('COMPLETED', 'CANCELLED');

-- Expression Index
CREATE INDEX idx_orders_no_lower ON orders(LOWER(order_no));
```

### 4. Flyway 遷移

```
db/migration/
├── V1__create_order_context_tables.sql
├── V2__create_product_context_tables.sql
├── V3__add_orders_metadata_column.sql
└── R__refresh_order_summary_view.sql  (Repeatable)
```

**安全遷移原則:**
- 加欄位: `ADD COLUMN ... DEFAULT NULL` (不鎖表)
- 加索引: `CREATE INDEX CONCURRENTLY` (不鎖表, 不可在 transaction 中)
- 改型別: 加新欄位→遷移資料→刪舊欄位
- 刪欄位: 先確認無程式碼引用

---

## Part 4: PostgreSQL 進階

### 1. JSONB 操作

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    attributes JSONB
);

-- 查詢: -> 回傳 JSON, ->> 回傳 text
SELECT name, attributes->>'brand' AS brand FROM products;
SELECT * FROM products WHERE attributes->>'brand' = 'Apple';
SELECT * FROM products WHERE (attributes->>'price')::numeric > 1000;
SELECT * FROM products WHERE attributes @> '{"priority": "high"}';

-- 更新
UPDATE products SET attributes = attributes || '{"warranty": "2y"}'::jsonb WHERE id = 1;
UPDATE products SET attributes = attributes - 'warranty' WHERE id = 1;

-- JSONB 索引
CREATE INDEX idx_attrs ON products USING GIN (attributes);
CREATE INDEX idx_brand ON products ((attributes->>'brand'));
```

### 2. 全文搜尋

```sql
ALTER TABLE articles ADD COLUMN search_vector tsvector
GENERATED ALWAYS AS (to_tsvector('english', coalesce(title,'') || ' ' || coalesce(content,''))) STORED;

CREATE INDEX idx_search ON articles USING GIN (search_vector);

-- 搜尋
SELECT title FROM articles WHERE search_vector @@ to_tsquery('english', 'postgresql & database');

-- 排名
SELECT title, ts_rank(search_vector, query) AS rank
FROM articles, to_tsquery('english', 'postgresql & database') query
WHERE search_vector @@ query ORDER BY rank DESC;

-- Highlight
SELECT ts_headline('english', content, to_tsquery('postgresql'), 'MaxWords=20') FROM articles;
```

### 3. 窗口函數

```sql
-- ROW_NUMBER / RANK / DENSE_RANK
SELECT name, dept, salary,
    ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salary DESC) AS row_num,
    RANK() OVER (PARTITION BY dept ORDER BY salary DESC) AS rank
FROM employees;

-- LAG / LEAD
SELECT order_date, amount,
    LAG(amount) OVER (ORDER BY order_date) AS prev_amount
FROM orders;

-- Running Total
SELECT order_date, amount,
    SUM(amount) OVER (ORDER BY order_date) AS running_total
FROM orders;

-- Moving Average
SELECT date, value,
    AVG(value) OVER (ORDER BY date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS avg_7d
FROM metrics;

-- NTILE
SELECT name, purchases, NTILE(4) OVER (ORDER BY purchases DESC) AS quartile FROM customers;
```

### 4. CTE (Common Table Expressions)

```sql
-- 多重 CTE
WITH monthly_sales AS (
    SELECT DATE_TRUNC('month', order_date) AS month, SUM(total_amount) AS total
    FROM orders GROUP BY month
),
avg_sales AS (
    SELECT AVG(total) AS avg_monthly FROM monthly_sales
)
SELECT ms.month, ms.total FROM monthly_sales ms, avg_sales a WHERE ms.total > a.avg_monthly;

-- Recursive CTE (組織階層)
WITH RECURSIVE org_tree AS (
    SELECT id, name, manager_id, 0 AS level FROM employees WHERE manager_id IS NULL
    UNION ALL
    SELECT e.id, e.name, e.manager_id, ot.level + 1 FROM employees e JOIN org_tree ot ON e.manager_id = ot.id
)
SELECT * FROM org_tree ORDER BY level, name;
```

### 5. 進階索引類型

```sql
-- B-tree (default): 等值 + 範圍查詢
CREATE INDEX idx_name ON users(name);

-- GIN: JSONB, 陣列, 全文搜尋
CREATE INDEX idx_tags ON products USING GIN (tags);

-- GiST: 幾何資料, 全文搜尋
CREATE INDEX idx_location ON stores USING GIST (location);

-- BRIN: 大表 + 自然排序 (如時間序列)
CREATE INDEX idx_created ON logs USING BRIN (created_at);

-- Covering Index
CREATE INDEX idx_user_orders ON orders(user_id) INCLUDE (total_amount, order_date);
```

### 6. 分區表

```sql
-- Range Partitioning
CREATE TABLE orders (
    id BIGSERIAL, order_date DATE NOT NULL, customer_id INTEGER, total NUMERIC(10,2)
) PARTITION BY RANGE (order_date);

CREATE TABLE orders_2024_q1 PARTITION OF orders FOR VALUES FROM ('2024-01-01') TO ('2024-04-01');
CREATE TABLE orders_2024_q2 PARTITION OF orders FOR VALUES FROM ('2024-04-01') TO ('2024-07-01');

-- List Partitioning
CREATE TABLE users (id BIGSERIAL, username VARCHAR(50), country CHAR(2))
PARTITION BY LIST (country);
CREATE TABLE users_us PARTITION OF users FOR VALUES IN ('US');
CREATE TABLE users_other PARTITION OF users DEFAULT;
```

### 7. 效能優化

```sql
-- EXPLAIN ANALYZE
EXPLAIN ANALYZE SELECT u.name, COUNT(o.id) FROM users u LEFT JOIN orders o ON u.id = o.user_id GROUP BY u.id;

-- 看重點: Seq Scan (大表不好) → Index Scan (好) → Hash Join (大 JOIN 好)

-- 檢查未使用索引
SELECT schemaname, tablename, indexname, idx_scan FROM pg_stat_user_indexes WHERE idx_scan = 0;

-- VACUUM
VACUUM ANALYZE users;
```

### Quick Reference

**Spring Boot Annotations:**
```
@SpringBootApplication, @RestController, @Service, @Repository, @Configuration
@GetMapping, @PostMapping, @PutMapping, @DeleteMapping
@PathVariable, @RequestParam, @RequestBody, @Valid
@Transactional, @RequiredArgsConstructor
```

**JPA Annotations:**
```
@Entity, @Table, @Id, @GeneratedValue, @Column
@OneToOne, @OneToMany, @ManyToOne, @ManyToMany, @JoinColumn
@Embedded, @Embeddable, @Enumerated, @Transient
@CreatedDate, @LastModifiedDate, @EntityListeners
```

**PostgreSQL Data Types:**
```
SMALLINT, INTEGER, BIGINT, NUMERIC, REAL, DOUBLE PRECISION
VARCHAR(n), TEXT, CHAR(n), BOOLEAN, UUID
DATE, TIME, TIMESTAMP, TIMESTAMPTZ, INTERVAL
JSON, JSONB, INTEGER[], TEXT[], BYTEA, INET
```
