# DDD Theory Fundamentals

> On-demand reference. Load when user is unfamiliar with DDD or asks about concepts.

## Strategic Design

### 1. Bounded Context（限界上下文）

限界上下文是 DDD 中最重要的戰略模式。每個 Bounded Context 都有自己的通用語言和領域模型。

**電商系統範例：**

同一個「Product」概念在不同上下文中有不同含義：

- **Sales Context**: Product 包含價格、折扣、促銷資訊
- **Inventory Context**: Product 包含庫存數量、倉儲位置、SKU
- **Shipping Context**: Product 包含重量、尺寸、運送限制

```java
// Sales Context
public class Product {
    private ProductId id;
    private String name;
    private Money price;
    private Discount discount;

    public Money calculateSellingPrice() {
        return discount.applyTo(price);
    }
}

// Inventory Context
public class Product {
    private ProductId id;
    private String sku;
    private int stockQuantity;
    private WarehouseLocation location;

    public boolean isAvailable(int requestedQuantity) {
        return stockQuantity >= requestedQuantity;
    }
}

// Shipping Context
public class Product {
    private ProductId id;
    private Weight weight;
    private Dimensions dimensions;
    private ShippingRestrictions restrictions;

    public boolean canShipTo(Address destination) {
        return !restrictions.isRestricted(destination);
    }
}
```

### 2. Context Mapping（上下文映射）

定義 Bounded Context 之間的關係和整合模式。

**Shared Kernel（共享核心）：** 兩個上下文共用一組程式碼。

```java
// Shared Kernel - 兩個上下文都使用
public class Money {
    private final BigDecimal amount;
    private final Currency currency;

    public Money(BigDecimal amount, Currency currency) {
        if (amount == null || currency == null) {
            throw new IllegalArgumentException("Amount and currency are required");
        }
        this.amount = amount;
        this.currency = currency;
    }

    public Money add(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException("Cannot add different currencies");
        }
        return new Money(this.amount.add(other.amount), this.currency);
    }

    // equals, hashCode based on amount and currency
}
```

**Anti-Corruption Layer（防腐層）：** 保護自己的領域模型不被外部系統污染。

```java
// Domain layer - 我們的介面
public interface PaymentGateway {
    PaymentResult charge(Money amount, PaymentMethod method);
}

// Infrastructure layer - 防腐層實作
public class StripePaymentAdapter implements PaymentGateway {
    private final StripeClient stripeClient;

    @Override
    public PaymentResult charge(Money amount, PaymentMethod method) {
        // 將我們的領域模型轉換為 Stripe 的 API 格式
        StripeChargeRequest request = new StripeChargeRequest();
        request.setAmountInCents(amount.toCents());
        request.setCurrency(amount.getCurrency().getCode());
        request.setSource(toStripeSource(method));

        // 呼叫外部系統
        StripeChargeResponse response = stripeClient.createCharge(request);

        // 將外部回應轉換回我們的領域模型
        return toPaymentResult(response);
    }

    private PaymentResult toPaymentResult(StripeChargeResponse response) {
        if ("succeeded".equals(response.getStatus())) {
            return PaymentResult.success(response.getId());
        }
        return PaymentResult.failed(response.getFailureMessage());
    }
}
```

## Tactical Design

### 1. Entity vs Value Object

**Entity（實體）：** 具有唯一識別碼，以 ID 區分。

```java
public class Order {
    private final OrderId id;  // 唯一識別碼
    private OrderStatus status;
    private List<OrderLine> lines;

    // Entity 以 ID 判斷相等
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Order)) return false;
        Order other = (Order) o;
        return id.equals(other.id);
    }

    @Override
    public int hashCode() {
        return id.hashCode();
    }
}
```

**Value Object（值對象）：** 沒有 ID，以屬性值判斷相等，不可變。

```java
public class Money {
    private final BigDecimal amount;
    private final Currency currency;

    public Money(BigDecimal amount, Currency currency) {
        this.amount = amount;
        this.currency = currency;
    }

    // Value Object 以值判斷相等
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Money)) return false;
        Money other = (Money) o;
        return amount.equals(other.amount) && currency.equals(other.currency);
    }

    // 不可變 - 返回新物件
    public Money add(Money other) {
        return new Money(this.amount.add(other.amount), this.currency);
    }
}

public class Address {
    private final String street;
    private final String city;
    private final String zipCode;
    private final String country;

    // 建構子驗證
    public Address(String street, String city, String zipCode, String country) {
        if (street == null || street.isBlank()) throw new IllegalArgumentException("Street is required");
        if (city == null || city.isBlank()) throw new IllegalArgumentException("City is required");
        this.street = street;
        this.city = city;
        this.zipCode = zipCode;
        this.country = country;
    }

    // 不可變 - 返回新物件
    public Address withCity(String newCity) {
        return new Address(this.street, newCity, this.zipCode, this.country);
    }

    // equals/hashCode based on all fields
}
```

### 2. Aggregate and Aggregate Root（聚合與聚合根）

聚合是一致性邊界，聚合根是唯一對外入口。

```java
public class Order {  // Aggregate Root
    private final OrderId id;
    private CustomerId customerId;
    private List<OrderLine> lines = new ArrayList<>();  // 內部 Entity
    private OrderStatus status;
    private Money totalAmount;

    // 只能透過聚合根操作內部物件
    public void addLine(ProductId productId, int quantity, Money unitPrice) {
        if (status != OrderStatus.DRAFT) {
            throw new IllegalStateException("Cannot modify a non-draft order");
        }
        OrderLine line = new OrderLine(productId, quantity, unitPrice);
        lines.add(line);
        recalculateTotal();
    }

    public void removeLine(ProductId productId) {
        if (status != OrderStatus.DRAFT) {
            throw new IllegalStateException("Cannot modify a non-draft order");
        }
        lines.removeIf(line -> line.getProductId().equals(productId));
        recalculateTotal();
    }

    public void submit() {
        if (lines.isEmpty()) {
            throw new IllegalStateException("Cannot submit an empty order");
        }
        this.status = OrderStatus.SUBMITTED;
    }

    private void recalculateTotal() {
        this.totalAmount = lines.stream()
            .map(OrderLine::getSubtotal)
            .reduce(Money.ZERO, Money::add);
    }
}

public class OrderLine {  // 聚合內部 Entity
    private final ProductId productId;
    private int quantity;
    private Money unitPrice;

    public Money getSubtotal() {
        return unitPrice.multiply(quantity);
    }
}
```

**Repository 只為聚合根定義：**

```java
// 正確 - 只有聚合根有 Repository
public interface OrderRepository {
    Order findById(OrderId id);
    void save(Order order);
}

// 錯誤 - OrderLine 不應有自己的 Repository
// public interface OrderLineRepository { ... }
```

### 3. Domain Service（領域服務）

當業務邏輯不屬於任何一個 Entity 時，使用 Domain Service。

```java
// 跨聚合的定價邏輯
public class OrderPricingService {
    public Money calculateTotal(Order order, DiscountPolicy discountPolicy) {
        Money subtotal = order.getSubtotal();
        Money discount = discountPolicy.calculateDiscount(subtotal, order.getCustomerId());
        return subtotal.subtract(discount);
    }
}

// 跨聚合的轉帳邏輯
public class TransferMoneyService {
    public void transfer(Account from, Account to, Money amount) {
        if (!from.hasSufficientBalance(amount)) {
            throw new InsufficientBalanceException(from.getId(), amount);
        }
        from.debit(amount);
        to.credit(amount);
    }
}
```

### 4. Domain Events（領域事件）

領域事件記錄領域中發生的重要事情。

```java
// 基礎事件類別
public abstract class DomainEvent {
    private final String eventId;
    private final LocalDateTime occurredOn;

    protected DomainEvent() {
        this.eventId = UUID.randomUUID().toString();
        this.occurredOn = LocalDateTime.now();
    }

    public String getEventId() { return eventId; }
    public LocalDateTime getOccurredOn() { return occurredOn; }
}

// 具體事件
public class OrderPlacedEvent extends DomainEvent {
    private final OrderId orderId;
    private final CustomerId customerId;
    private final Money totalAmount;

    public OrderPlacedEvent(OrderId orderId, CustomerId customerId, Money totalAmount) {
        super();
        this.orderId = orderId;
        this.customerId = customerId;
        this.totalAmount = totalAmount;
    }
}

public class OrderShippedEvent extends DomainEvent {
    private final OrderId orderId;
    private final TrackingNumber trackingNumber;

    public OrderShippedEvent(OrderId orderId, TrackingNumber trackingNumber) {
        super();
        this.orderId = orderId;
        this.trackingNumber = trackingNumber;
    }
}
```

**Entity 收集事件：**

```java
public abstract class AggregateRoot {
    private final List<DomainEvent> domainEvents = new ArrayList<>();

    protected void registerEvent(DomainEvent event) {
        domainEvents.add(event);
    }

    public List<DomainEvent> getDomainEvents() {
        return Collections.unmodifiableList(domainEvents);
    }

    public void clearEvents() {
        domainEvents.clear();
    }
}

public class Order extends AggregateRoot {
    public void place() {
        this.status = OrderStatus.PLACED;
        registerEvent(new OrderPlacedEvent(this.id, this.customerId, this.totalAmount));
    }
}
```

**事件發布與處理：**

```java
public interface EventPublisher {
    void publish(DomainEvent event);
}

public interface EventHandler<T extends DomainEvent> {
    void handle(T event);
}

// 範例：訂單成立後通知庫存
public class OrderPlacedEventHandler implements EventHandler<OrderPlacedEvent> {
    private final InventoryService inventoryService;

    @Override
    public void handle(OrderPlacedEvent event) {
        inventoryService.reserveStock(event.getOrderId());
    }
}
```

### 5. Repository Pattern（倉儲模式）

Repository 介面定義在領域層，實作在基礎設施層。

```java
// Domain Layer - 介面
public interface OrderRepository {
    Order findById(OrderId id);
    Optional<Order> findByIdOptional(OrderId id);
    void save(Order order);
    void delete(Order order);
    List<Order> findByCustomerId(CustomerId customerId);
}

// Infrastructure Layer - JPA 實作
@Repository
public class JpaOrderRepository implements OrderRepository {
    private final SpringDataOrderRepository springDataRepo;

    @Override
    public Order findById(OrderId id) {
        return springDataRepo.findById(id.getValue())
            .orElseThrow(() -> new OrderNotFoundException(id));
    }

    @Override
    public void save(Order order) {
        springDataRepo.save(order);
    }

    // ... 其他方法
}

// Spring Data JPA
public interface SpringDataOrderRepository extends JpaRepository<Order, UUID> {
    List<Order> findByCustomerId(UUID customerId);
}
```

### 6. CQRS Pattern（命令查詢職責分離）

將寫入（Command）與讀取（Query）分離。

**Command Side（寫入端）：**

```java
// Command
public class CreateOrderCommand {
    private final CustomerId customerId;
    private final List<OrderLineRequest> lines;

    // constructor, getters
}

// Command Handler (Application Service)
@Service
public class OrderCommandService {
    private final OrderRepository orderRepository;
    private final EventPublisher eventPublisher;

    @Transactional
    public OrderId handle(CreateOrderCommand command) {
        Order order = Order.create(command.getCustomerId(), command.getLines());
        orderRepository.save(order);

        // 發布領域事件
        order.getDomainEvents().forEach(eventPublisher::publish);
        order.clearEvents();

        return order.getId();
    }
}
```

**Query Side（查詢端）：**

```java
// Query Model（專為讀取優化）
public class OrderSummaryQueryModel {
    private String orderId;
    private String customerName;
    private String status;
    private BigDecimal totalAmount;
    private LocalDateTime createdAt;
    private int itemCount;

    // constructor, getters - 扁平化結構，適合顯示
}

// Query Service（可直接查 DB，不經過 Domain Model）
@Service
public class OrderQueryService {
    private final JdbcTemplate jdbcTemplate;

    public OrderSummaryQueryModel findOrderSummary(String orderId) {
        return jdbcTemplate.queryForObject(
            "SELECT o.id, c.name, o.status, o.total_amount, o.created_at, " +
            "       (SELECT COUNT(*) FROM order_lines ol WHERE ol.order_id = o.id) as item_count " +
            "FROM orders o JOIN customers c ON o.customer_id = c.id " +
            "WHERE o.id = ?",
            (rs, rowNum) -> new OrderSummaryQueryModel(
                rs.getString("id"),
                rs.getString("name"),
                rs.getString("status"),
                rs.getBigDecimal("total_amount"),
                rs.getTimestamp("created_at").toLocalDateTime(),
                rs.getInt("item_count")
            ),
            orderId
        );
    }

    public List<OrderSummaryQueryModel> findOrdersByCustomer(String customerId, int page, int size) {
        // 可使用專門的讀取模型表、View、或直接 JOIN 查詢
        // 不需經過 Domain Model，效能更好
    }
}
```

## Best Practices

1. **Ubiquitous Language（通用語言）：** 程式碼反映業務語言

```java
// Good - 使用業務語言
order.place();
order.cancel("Customer requested");
account.debit(amount);

// Bad - 使用技術語言
order.setStatus(Status.PLACED);
order.updateStatusToCancelled();
account.setBalance(account.getBalance() - amount);
```

2. **Keep Aggregates Small（保持聚合精簡）：** 跨聚合用 ID 引用

```java
// Good - 引用 ID
public class Order {
    private CustomerId customerId;  // 引用，不持有完整 Customer
}

// Bad - 直接引用另一個聚合
public class Order {
    private Customer customer;  // 會導致聚合過大
}
```

3. **Protect Invariants（保護不變條件）：**

```java
public class Order {
    public void addLine(ProductId productId, int quantity, Money unitPrice) {
        if (quantity <= 0) {
            throw new IllegalArgumentException("Quantity must be positive");
        }
        if (status != OrderStatus.DRAFT) {
            throw new IllegalStateException("Can only add lines to draft orders");
        }
        // ... add line
    }
}
```

4. **Model True Business Rules（建模真實業務規則）：**

```java
public class OrderApprovalService {
    public void approve(Order order, Approver approver) {
        if (order.getTotalAmount().isGreaterThan(Money.of(10000))) {
            if (!approver.hasRole(Role.SENIOR_MANAGER)) {
                throw new InsufficientApprovalAuthorityException();
            }
        }
        order.approve(approver.getId());
    }
}
```

## Quick Reference

**DDD Building Blocks:**

| Building Block | 特徵 | 範例 |
|---|---|---|
| Entity | 有 ID、可變、以 ID 相等 | Order, Customer, Product |
| Value Object | 無 ID、不可變、以值相等 | Money, Address, DateRange |
| Aggregate | 一致性邊界、聚合根為入口 | Order (root) + OrderLine |
| Domain Event | 過去式命名、不可變 | OrderPlacedEvent |
| Repository | 聚合根的持久化介面 | OrderRepository |
| Domain Service | 跨 Entity 的業務邏輯 | TransferMoneyService |
| Factory | 複雜物件的建立邏輯 | OrderFactory |

**Strategic vs Tactical:**

| 層面 | Strategic Design | Tactical Design |
|---|---|---|
| 關注點 | 系統邊界與上下文關係 | 單一上下文內的模型設計 |
| 產出 | Bounded Context, Context Map | Entity, VO, Aggregate, Event |
| 參與者 | 架構師、領域專家、團隊 | 開發者、領域專家 |
| 粒度 | 粗粒度（系統層級） | 細粒度（程式碼層級） |

**Remember:** DDD is about modeling the business domain accurately, not just applying patterns.
