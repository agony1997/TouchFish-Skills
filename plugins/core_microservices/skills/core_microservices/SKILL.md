---
name: core_microservices
description: "微服務架構模式專家。專精於服務拆分、服務通訊、API Gateway、服務治理、分散式事務、微服務安全、可觀測性。關鍵字: microservices, 微服務, service mesh, api gateway, distributed system, 分散式系統, 服務治理"
---

# Microservices Architecture Expert

You are a Microservices Architecture Expert specializing in designing, implementing, and operating distributed microservices systems.

## Overview

Microservices architecture structures an application as a collection of loosely coupled, independently deployable services. Each service is focused on a specific business capability and can be developed, deployed, and scaled independently.

**Core capabilities:**
- Service decomposition strategies
- Inter-service communication (REST, gRPC, messaging)
- API Gateway patterns
- Service discovery and load balancing
- Distributed transaction management (Saga pattern)
- Microservices security
- Observability (logging, metrics, tracing)

## When to use this skill

Activate this skill when users:
- Design microservices architecture (關鍵字: "microservices", "微服務架構", "服務拆分")
- Implement service communication (關鍵字: "service communication", "服務通訊", "api gateway")
- Handle distributed transactions (關鍵字: "distributed transaction", "分散式事務", "saga")
- Need service discovery (關鍵字: "service discovery", "服務發現", "consul")
- Implement observability (關鍵字: "observability", "可觀測性", "monitoring")

## Core Concepts

### 1. Service Decomposition

**By Business Capability:**
```
E-commerce System
├── User Service (Authentication, Profile)
├── Product Service (Catalog, Inventory)
├── Order Service (Order Management)
├── Payment Service (Payment Processing)
├── Shipping Service (Delivery Management)
└── Notification Service (Email, SMS)
```

**By Subdomain (DDD approach):**
```
Core Domain:
- Order Management Service
- Product Catalog Service

Supporting Domain:
- User Management Service
- Payment Service

Generic Domain:
- Notification Service
- Reporting Service
```

### 2. Service Communication Patterns

**Synchronous - REST:**
```java
@RestController
@RequestMapping("/api/orders")
public class OrderController {

    @Autowired
    private RestTemplate restTemplate;

    @PostMapping
    public OrderDTO createOrder(@RequestBody OrderRequest request) {
        // Call User Service
        String userServiceUrl = "http://user-service/api/users/" + request.getUserId();
        UserDTO user = restTemplate.getForObject(userServiceUrl, UserDTO.class);

        // Call Product Service
        String productServiceUrl = "http://product-service/api/products/" + request.getProductId();
        ProductDTO product = restTemplate.getForObject(productServiceUrl, ProductDTO.class);

        // Create order
        Order order = new Order(user, product);
        return orderService.create(order);
    }
}
```

**Synchronous - gRPC:**
```protobuf
syntax = "proto3";

service UserService {
  rpc GetUser (GetUserRequest) returns (UserResponse);
  rpc CreateUser (CreateUserRequest) returns (UserResponse);
}

message GetUserRequest {
  int64 user_id = 1;
}

message UserResponse {
  int64 id = 1;
  string name = 2;
  string email = 3;
}
```

**Asynchronous - Message Queue:**
```java
// Publisher
@Service
public class OrderService {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void createOrder(Order order) {
        orderRepository.save(order);

        // Publish event
        OrderCreatedEvent event = new OrderCreatedEvent(order.getId(), order.getUserId());
        rabbitTemplate.convertAndSend("order.exchange", "order.created", event);
    }
}

// Consumer
@Component
public class InventoryEventListener {

    @RabbitListener(queues = "inventory.queue")
    public void handleOrderCreated(OrderCreatedEvent event) {
        // Reserve inventory
        inventoryService.reserve(event.getOrderId());
    }
}
```

### 3. API Gateway Pattern

**Spring Cloud Gateway:**
```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: user-service
          uri: lb://user-service
          predicates:
            - Path=/api/users/**
          filters:
            - StripPrefix=1
            - AuthenticationFilter

        - id: order-service
          uri: lb://order-service
          predicates:
            - Path=/api/orders/**
          filters:
            - StripPrefix=1
            - RateLimitFilter
```

**Custom Gateway Filter:**
```java
@Component
public class AuthenticationFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String token = exchange.getRequest().getHeaders().getFirst("Authorization");

        if (token == null || !validateToken(token)) {
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }

        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return -100;
    }
}
```

### 4. Distributed Transaction - Saga Pattern

**Choreography-based Saga:**
```java
// Order Service
@Service
public class OrderService {

    @Autowired
    private EventPublisher eventPublisher;

    @Transactional
    public void createOrder(OrderRequest request) {
        Order order = new Order(request);
        order.setStatus(OrderStatus.PENDING);
        orderRepository.save(order);

        // Publish event
        eventPublisher.publish(new OrderCreatedEvent(order.getId()));
    }

    @EventListener
    public void handlePaymentCompleted(PaymentCompletedEvent event) {
        Order order = orderRepository.findById(event.getOrderId());
        order.setStatus(OrderStatus.PAID);
        orderRepository.save(order);

        eventPublisher.publish(new OrderPaidEvent(order.getId()));
    }

    @EventListener
    public void handlePaymentFailed(PaymentFailedEvent event) {
        Order order = orderRepository.findById(event.getOrderId());
        order.setStatus(OrderStatus.CANCELLED);
        orderRepository.save(order);
    }
}

// Payment Service
@Service
public class PaymentService {

    @EventListener
    public void handleOrderCreated(OrderCreatedEvent event) {
        try {
            Payment payment = processPayment(event.getOrderId());
            eventPublisher.publish(new PaymentCompletedEvent(event.getOrderId()));
        } catch (Exception e) {
            eventPublisher.publish(new PaymentFailedEvent(event.getOrderId()));
        }
    }
}
```

**Orchestration-based Saga:**
```java
@Service
public class OrderSagaOrchestrator {

    public void createOrder(OrderRequest request) {
        String sagaId = UUID.randomUUID().toString();

        try {
            // Step 1: Create order
            Order order = orderService.createOrder(request);

            // Step 2: Reserve inventory
            inventoryService.reserve(order.getId());

            // Step 3: Process payment
            paymentService.process(order.getId());

            // Step 4: Arrange shipping
            shippingService.arrange(order.getId());

            // Success
            order.setStatus(OrderStatus.COMPLETED);

        } catch (Exception e) {
            // Compensating transactions
            compensate(sagaId);
        }
    }

    private void compensate(String sagaId) {
        shippingService.cancel(sagaId);
        paymentService.refund(sagaId);
        inventoryService.release(sagaId);
        orderService.cancel(sagaId);
    }
}
```

### 5. Service Discovery

**Eureka Client:**
```java
@SpringBootApplication
@EnableDiscoveryClient
public class UserServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}
```

```yaml
eureka:
  client:
    service-url:
      defaultZone: http://localhost:8761/eureka/
  instance:
    prefer-ip-address: true
    instance-id: ${spring.application.name}:${spring.application.instance_id:${random.value}}
```

### 6. Circuit Breaker

**Resilience4j:**
```java
@Service
public class OrderService {

    @CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
    @Retry(name = "paymentService")
    @TimeLimiter(name = "paymentService")
    public CompletableFuture<Payment> processPayment(Order order) {
        return CompletableFuture.supplyAsync(() ->
            paymentClient.process(order.getId(), order.getTotalAmount())
        );
    }

    private CompletableFuture<Payment> paymentFallback(Order order, Exception ex) {
        log.error("Payment service unavailable for order: {}", order.getId());
        return CompletableFuture.completedFuture(
            Payment.failed(order.getId(), "Service temporarily unavailable")
        );
    }
}
```

### 7. Distributed Tracing

**Configuration:**
```yaml
management:
  tracing:
    sampling:
      probability: 1.0
  zipkin:
    tracing:
      endpoint: http://localhost:9411/api/v2/spans

logging:
  pattern:
    level: '%5p [${spring.application.name:},%X{traceId:-},%X{spanId:-}]'
```

### 8. Centralized Configuration

**Config Server:**
```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/your-org/config-repo
          default-label: main
```

**Config Client:**
```yaml
spring:
  application:
    name: user-service
  config:
    import: configserver:http://localhost:8888
```

## Common Patterns & Examples

### Pattern 1: Database per Service

```
User Service → User Database (PostgreSQL)
Order Service → Order Database (MySQL)
Product Service → Product Database (MongoDB)
```

**Advantages:**
- Loose coupling
- Independent scaling
- Technology diversity

**Challenges:**
- Data consistency
- Complex queries across services

### Pattern 2: Shared Database (Anti-pattern)

Avoid sharing databases between services as it creates tight coupling.

### Pattern 3: Event-Driven Architecture

```java
// Event Store
@Entity
public class DomainEvent {
    @Id
    private String id;
    private String aggregateId;
    private String eventType;
    private String payload;
    private LocalDateTime occurredAt;
}

// Event Publisher
@Service
public class EventPublisher {

    @Autowired
    private KafkaTemplate<String, DomainEvent> kafkaTemplate;

    public void publish(DomainEvent event) {
        kafkaTemplate.send("domain-events", event.getAggregateId(), event);
    }
}

// Event Listener
@Component
public class OrderEventListener {

    @KafkaListener(topics = "domain-events", groupId = "order-service")
    public void handleEvent(DomainEvent event) {
        if ("PaymentCompleted".equals(event.getEventType())) {
            orderService.markAsPaid(event.getAggregateId());
        }
    }
}
```

### Pattern 4: CQRS (Command Query Responsibility Segregation)

```java
// Command Side
@Service
public class OrderCommandService {

    public OrderId createOrder(CreateOrderCommand command) {
        Order order = new Order(command);
        orderRepository.save(order);
        eventPublisher.publish(new OrderCreatedEvent(order.getId()));
        return order.getId();
    }
}

// Query Side (Denormalized)
@Service
public class OrderQueryService {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    public List<OrderSummary> findRecentOrders() {
        return jdbcTemplate.query(
            "SELECT * FROM order_summary_view ORDER BY created_at DESC LIMIT 10",
            (rs, rowNum) -> new OrderSummary(...)
        );
    }
}
```

## Best Practices

### 1. Service Independence
- Each service should have its own database
- Services should be deployable independently
- Avoid shared libraries (prefer API contracts)

### 2. API Versioning
```java
@RestController
@RequestMapping("/api/v1/users")
public class UserV1Controller {
    // Version 1 API
}

@RestController
@RequestMapping("/api/v2/users")
public class UserV2Controller {
    // Version 2 API
}
```

### 3. Health Checks
```java
@Component
public class CustomHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {
        if (checkDatabaseConnection() && checkExternalService()) {
            return Health.up()
                .withDetail("database", "Connected")
                .withDetail("externalService", "Available")
                .build();
        }
        return Health.down().build();
    }
}
```

### 4. Graceful Degradation
```java
@Service
public class RecommendationService {

    @CircuitBreaker(name = "mlService", fallbackMethod = "getDefaultRecommendations")
    public List<Product> getRecommendations(Long userId) {
        return mlServiceClient.getRecommendations(userId);
    }

    private List<Product> getDefaultRecommendations(Long userId, Exception ex) {
        // Return popular products as fallback
        return productService.getPopularProducts();
    }
}
```

### 5. Idempotency
```java
@Service
public class PaymentService {

    public Payment processPayment(String idempotencyKey, PaymentRequest request) {
        // Check if already processed
        Optional<Payment> existing = paymentRepository.findByIdempotencyKey(idempotencyKey);
        if (existing.isPresent()) {
            return existing.get();
        }

        // Process payment
        Payment payment = new Payment(idempotencyKey, request);
        return paymentRepository.save(payment);
    }
}
```

### 6. Monitoring and Alerting
```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  metrics:
    export:
      prometheus:
        enabled: true
    tags:
      application: ${spring.application.name}
```

### 7. Security
```java
// JWT Token Validation in Gateway
@Component
public class JwtAuthenticationFilter implements GlobalFilter {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String token = extractToken(exchange.getRequest());

        if (!jwtUtil.validateToken(token)) {
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }

        // Add user context to request headers
        ServerHttpRequest modifiedRequest = exchange.getRequest().mutate()
            .header("X-User-Id", jwtUtil.getUserId(token))
            .build();

        return chain.filter(exchange.mutate().request(modifiedRequest).build());
    }
}
```

## Quick Reference

### Microservices Characteristics

| Characteristic | Description |
|----------------|-------------|
| **Independently Deployable** | Each service can be deployed without affecting others |
| **Organized around Business Capabilities** | Services align with business domains |
| **Decentralized Data Management** | Each service owns its data |
| **Infrastructure Automation** | CI/CD, containerization |
| **Design for Failure** | Circuit breakers, retries, fallbacks |

### Communication Patterns

| Pattern | Use Case | Technology |
|---------|----------|------------|
| **Synchronous REST** | Simple queries, CRUD | Spring Web, RestTemplate |
| **gRPC** | High-performance, type-safe | gRPC, Protocol Buffers |
| **Async Messaging** | Event-driven, decoupling | RabbitMQ, Kafka |
| **GraphQL** | Flexible queries, mobile apps | Spring GraphQL |

### Common Anti-patterns

1. **Distributed Monolith** - Services too coupled
2. **Chatty Services** - Too many service calls
3. **Shared Database** - Violates independence
4. **No API Gateway** - Client complexity
5. **Lack of Monitoring** - Can't troubleshoot

### Tools & Technologies

- **Service Discovery:** Eureka, Consul, Kubernetes
- **API Gateway:** Spring Cloud Gateway, Kong, Nginx
- **Message Queue:** RabbitMQ, Kafka, AWS SQS
- **Tracing:** Zipkin, Jaeger, AWS X-Ray
- **Monitoring:** Prometheus, Grafana, ELK

---

**Remember:** Microservices add complexity. Only adopt this architecture when you need independent scaling, deployment, and team autonomy.
