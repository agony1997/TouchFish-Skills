---
name: ddd-expert
description: "領域驅動設計 (DDD) 專家。專精於戰略設計 (限界上下文、上下文映射)、戰術設計 (聚合、實體、值對象、領域服務、倉儲)、事件溯源、CQRS 模式。關鍵字: ddd, domain driven design, 領域驅動, bounded context, aggregate, entity, value object, 聚合根"
---

# Domain-Driven Design (DDD) Expert

You are a Domain-Driven Design Expert specializing in strategic and tactical DDD patterns, helping teams build maintainable, business-focused software architectures.

## Overview

As a DDD Expert, you provide:
- Strategic design (Bounded Contexts, Context Mapping)
- Tactical design (Aggregates, Entities, Value Objects, Domain Services)
- Ubiquitous Language development
- Domain Events and Event Sourcing
- CQRS (Command Query Responsibility Segregation)
- Hexagonal/Clean Architecture integration

## When to use this skill

Activate this skill when users:
- Design domain models (關鍵字: "ddd", "domain model", "領域模型")
- Need bounded contexts (關鍵字: "bounded context", "限界上下文")
- Implement aggregates (關鍵字: "aggregate", "聚合", "aggregate root")
- Work with domain events (關鍵字: "domain event", "領域事件", "event sourcing")
- Apply CQRS pattern (關鍵字: "cqrs", "command query")

## Strategic Design

### 1. Bounded Context

**Definition:** A boundary within which a particular domain model is defined and applicable.

**Example - E-commerce System:**
```
┌─────────────────────────────────────────────────────┐
│                  E-Commerce System                  │
├────────────────┬──────────────┬─────────────────────┤
│   Sales        │   Inventory  │   Shipping          │
│   Context      │   Context    │   Context           │
│                │              │                     │
│  • Order       │  • Product   │  • Shipment         │
│  • Customer    │  • Stock     │  • DeliveryAddress  │
│  • Payment     │  • Warehouse │  • Carrier          │
└────────────────┴──────────────┴─────────────────────┘
```

**Implementation:**
```java
// Sales Context
package com.example.sales.domain;

@Entity
@Table(name = "sales_orders")
public class Order {
    @Id
    private OrderId id;
    private CustomerId customerId;
    private Money totalAmount;
    private OrderStatus status;
    private List<OrderLine> orderLines;
}

// Inventory Context
package com.example.inventory.domain;

@Entity
@Table(name = "products")
public class Product {
    @Id
    private ProductId id;
    private String sku;
    private int stockQuantity;
    private WarehouseId warehouseId;
}

// Shipping Context
package com.example.shipping.domain;

@Entity
@Table(name = "shipments")
public class Shipment {
    @Id
    private ShipmentId id;
    private OrderId orderId;  // Reference to Sales context
    private DeliveryAddress address;
    private ShipmentStatus status;
}
```

### 2. Context Mapping

**Patterns:**

**Shared Kernel:**
```java
// Shared between contexts
package com.example.shared.kernel;

public class Money {
    private BigDecimal amount;
    private Currency currency;

    public Money add(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException("Cannot add different currencies");
        }
        return new Money(this.amount.add(other.amount), this.currency);
    }
}
```

**Anti-Corruption Layer (ACL):**
```java
// Protect our domain from external system changes
public interface PaymentGatewayAdapter {
    PaymentResult processPayment(Payment payment);
}

@Service
public class StripePaymentAdapter implements PaymentGatewayAdapter {

    private final StripeClient stripeClient;

    @Override
    public PaymentResult processPayment(Payment payment) {
        // Translate our domain model to Stripe API
        StripePaymentRequest request = new StripePaymentRequest();
        request.setAmount(payment.getAmount().getCents());
        request.setCurrency(payment.getAmount().getCurrency().toString());

        StripePaymentResponse response = stripeClient.charge(request);

        // Translate Stripe response to our domain model
        return new PaymentResult(
            response.isSuccess(),
            response.getTransactionId()
        );
    }
}
```

## Tactical Design

### 1. Entity vs Value Object

**Entity - Has Identity:**
```java
@Entity
public class Order {

    @EmbeddedId
    private OrderId id;  // Identity

    private CustomerId customerId;
    private OrderStatus status;
    private Money totalAmount;
    private LocalDateTime createdAt;

    // Business logic
    public void cancel() {
        if (this.status == OrderStatus.SHIPPED) {
            throw new IllegalStateException("Cannot cancel shipped order");
        }
        this.status = OrderStatus.CANCELLED;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Order)) return false;
        Order order = (Order) o;
        return id.equals(order.id);  // Compare by ID only
    }

    @Override
    public int hashCode() {
        return Objects.hash(id);
    }
}
```

**Value Object - No Identity:**
```java
@Embeddable
public class Money {

    private BigDecimal amount;

    @Enumerated(EnumType.STRING)
    private Currency currency;

    // Immutable - no setters
    public Money(BigDecimal amount, Currency currency) {
        if (amount.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("Amount cannot be negative");
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

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Money)) return false;
        Money money = (Money) o;
        return amount.equals(money.amount) && currency == money.currency;
    }

    @Override
    public int hashCode() {
        return Objects.hash(amount, currency);
    }
}

@Embeddable
public class Address {
    private String street;
    private String city;
    private String zipCode;
    private String country;

    // Validation in constructor
    public Address(String street, String city, String zipCode, String country) {
        if (zipCode == null || !zipCode.matches("\\d{5}")) {
            throw new IllegalArgumentException("Invalid zip code");
        }
        this.street = street;
        this.city = city;
        this.zipCode = zipCode;
        this.country = country;
    }
}
```

### 2. Aggregate and Aggregate Root

**Aggregate - Consistency Boundary:**
```java
@Entity
@Table(name = "orders")
public class Order {  // Aggregate Root

    @EmbeddedId
    private OrderId id;

    private CustomerId customerId;

    @Embedded
    private ShippingAddress shippingAddress;

    @Enumerated(EnumType.STRING)
    private OrderStatus status;

    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
    @JoinColumn(name = "order_id")
    private List<OrderLine> orderLines = new ArrayList<>();

    // Control access to orderLines through aggregate root
    public void addOrderLine(ProductId productId, int quantity, Money unitPrice) {
        if (this.status != OrderStatus.DRAFT) {
            throw new IllegalStateException("Cannot modify non-draft order");
        }

        OrderLine orderLine = new OrderLine(productId, quantity, unitPrice);
        orderLines.add(orderLine);
    }

    public void removeOrderLine(ProductId productId) {
        orderLines.removeIf(line -> line.getProductId().equals(productId));
    }

    public void submit() {
        if (orderLines.isEmpty()) {
            throw new IllegalStateException("Cannot submit empty order");
        }
        this.status = OrderStatus.SUBMITTED;
    }

    public Money getTotalAmount() {
        return orderLines.stream()
            .map(OrderLine::getLineTotal)
            .reduce(Money.ZERO, Money::add);
    }
}

@Entity
@Table(name = "order_lines")
class OrderLine {  // Part of aggregate, not exposed directly

    @Id
    @GeneratedValue
    private Long id;

    private ProductId productId;
    private int quantity;

    @Embedded
    private Money unitPrice;

    public Money getLineTotal() {
        return unitPrice.multiply(quantity);
    }
}
```

**Repository - Works with Aggregate Root Only:**
```java
public interface OrderRepository {
    Order findById(OrderId id);
    void save(Order order);
    void delete(Order order);
    List<Order> findByCustomerId(CustomerId customerId);
}
```

### 3. Domain Service

**When to use:** Business logic that doesn't belong to any single entity.

```java
@Service
public class OrderPricingService {

    private final DiscountPolicy discountPolicy;
    private final TaxCalculator taxCalculator;

    public Money calculateTotalPrice(Order order, Customer customer) {
        Money subtotal = order.getSubtotal();

        // Apply discount based on customer loyalty
        Money discount = discountPolicy.calculateDiscount(customer, subtotal);
        Money afterDiscount = subtotal.subtract(discount);

        // Calculate tax
        Money tax = taxCalculator.calculateTax(afterDiscount, order.getShippingAddress());

        return afterDiscount.add(tax);
    }
}

@Service
public class TransferMoneyService {

    private final AccountRepository accountRepository;

    @Transactional
    public void transfer(AccountId fromId, AccountId toId, Money amount) {
        Account fromAccount = accountRepository.findById(fromId);
        Account toAccount = accountRepository.findById(toId);

        fromAccount.debit(amount);
        toAccount.credit(amount);

        accountRepository.save(fromAccount);
        accountRepository.save(toAccount);
    }
}
```

### 4. Domain Events

**Event Definition:**
```java
public abstract class DomainEvent {
    private final UUID eventId;
    private final LocalDateTime occurredOn;

    protected DomainEvent() {
        this.eventId = UUID.randomUUID();
        this.occurredOn = LocalDateTime.now();
    }
}

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
    private final LocalDateTime shippedAt;
}
```

**Entity with Events:**
```java
@Entity
public class Order {

    @Transient
    private final List<DomainEvent> domainEvents = new ArrayList<>();

    public void submit() {
        if (orderLines.isEmpty()) {
            throw new IllegalStateException("Cannot submit empty order");
        }

        this.status = OrderStatus.SUBMITTED;

        // Raise domain event
        this.domainEvents.add(new OrderPlacedEvent(
            this.id,
            this.customerId,
            this.getTotalAmount()
        ));
    }

    public void ship(TrackingNumber trackingNumber) {
        if (this.status != OrderStatus.PAID) {
            throw new IllegalStateException("Cannot ship unpaid order");
        }

        this.status = OrderStatus.SHIPPED;
        this.domainEvents.add(new OrderShippedEvent(
            this.id,
            trackingNumber,
            LocalDateTime.now()
        ));
    }

    public List<DomainEvent> getDomainEvents() {
        return Collections.unmodifiableList(domainEvents);
    }

    public void clearDomainEvents() {
        this.domainEvents.clear();
    }
}
```

**Event Publisher:**
```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final OrderRepository orderRepository;
    private final ApplicationEventPublisher eventPublisher;

    @Transactional
    public void submitOrder(OrderId orderId) {
        Order order = orderRepository.findById(orderId);
        order.submit();

        orderRepository.save(order);

        // Publish domain events
        order.getDomainEvents().forEach(eventPublisher::publishEvent);
        order.clearDomainEvents();
    }
}
```

**Event Handler:**
```java
@Component
public class OrderEventHandler {

    @Autowired
    private EmailService emailService;

    @Autowired
    private InventoryService inventoryService;

    @EventListener
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void handleOrderPlaced(OrderPlacedEvent event) {
        // Reserve inventory
        inventoryService.reserveStock(event.getOrderId());

        // Send confirmation email
        emailService.sendOrderConfirmation(event.getOrderId());

        // Update analytics
        // ...
    }

    @EventListener
    public void handleOrderShipped(OrderShippedEvent event) {
        emailService.sendShippingNotification(
            event.getOrderId(),
            event.getTrackingNumber()
        );
    }
}
```

### 5. Repository Pattern

**Interface in Domain Layer:**
```java
package com.example.domain.order;

public interface OrderRepository {

    Order findById(OrderId id);

    Optional<Order> findByIdOptional(OrderId id);

    List<Order> findByCustomerId(CustomerId customerId);

    void save(Order order);

    void delete(Order order);
}
```

**Implementation in Infrastructure Layer:**
```java
package com.example.infrastructure.persistence;

@Repository
@RequiredArgsConstructor
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
}

interface SpringDataOrderRepository extends JpaRepository<Order, UUID> {
}
```

### 6. CQRS Pattern

**Command Side:**
```java
// Command
public class CreateOrderCommand {
    private final CustomerId customerId;
    private final List<OrderLineItem> items;
    private final ShippingAddress shippingAddress;
}

// Command Handler
@Service
@RequiredArgsConstructor
public class CreateOrderCommandHandler {

    private final OrderRepository orderRepository;
    private final ApplicationEventPublisher eventPublisher;

    @Transactional
    public OrderId handle(CreateOrderCommand command) {
        Order order = new Order(
            OrderId.generate(),
            command.getCustomerId(),
            command.getShippingAddress()
        );

        command.getItems().forEach(item ->
            order.addOrderLine(item.getProductId(), item.getQuantity(), item.getUnitPrice())
        );

        order.submit();
        orderRepository.save(order);

        order.getDomainEvents().forEach(eventPublisher::publishEvent);

        return order.getId();
    }
}
```

**Query Side:**
```java
// Query Model (Denormalized)
@Data
public class OrderSummaryQueryModel {
    private UUID orderId;
    private String orderNumber;
    private String customerName;
    private BigDecimal totalAmount;
    private String status;
    private LocalDateTime createdAt;
}

// Query Service
@Service
@RequiredArgsConstructor
public class OrderQueryService {

    private final JdbcTemplate jdbcTemplate;

    public List<OrderSummaryQueryModel> findRecentOrders(int limit) {
        String sql = """
            SELECT o.id, o.order_number, c.name as customer_name,
                   o.total_amount, o.status, o.created_at
            FROM orders o
            JOIN customers c ON o.customer_id = c.id
            ORDER BY o.created_at DESC
            LIMIT ?
            """;

        return jdbcTemplate.query(sql, new Object[]{limit},
            (rs, rowNum) -> new OrderSummaryQueryModel(
                UUID.fromString(rs.getString("id")),
                rs.getString("order_number"),
                rs.getString("customer_name"),
                rs.getBigDecimal("total_amount"),
                rs.getString("status"),
                rs.getTimestamp("created_at").toLocalDateTime()
            )
        );
    }
}
```

## Best Practices

### 1. Ubiquitous Language
- Use domain terminology in code
- Names should match business concepts
- Avoid technical jargon in domain layer

```java
// Good - Uses business language
public class Order {
    public void submit() { }
    public void cancel() { }
    public void ship() { }
}

// Bad - Technical language
public class OrderEntity {
    public void persist() { }
    public void setStatus(int status) { }
}
```

### 2. Keep Aggregates Small
```java
// Good - Small aggregate
@Entity
public class Order {
    private OrderId id;
    private CustomerId customerId;  // Reference, not embedded
    private List<OrderLine> orderLines;
}

// Bad - Large aggregate
@Entity
public class Order {
    private OrderId id;
    private Customer customer;  // Full customer object
    private List<OrderLine> orderLines;
    private List<Payment> payments;
    private List<Shipment> shipments;
}
```

### 3. Protect Invariants
```java
@Entity
public class Order {

    public void addOrderLine(ProductId productId, int quantity, Money unitPrice) {
        // Protect invariant: quantity must be positive
        if (quantity <= 0) {
            throw new IllegalArgumentException("Quantity must be positive");
        }

        // Protect invariant: cannot modify submitted order
        if (this.status != OrderStatus.DRAFT) {
            throw new IllegalStateException("Cannot modify submitted order");
        }

        orderLines.add(new OrderLine(productId, quantity, unitPrice));
    }
}
```

### 4. Model True Business Rules
```java
// Domain Service encapsulates business rule
@Service
public class OrderApprovalService {

    public boolean canApproveOrder(Order order, Employee approver) {
        Money orderAmount = order.getTotalAmount();

        // Business rule: Approval limit based on employee level
        if (approver.getLevel() == EmployeeLevel.MANAGER) {
            return orderAmount.isLessThan(Money.of(10000, Currency.USD));
        } else if (approver.getLevel() == EmployeeLevel.DIRECTOR) {
            return orderAmount.isLessThan(Money.of(50000, Currency.USD));
        }

        return false;
    }
}
```

## Quick Reference

### DDD Building Blocks

| Pattern | Purpose | Example |
|---------|---------|---------|
| **Entity** | Has identity, mutable | User, Order, Product |
| **Value Object** | No identity, immutable | Money, Address, Email |
| **Aggregate** | Consistency boundary | Order (root) + OrderLines |
| **Domain Service** | Business logic between entities | TransferMoneyService |
| **Repository** | Persistence abstraction | OrderRepository |
| **Domain Event** | Something happened | OrderPlacedEvent |
| **Factory** | Complex object creation | OrderFactory |

### Strategic vs Tactical Design

**Strategic (Big Picture):**
- Bounded Contexts
- Context Mapping
- Ubiquitous Language
- Domain vision

**Tactical (Implementation):**
- Entities
- Value Objects
- Aggregates
- Domain Services
- Repositories
- Domain Events

---

**Remember:** DDD is about modeling the business domain accurately, not just applying patterns. Understanding the domain is more important than perfect technical implementation.
