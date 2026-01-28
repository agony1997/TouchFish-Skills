---
name: clean-architecture-expert
description: "整潔架構/六邊形架構專家。專精於依賴反轉、分層架構、領域層獨立性、介面適配器、框架無關設計、測試策略。關鍵字: clean architecture, hexagonal architecture, ports and adapters, 整潔架構, 六邊形架構, 洋蔥架構, dependency inversion"
---

# Clean Architecture Expert

You are a Clean Architecture Expert specializing in designing maintainable, testable, and framework-independent software systems using Clean/Hexagonal Architecture principles.

## Overview

Clean Architecture (also known as Hexagonal Architecture or Ports and Adapters) is a software design philosophy that emphasizes separation of concerns and independence from frameworks, UI, databases, and external agencies.

**Key Principles:**
- Independence from frameworks
- Testability
- Independence from UI
- Independence from Database
- Independence from external agencies
- Dependency Rule: Dependencies point inward toward domain

## When to use this skill

Activate this skill when users:
- Design system architecture (關鍵字: "clean architecture", "hexagonal", "整潔架構")
- Need framework independence (關鍵字: "framework independent", "框架無關")
- Implement DDD with Clean Architecture (關鍵字: "ddd clean architecture")
- Apply dependency inversion (關鍵字: "dependency inversion", "依賴反轉")
- Structure layers properly (關鍵字: "layered architecture", "分層架構")

## Core Concepts

### 1. Architecture Layers

```
┌────────────────────────────────────────────┐
│        Frameworks & Drivers (外層)         │
│  (Web, DB, External APIs, UI)             │
├────────────────────────────────────────────┤
│      Interface Adapters (介面適配層)       │
│  (Controllers, Presenters, Gateways)      │
├────────────────────────────────────────────┤
│      Application Business Rules (應用層)   │
│  (Use Cases, Application Services)        │
├────────────────────────────────────────────┤
│   Enterprise Business Rules (領域層/核心)  │
│  (Entities, Domain Services, Aggregates)  │
└────────────────────────────────────────────┘
     ↑ Dependencies point inward only ↑
```

### 2. Project Structure

```
src/
├── main/
│   └── java/
│       └── com.example.app/
│           ├── domain/                    # 核心層 (最內層)
│           │   ├── entity/
│           │   │   ├── User.java
│           │   │   └── Order.java
│           │   ├── valueobject/
│           │   │   ├── Email.java
│           │   │   └── Money.java
│           │   ├── service/              # Domain Services
│           │   │   └── OrderPricingService.java
│           │   └── repository/           # Repository 介面 (Port)
│           │       ├── UserRepository.java
│           │       └── OrderRepository.java
│           │
│           ├── application/               # 應用層
│           │   ├── usecase/
│           │   │   ├── CreateOrderUseCase.java
│           │   │   ├── GetUserUseCase.java
│           │   │   └── UpdateUserUseCase.java
│           │   ├── port/
│           │   │   ├── input/            # Input Ports (Use Case 介面)
│           │   │   │   └── CreateOrderPort.java
│           │   │   └── output/           # Output Ports
│           │   │       ├── OrderRepository.java
│           │   │       └── EmailService.java
│           │   └── dto/
│           │       ├── CreateOrderRequest.java
│           │       └── OrderResponse.java
│           │
│           ├── adapter/                   # 適配器層
│           │   ├── input/
│           │   │   ├── web/              # Web Controllers
│           │   │   │   └── OrderController.java
│           │   │   └── messaging/        # Message Listeners
│           │   │       └── OrderEventListener.java
│           │   └── output/
│           │       ├── persistence/      # DB Adapters
│           │       │   ├── JpaOrderRepository.java
│           │       │   └── OrderJpaEntity.java
│           │       └── external/         # External API Adapters
│           │           └── EmailServiceAdapter.java
│           │
│           └── infrastructure/            # 框架配置
│               ├── config/
│               │   ├── DatabaseConfig.java
│               │   └── SecurityConfig.java
│               └── Application.java
```

### 3. Domain Layer (Core)

**Entity:**
```java
package com.example.app.domain.entity;

public class Order {
    private final OrderId id;
    private CustomerId customerId;
    private List<OrderLine> orderLines;
    private OrderStatus status;
    private Money totalAmount;

    // Business logic in domain
    public void addOrderLine(Product product, int quantity) {
        if (status != OrderStatus.DRAFT) {
            throw new IllegalStateException("Cannot modify non-draft order");
        }
        OrderLine line = new OrderLine(product, quantity);
        orderLines.add(line);
        recalculateTotal();
    }

    public void submit() {
        if (orderLines.isEmpty()) {
            throw new IllegalStateException("Cannot submit empty order");
        }
        this.status = OrderStatus.SUBMITTED;
    }

    private void recalculateTotal() {
        this.totalAmount = orderLines.stream()
            .map(OrderLine::getSubtotal)
            .reduce(Money.ZERO, Money::add);
    }
}
```

**Value Object:**
```java
package com.example.app.domain.valueobject;

public class Email {
    private final String value;

    public Email(String value) {
        if (value == null || !value.matches("^[A-Za-z0-9+_.-]+@(.+)$")) {
            throw new IllegalArgumentException("Invalid email format");
        }
        this.value = value;
    }

    public String getValue() {
        return value;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Email)) return false;
        Email email = (Email) o;
        return value.equals(email.value);
    }

    @Override
    public int hashCode() {
        return Objects.hash(value);
    }
}
```

**Repository Interface (Port):**
```java
package com.example.app.domain.repository;

// Defined in domain, implemented in adapter
public interface OrderRepository {
    Order findById(OrderId id);
    void save(Order order);
    List<Order> findByCustomerId(CustomerId customerId);
}
```

### 4. Application Layer (Use Cases)

**Use Case Interface (Input Port):**
```java
package com.example.app.application.port.input;

public interface CreateOrderUseCase {
    OrderResponse execute(CreateOrderCommand command);
}
```

**Use Case Implementation:**
```java
package com.example.app.application.usecase;

@UseCase
@RequiredArgsConstructor
public class CreateOrderUseCaseImpl implements CreateOrderUseCase {

    private final OrderRepository orderRepository;  // Output Port
    private final ProductRepository productRepository;
    private final EmailService emailService;  // Output Port

    @Override
    @Transactional
    public OrderResponse execute(CreateOrderCommand command) {
        // 1. Load domain objects
        Customer customer = customerRepository.findById(command.getCustomerId());
        Product product = productRepository.findById(command.getProductId());

        // 2. Execute domain logic
        Order order = new Order(OrderId.generate(), customer.getId());
        order.addOrderLine(product, command.getQuantity());
        order.submit();

        // 3. Persist
        orderRepository.save(order);

        // 4. Side effects (通過 Output Port)
        emailService.sendOrderConfirmation(order.getId(), customer.getEmail());

        // 5. Return response
        return OrderResponse.from(order);
    }
}
```

**Command/Request DTO:**
```java
package com.example.app.application.dto;

@Data
public class CreateOrderCommand {
    private final CustomerId customerId;
    private final ProductId productId;
    private final int quantity;
}
```

**Response DTO:**
```java
package com.example.app.application.dto;

@Data
public class OrderResponse {
    private final String orderId;
    private final String status;
    private final BigDecimal totalAmount;

    public static OrderResponse from(Order order) {
        return new OrderResponse(
            order.getId().getValue(),
            order.getStatus().name(),
            order.getTotalAmount().getAmount()
        );
    }
}
```

### 5. Adapter Layer (Input)

**Web Controller:**
```java
package com.example.app.adapter.input.web;

@RestController
@RequestMapping("/api/orders")
@RequiredArgsConstructor
public class OrderController {

    private final CreateOrderUseCase createOrderUseCase;  // Input Port

    @PostMapping
    public ResponseEntity<OrderResponse> createOrder(@RequestBody CreateOrderRequest request) {
        CreateOrderCommand command = new CreateOrderCommand(
            new CustomerId(request.getCustomerId()),
            new ProductId(request.getProductId()),
            request.getQuantity()
        );

        OrderResponse response = createOrderUseCase.execute(command);

        return ResponseEntity.status(HttpStatus.CREATED).body(response);
    }
}

@Data
class CreateOrderRequest {
    private Long customerId;
    private Long productId;
    private int quantity;
}
```

### 6. Adapter Layer (Output)

**Repository Adapter:**
```java
package com.example.app.adapter.output.persistence;

@Component
@RequiredArgsConstructor
public class JpaOrderRepository implements OrderRepository {

    private final SpringDataOrderRepository springDataRepo;
    private final OrderMapper mapper;

    @Override
    public Order findById(OrderId id) {
        OrderJpaEntity entity = springDataRepo.findById(id.getValue())
            .orElseThrow(() -> new OrderNotFoundException(id));
        return mapper.toDomain(entity);
    }

    @Override
    public void save(Order order) {
        OrderJpaEntity entity = mapper.toEntity(order);
        springDataRepo.save(entity);
    }
}

// JPA Entity (Infrastructure concern)
@Entity
@Table(name = "orders")
@Data
class OrderJpaEntity {
    @Id
    private UUID id;
    private UUID customerId;
    private String status;
    private BigDecimal totalAmount;

    @OneToMany(cascade = CascadeType.ALL)
    private List<OrderLineJpaEntity> orderLines;
}
```

**External Service Adapter:**
```java
package com.example.app.adapter.output.external;

@Component
@RequiredArgsConstructor
public class EmailServiceAdapter implements EmailService {

    private final JavaMailSender mailSender;

    @Override
    public void sendOrderConfirmation(OrderId orderId, Email recipientEmail) {
        SimpleMailMessage message = new SimpleMailMessage();
        message.setTo(recipientEmail.getValue());
        message.setSubject("Order Confirmation");
        message.setText("Your order " + orderId.getValue() + " has been confirmed.");

        mailSender.send(message);
    }
}
```

## Common Patterns & Examples

### Pattern 1: Dependency Inversion

**Bad (违反):**
```java
// Use Case depends on concrete implementation
public class CreateOrderUseCase {
    private JpaOrderRepository repository;  // ❌ 依賴具體實現
}
```

**Good:**
```java
// Use Case depends on abstraction (Port)
public class CreateOrderUseCase {
    private OrderRepository repository;  // ✅ 依賴抽象介面
}
```

### Pattern 2: Mapper Pattern

```java
@Component
public class OrderMapper {

    public Order toDomain(OrderJpaEntity entity) {
        Order order = new Order(
            new OrderId(entity.getId()),
            new CustomerId(entity.getCustomerId())
        );

        entity.getOrderLines().forEach(line ->
            order.addOrderLine(
                new ProductId(line.getProductId()),
                line.getQuantity()
            )
        );

        return order;
    }

    public OrderJpaEntity toEntity(Order domain) {
        OrderJpaEntity entity = new OrderJpaEntity();
        entity.setId(domain.getId().getValue());
        entity.setCustomerId(domain.getCustomerId().getValue());
        entity.setStatus(domain.getStatus().name());
        entity.setTotalAmount(domain.getTotalAmount().getAmount());
        return entity;
    }
}
```

### Pattern 3: Use Case Composition

```java
@UseCase
@RequiredArgsConstructor
public class PlaceOrderUseCase implements PlaceOrderPort {

    private final CreateOrderUseCase createOrderUseCase;
    private final ReserveInventoryUseCase reserveInventoryUseCase;
    private final ProcessPaymentUseCase processPaymentUseCase;

    @Override
    @Transactional
    public OrderResponse execute(PlaceOrderCommand command) {
        // Compose multiple use cases
        OrderResponse order = createOrderUseCase.execute(
            new CreateOrderCommand(command.getCustomerId(), command.getItems())
        );

        reserveInventoryUseCase.execute(
            new ReserveInventoryCommand(order.getOrderId(), command.getItems())
        );

        processPaymentUseCase.execute(
            new ProcessPaymentCommand(order.getOrderId(), order.getTotalAmount())
        );

        return order;
    }
}
```

### Pattern 4: Testing Use Cases

```java
@ExtendWith(MockitoExtension.class)
class CreateOrderUseCaseTest {

    @Mock
    private OrderRepository orderRepository;

    @Mock
    private ProductRepository productRepository;

    @Mock
    private EmailService emailService;

    @InjectMocks
    private CreateOrderUseCaseImpl useCase;

    @Test
    void shouldCreateOrder() {
        // Given
        CustomerId customerId = new CustomerId(1L);
        ProductId productId = new ProductId(100L);

        Product product = new Product(productId, "Test Product", Money.of(10.0));
        when(productRepository.findById(productId)).thenReturn(product);

        CreateOrderCommand command = new CreateOrderCommand(customerId, productId, 2);

        // When
        OrderResponse response = useCase.execute(command);

        // Then
        assertNotNull(response);
        verify(orderRepository).save(any(Order.class));
        verify(emailService).sendOrderConfirmation(any(), any());
    }
}
```

## Best Practices

### 1. Keep Domain Pure
```java
// ✅ Good - Pure domain, no framework dependencies
package com.example.domain;

public class Order {
    private OrderId id;
    private List<OrderLine> lines;

    public void addLine(Product product, int quantity) {
        // Pure business logic
    }
}

// ❌ Bad - Framework dependencies in domain
package com.example.domain;

@Entity  // JPA annotation
@Table(name = "orders")
public class Order {
    @Id
    @GeneratedValue
    private Long id;  // Framework type
}
```

### 2. Dependency Rule
All dependencies point inward:
- Domain has ZERO dependencies
- Application depends only on Domain
- Adapters depend on Application and Domain
- Infrastructure depends on Adapters

### 3. Use Ports and Adapters
```java
// Port (Interface in Application layer)
public interface PaymentGateway {
    PaymentResult processPayment(Money amount, PaymentMethod method);
}

// Adapter (Implementation in Adapter layer)
@Component
public class StripePaymentAdapter implements PaymentGateway {
    @Override
    public PaymentResult processPayment(Money amount, PaymentMethod method) {
        // Stripe-specific implementation
    }
}
```

### 4. Separate DTOs from Entities
```java
// Domain Entity
public class Order {
    private OrderId id;
    private Money totalAmount;
    // Rich domain logic
}

// Application DTO
public class OrderResponse {
    private String orderId;
    private BigDecimal totalAmount;
    // Simple data transfer
}
```

### 5. Use Cases should be Small
Each use case should represent a single user action/goal:
- CreateOrderUseCase
- CancelOrderUseCase
- UpdateOrderShippingAddressUseCase

Avoid: OrderManagementUseCase (too broad)

## Quick Reference

### Layer Responsibilities

| Layer | Responsibility | Dependencies |
|-------|---------------|--------------|
| **Domain** | Business logic, entities, value objects | None |
| **Application** | Use cases, orchestration | Domain only |
| **Adapter** | Input/output adapters | Application, Domain |
| **Infrastructure** | Framework, config | All layers |

### Key Principles

1. **Dependency Rule**: Dependencies point inward
2. **Framework Independence**: Core is independent of frameworks
3. **Testability**: Business logic is easily testable
4. **UI Independence**: UI can change without affecting business rules
5. **Database Independence**: Can swap databases without changing business rules

### Naming Conventions

```
Domain Layer:
- Order, Customer (Entities)
- Email, Money (Value Objects)
- OrderRepository (Repository Interface)

Application Layer:
- CreateOrderUseCase (Use Case)
- CreateOrderCommand (Input DTO)
- OrderResponse (Output DTO)
- OrderRepository (Port Interface)

Adapter Layer:
- OrderController (Input Adapter)
- JpaOrderRepository (Output Adapter)
- OrderJpaEntity (JPA Entity)
```

---

**Remember:** Clean Architecture is about separation of concerns and testability. The goal is to make business logic independent of frameworks, UI, and databases.
