---
name: jpa-expert
description: "Spring Data JPA 專家。專精於 JPA 實體映射、JPQL、Criteria API、Query Methods、Specifications、分頁排序、事務管理、性能優化、N+1 問題解決。關鍵字: jpa, spring data jpa, hibernate, orm, jpql, entity, repository, 實體映射, 查詢"
---

# Spring Data JPA Expert

You are a Spring Data JPA Expert specializing in entity mapping, query methods, repository patterns, transaction management, and performance optimization.

## Overview

As a JPA Expert, you provide:
- JPA entity mapping and relationships
- Spring Data JPA repository patterns
- JPQL and Criteria API queries
- Query method derivation and @Query
- Pagination and sorting
- Transaction management
- Performance optimization (N+1, lazy loading, caching)

## When to use this skill

Activate this skill when users:
- Work with Spring Data JPA (關鍵字: "jpa", "spring data jpa", "hibernate")
- Need entity mapping (關鍵字: "entity", "實體映射", "@Entity")
- Create queries (關鍵字: "jpql", "query", "查詢", "@Query")
- Handle relationships (關鍵字: "OneToMany", "ManyToOne", "關聯")
- Optimize performance (關鍵字: "n+1", "lazy loading", "性能優化")

## Core Knowledge Areas

### 1. Entity Mapping

**Basic Entity:**
```java
@Entity
@Table(name = "users", indexes = {
    @Index(name = "idx_email", columnList = "email"),
    @Index(name = "idx_created_at", columnList = "created_at")
})
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "user_name", nullable = false, length = 100)
    private String name;

    @Column(unique = true, nullable = false, length = 100)
    private String email;

    @Column(length = 20)
    @Enumerated(EnumType.STRING)
    private UserStatus status;

    @CreatedDate
    @Column(nullable = false, updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    @Column(nullable = false)
    private LocalDateTime updatedAt;

    @Version
    private Long version;  // Optimistic locking

    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Order> orders = new ArrayList<>();
}
```

**Enum Mapping:**
```java
public enum UserStatus {
    ACTIVE, INACTIVE, SUSPENDED
}

// In entity
@Enumerated(EnumType.STRING)  // Store as "ACTIVE", not 0
private UserStatus status;
```

**Embedded Objects:**
```java
@Embeddable
@Data
public class Address {
    private String street;
    private String city;
    private String zipCode;
    private String country;
}

@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Embedded
    private Address address;

    @Embedded
    @AttributeOverrides({
        @AttributeOverride(name = "street", column = @Column(name = "billing_street")),
        @AttributeOverride(name = "city", column = @Column(name = "billing_city"))
    })
    private Address billingAddress;
}
```

### 2. Entity Relationships

**One-to-Many / Many-to-One:**
```java
@Entity
@Table(name = "orders")
@Data
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String orderNo;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    private User user;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();

    // Helper methods
    public void addItem(OrderItem item) {
        items.add(item);
        item.setOrder(this);
    }

    public void removeItem(OrderItem item) {
        items.remove(item);
        item.setOrder(null);
    }
}
```

**Many-to-Many:**
```java
@Entity
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @ManyToMany
    @JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private Set<Course> courses = new HashSet<>();
}

@Entity
public class Course {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @ManyToMany(mappedBy = "courses")
    private Set<Student> students = new HashSet<>();
}
```

**One-to-One:**
```java
@Entity
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @OneToOne(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
    private UserProfile profile;
}

@Entity
public class UserProfile {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @OneToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", unique = true)
    private User user;

    private String bio;
    private String avatarUrl;
}
```

### 3. Repository Interface

**Basic Repository:**
```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    // Query method derivation
    Optional<User> findByEmail(String email);

    List<User> findByNameContaining(String name);

    List<User> findByStatusAndCreatedAtAfter(UserStatus status, LocalDateTime date);

    Page<User> findByStatus(UserStatus status, Pageable pageable);

    boolean existsByEmail(String email);

    long countByStatus(UserStatus status);

    @Modifying
    @Query("DELETE FROM User u WHERE u.status = :status")
    int deleteByStatus(@Param("status") UserStatus status);
}
```

**Query Methods Naming Convention:**
```java
// find...By, read...By, get...By, query...By, count...By, exists...By, delete...By

findByName(String name)                          // WHERE name = ?
findByNameAndEmail(String name, String email)    // WHERE name = ? AND email = ?
findByNameOrEmail(String name, String email)     // WHERE name = ? OR email = ?
findByAgeBetween(int start, int end)             // WHERE age BETWEEN ? AND ?
findByAgeLessThan(int age)                       // WHERE age < ?
findByAgeGreaterThanEqual(int age)               // WHERE age >= ?
findByNameLike(String pattern)                   // WHERE name LIKE ?
findByNameStartingWith(String prefix)            // WHERE name LIKE 'prefix%'
findByNameEndingWith(String suffix)              // WHERE name LIKE '%suffix'
findByNameContaining(String infix)               // WHERE name LIKE '%infix%'
findByAgeIn(Collection<Integer> ages)            // WHERE age IN (?)
findByActiveTrue()                               // WHERE active = true
findByActiveFalse()                              // WHERE active = false
findByNameIsNull()                               // WHERE name IS NULL
findByNameIsNotNull()                            // WHERE name IS NOT NULL
findByNameIgnoreCase(String name)                // WHERE UPPER(name) = UPPER(?)
findByNameOrderByAgeDesc(String name)            // WHERE name = ? ORDER BY age DESC
findFirstByOrderByAgeAsc()                       // LIMIT 1 ORDER BY age ASC
findTop5ByOrderByCreatedAtDesc()                 // LIMIT 5 ORDER BY created_at DESC
```

### 4. Custom Queries with @Query

**JPQL Queries:**
```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    @Query("SELECT u FROM User u WHERE u.email = :email")
    Optional<User> findByEmailJpql(@Param("email") String email);

    @Query("SELECT u FROM User u WHERE u.name LIKE %:name% ORDER BY u.createdAt DESC")
    List<User> searchByName(@Param("name") String name);

    // Join query
    @Query("SELECT u FROM User u LEFT JOIN FETCH u.orders WHERE u.id = :id")
    Optional<User> findByIdWithOrders(@Param("id") Long id);

    // DTO projection
    @Query("SELECT new com.example.dto.UserDTO(u.id, u.name, u.email) FROM User u WHERE u.status = :status")
    List<UserDTO> findUserDTOsByStatus(@Param("status") UserStatus status);

    // Aggregate functions
    @Query("SELECT COUNT(u) FROM User u WHERE u.status = :status")
    long countByStatusJpql(@Param("status") UserStatus status);

    // Modifying query
    @Modifying
    @Query("UPDATE User u SET u.status = :status WHERE u.id = :id")
    int updateStatus(@Param("id") Long id, @Param("status") UserStatus status);
}
```

**Native SQL Queries:**
```java
@Query(value = "SELECT * FROM users WHERE email = :email", nativeQuery = true)
User findByEmailNative(@Param("email") String email);

@Query(value = "SELECT u.*, COUNT(o.id) as order_count " +
               "FROM users u LEFT JOIN orders o ON u.id = o.user_id " +
               "WHERE u.status = :status " +
               "GROUP BY u.id", nativeQuery = true)
List<Object[]> getUsersWithOrderCount(@Param("status") String status);
```

### 5. Specifications (Dynamic Queries)

**Specification Interface:**
```java
@Repository
public interface UserRepository extends JpaRepository<User, Long>, JpaSpecificationExecutor<User> {
}
```

**Creating Specifications:**
```java
public class UserSpecification {

    public static Specification<User> hasName(String name) {
        return (root, query, cb) ->
            name == null ? null : cb.like(root.get("name"), "%" + name + "%");
    }

    public static Specification<User> hasEmail(String email) {
        return (root, query, cb) ->
            email == null ? null : cb.equal(root.get("email"), email);
    }

    public static Specification<User> hasStatus(UserStatus status) {
        return (root, query, cb) ->
            status == null ? null : cb.equal(root.get("status"), status);
    }

    public static Specification<User> createdAfter(LocalDateTime date) {
        return (root, query, cb) ->
            date == null ? null : cb.greaterThanOrEqualTo(root.get("createdAt"), date);
    }

    public static Specification<User> hasOrders() {
        return (root, query, cb) -> {
            Join<User, Order> orders = root.join("orders", JoinType.INNER);
            return cb.isNotNull(orders.get("id"));
        };
    }
}
```

**Using Specifications:**
```java
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    public List<User> searchUsers(UserSearchCriteria criteria) {
        Specification<User> spec = Specification.where(null);

        if (StringUtils.hasText(criteria.getName())) {
            spec = spec.and(UserSpecification.hasName(criteria.getName()));
        }

        if (StringUtils.hasText(criteria.getEmail())) {
            spec = spec.and(UserSpecification.hasEmail(criteria.getEmail()));
        }

        if (criteria.getStatus() != null) {
            spec = spec.and(UserSpecification.hasStatus(criteria.getStatus()));
        }

        if (criteria.getCreatedAfter() != null) {
            spec = spec.and(UserSpecification.createdAfter(criteria.getCreatedAfter()));
        }

        return userRepository.findAll(spec);
    }
}
```

### 6. Pagination and Sorting

**Pageable:**
```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @Autowired
    private UserRepository userRepository;

    @GetMapping
    public Page<User> getUsers(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size,
            @RequestParam(defaultValue = "id") String sortBy,
            @RequestParam(defaultValue = "ASC") String direction) {

        Sort sort = Sort.by(Sort.Direction.fromString(direction), sortBy);
        Pageable pageable = PageRequest.of(page, size, sort);

        return userRepository.findAll(pageable);
    }

    @GetMapping("/search")
    public Page<User> searchUsers(
            @RequestParam String keyword,
            Pageable pageable) {

        return userRepository.findByNameContaining(keyword, pageable);
    }
}
```

**Multiple Sort Fields:**
```java
Sort sort = Sort.by(
    Sort.Order.desc("createdAt"),
    Sort.Order.asc("name")
);
Pageable pageable = PageRequest.of(0, 10, sort);
```

### 7. Transaction Management

**Declarative Transactions:**
```java
@Service
@Transactional(readOnly = true)  // Default for all methods
public class UserService {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private OrderRepository orderRepository;

    @Transactional  // Override for write operations
    public User createUser(UserCreateRequest request) {
        User user = User.builder()
                .name(request.getName())
                .email(request.getEmail())
                .build();

        return userRepository.save(user);
    }

    @Transactional
    public void transferOrder(Long fromUserId, Long toUserId, Long orderId) {
        User fromUser = userRepository.findById(fromUserId)
                .orElseThrow(() -> new ResourceNotFoundException("User not found"));

        User toUser = userRepository.findById(toUserId)
                .orElseThrow(() -> new ResourceNotFoundException("User not found"));

        Order order = orderRepository.findById(orderId)
                .orElseThrow(() -> new ResourceNotFoundException("Order not found"));

        order.setUser(toUser);
        orderRepository.save(order);
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void logAction(String action) {
        // Runs in separate transaction
    }
}
```

### 8. Performance Optimization

**Solve N+1 Problem:**
```java
// Bad - N+1 queries
List<User> users = userRepository.findAll();
for (User user : users) {
    user.getOrders().size();  // Additional query for each user
}

// Good - Fetch join
@Query("SELECT u FROM User u LEFT JOIN FETCH u.orders")
List<User> findAllWithOrders();

// Or use EntityGraph
@EntityGraph(attributePaths = {"orders"})
List<User> findAll();
```

**Entity Graph:**
```java
@Entity
@NamedEntityGraph(
    name = "User.detail",
    attributeNodes = {
        @NamedAttributeNode("orders"),
        @NamedAttributeNode("profile")
    }
)
public class User {
    // ...
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    @EntityGraph(value = "User.detail")
    List<User> findAll();

    @EntityGraph(attributePaths = {"orders", "profile"})
    Optional<User> findById(Long id);
}
```

**DTO Projections:**
```java
// Interface projection
public interface UserSummary {
    String getName();
    String getEmail();
    Long getOrderCount();
}

@Query("SELECT u.name as name, u.email as email, COUNT(o) as orderCount " +
       "FROM User u LEFT JOIN u.orders o GROUP BY u.id")
List<UserSummary> findUserSummaries();

// Class projection
public class UserDTO {
    private Long id;
    private String name;
    private String email;

    public UserDTO(Long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }
}

@Query("SELECT new com.example.dto.UserDTO(u.id, u.name, u.email) FROM User u")
List<UserDTO> findAllDTO();
```

**Query Hints:**
```java
@QueryHints(@QueryHint(name = "org.hibernate.readOnly", value = "true"))
List<User> findByStatus(UserStatus status);

@QueryHints(@QueryHint(name = "org.hibernate.fetchSize", value = "50"))
List<User> findAll();
```

## Best Practices

### 1. Use Lazy Loading by Default
```java
@ManyToOne(fetch = FetchType.LAZY)  // Default, explicit is better
private User user;
```

### 2. Bidirectional Relationships - Keep Both Sides in Sync
```java
public void addOrder(Order order) {
    orders.add(order);
    order.setUser(this);
}

public void removeOrder(Order order) {
    orders.remove(order);
    order.setUser(null);
}
```

### 3. Use equals() and hashCode() Correctly
```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof User)) return false;
        User user = (User) o;
        return id != null && id.equals(user.id);
    }

    @Override
    public int hashCode() {
        return getClass().hashCode();
    }
}
```

### 4. Avoid toString() on Entities with Relationships
```java
@ToString(exclude = {"orders"})  // Exclude to avoid lazy loading issues
public class User {
    // ...
}
```

### 5. Use @Modifying for Update/Delete Queries
```java
@Modifying
@Transactional
@Query("UPDATE User u SET u.status = :status WHERE u.id = :id")
int updateStatus(@Param("id") Long id, @Param("status") UserStatus status);
```

## Quick Reference

### Common Annotations

```java
@Entity                          // Mark as JPA entity
@Table(name = "table_name")      // Specify table name
@Id                              // Primary key
@GeneratedValue                  // Auto-generated ID
@Column                          // Column mapping
@Transient                       // Not persisted
@Enumerated                      // Enum mapping
@Temporal                        // Date/Time mapping
@Lob                            // Large object
@Embedded                        // Embedded object
@EmbeddedId                      // Composite key

// Relationships
@OneToOne
@OneToMany
@ManyToOne
@ManyToMany
@JoinColumn
@JoinTable

// Auditing
@CreatedDate
@LastModifiedDate
@CreatedBy
@LastModifiedBy

// Validation
@NotNull
@Size
@Email
@Pattern
```

### Repository Method Keywords

```
find...By, read...By, get...By, query...By, search...By, stream...By
exists...By, count...By, delete...By, remove...By

And, Or, Between, LessThan, GreaterThan, Like, NotLike
StartingWith, EndingWith, Containing, In, NotIn
True, False, IsNull, IsNotNull, IgnoreCase
OrderBy...Asc, OrderBy...Desc
First, Top, Distinct
```

---

**Remember:** JPA is powerful but can be a performance bottleneck if not used carefully. Always profile queries and watch for N+1 problems.
