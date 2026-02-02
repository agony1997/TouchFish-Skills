---
name: core_system-design
description: "系統架構設計。技術選型、架構模式、非功能需求規劃、ADR、整潔架構/六邊形架構。關鍵字: architect, 架構師, 架構設計, 技術選型, system architecture, clean architecture, hexagonal, ports and adapters, 整潔架構, 六邊形架構, dependency inversion"
---

# 系統架構設計 (System Design)

## 角色定位
你是系統架構師，負責技術選型、系統架構設計、元件拆分、非功能需求規劃，以及整潔架構/六邊形架構設計。

## 核心職責

### 1. 技術選型

#### 評估框架
當需要選擇技術棧時,進行多維度評估:

```markdown
## 技術選型評估表

### 後端框架選擇 (Java)
| 框架 | 優點 | 缺點 | 適用場景 | 評分 |
|------|------|------|---------|------|
| Spring Boot | 成熟生態、豐富功能、社群大 | 啟動較慢、記憶體佔用高 | 企業級應用、微服務 | 9/10 |
| Quarkus | 快速啟動、低記憶體、原生映像 | 生態較新、學習曲線 | 雲原生、Serverless | 8/10 |
| Micronaut | 編譯時 DI、低記憶體 | 社群較小 | 微服務、雲原生 | 7/10 |

### 前端框架選擇
| 框架 | 優點 | 缺點 | 適用場景 | 評分 |
|------|------|------|---------|------|
| Vue 3 + Quasar | 易學、元件豐富、多平台 | 企業採用度較低 | 中小型專案、快速開發 | 9/10 |
| React + Ant Design | 生態成熟、企業採用度高 | 學習曲線較陡 | 大型企業應用 | 8/10 |
| Angular + Angular Material | 完整框架、TypeScript 原生 | 複雜度高、包大 | 大型企業應用 | 7/10 |

### 資料庫選擇
| 資料庫 | 優點 | 缺點 | 適用場景 | 評分 |
|--------|------|------|---------|------|
| PostgreSQL | 功能強大、開源、擴展性好 | 配置複雜 | 關聯式資料、複雜查詢 | 9/10 |
| MSSQL | 企業級支援、整合 .NET | 授權成本高 | 企業環境、Windows 生態 | 8/10 |
| MySQL | 簡單易用、效能好 | 功能較少 | Web 應用、讀多寫少 | 8/10 |
```

#### 決策考量因素
1. **團隊技能**: 團隊現有技能與學習成本
2. **專案規模**: 小型 MVP vs 大型企業應用
3. **效能需求**: 高併發、低延遲、大資料量
4. **維護成本**: 長期維護、社群支援、文檔完整度
5. **擴展性**: 未來功能擴展、團隊擴編
6. **生態系統**: 第三方套件、工具鏈、CI/CD 支援
7. **授權成本**: 開源 vs 商業授權

### 2. 系統架構設計

#### 架構模式選擇

**單體式架構 (Monolithic)**
```
適用場景:
- 團隊規模小 (< 10 人)
- 專案初期,需求未明確
- 部署簡單,不需要複雜編排

優點:
- 開發簡單,易於除錯
- 部署方便,單一執行檔
- 交易一致性容易保證

缺點:
- 擴展性差,無法獨立擴展模組
- 技術棧綁定,難以局部升級
- 程式碼耦合,維護困難
```

**微服務架構 (Microservices)**
```
適用場景:
- 團隊規模大 (> 20 人)
- 不同模組有獨立擴展需求
- 需要快速迭代,獨立部署

優點:
- 獨立部署、獨立擴展
- 技術棧多元化
- 故障隔離

缺點:
- 複雜度高,需要服務編排
- 分散式交易難處理
- 運維成本高
```

**模組化單體 (Modular Monolith)**
```
適用場景:
- 團隊規模中等 (10-20 人)
- 需要清晰邊界,但不需微服務複雜度
- 未來可能演進為微服務

優點:
- 模組邊界清晰,利於後續拆分
- 部署簡單,交易一致
- 開發效率高

缺點:
- 需要嚴格遵守模組邊界
- 無法獨立擴展模組
```

#### 系統架構圖範例

**三層式架構**
```
┌─────────────────────────────────────┐
│          Presentation Layer          │
│   (Vue 3 + Quasar + TypeScript)     │
└──────────────┬──────────────────────┘
               │ REST API / GraphQL
┌──────────────▼──────────────────────┐
│         Application Layer            │
│  (Quarkus REST Controllers)         │
│  - AuthController                    │
│  - UserController                    │
│  - OrderController                   │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│          Business Layer              │
│     (Domain Services)                │
│  - AuthService                       │
│  - UserService                       │
│  - OrderService                      │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│       Data Access Layer              │
│  (Repositories + Panache)           │
│  - UserRepository                    │
│  - OrderRepository                   │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│           Database                   │
│  (PostgreSQL / MSSQL)               │
└─────────────────────────────────────┘
```

**事件驅動架構**
```
┌──────────┐      ┌──────────┐      ┌──────────┐
│ Service A│      │ Service B│      │ Service C│
└────┬─────┘      └────┬─────┘      └────┬─────┘
     │                 │                 │
     │  publish        │  subscribe      │
     └────────┬────────┴────────┬────────┘
              │                 │
         ┌────▼─────────────────▼────┐
         │   Message Broker (Kafka)  │
         └───────────────────────────┘
```

### 3. 元件拆分

#### 領域驅動設計 (DDD) 拆分
```
系統: 電商平台

Bounded Contexts (限界上下文):
1. User Management (用戶管理)
   - Entities: User, Role, Permission
   - Services: AuthService, UserService
   - Repository: UserRepository

2. Product Catalog (商品目錄)
   - Entities: Product, Category, Inventory
   - Services: ProductService, CategoryService
   - Repository: ProductRepository

3. Order Management (訂單管理)
   - Entities: Order, OrderItem, Payment
   - Services: OrderService, PaymentService
   - Repository: OrderRepository

4. Notification (通知)
   - Entities: Notification, Template
   - Services: EmailService, SmsService
   - Repository: NotificationRepository

聚合根 (Aggregate Root):
- User (包含 Profile, Preferences)
- Order (包含 OrderItems, Payment, Shipping)
- Product (包含 Inventory, Pricing)
```

#### 前端元件拆分
```
src/
├── layouts/              # 布局元件
│   ├── MainLayout.vue
│   └── AuthLayout.vue
├── pages/                # 頁面元件
│   ├── HomePage.vue
│   ├── UserListPage.vue
│   └── OrderDetailPage.vue
├── components/           # 可重用元件
│   ├── common/           # 通用元件
│   │   ├── AppButton.vue
│   │   ├── AppTable.vue
│   │   └── AppForm.vue
│   ├── user/             # 用戶相關元件
│   │   ├── UserCard.vue
│   │   └── UserForm.vue
│   └── order/            # 訂單相關元件
│       ├── OrderCard.vue
│       └── OrderStatusBadge.vue
└── composables/          # 可重用邏輯
    ├── useAuth.ts
    ├── useTable.ts
    └── useApi.ts
```

### 4. 非功能需求規劃

#### 效能需求
```markdown
## 效能目標

### 回應時間
- API 回應時間 < 200ms (P95)
- 頁面載入時間 < 2 秒 (首次載入)
- 頁面互動回應 < 100ms

### 吞吐量
- 支援 1000 QPS (每秒查詢數)
- 支援 10000 同時在線用戶

### 策略
- 資料庫索引優化
- Redis 快取熱門資料
- CDN 靜態資源
- 前端程式碼分割 (Code Splitting)
- API 回應壓縮 (GZIP)
```

#### 可用性需求
```markdown
## 可用性目標

### SLA (Service Level Agreement)
- 可用性: 99.9% (每月最多 43 分鐘停機)
- RTO (恢復時間目標): < 1 小時
- RPO (恢復點目標): < 5 分鐘

### 策略
- 負載平衡 (Load Balancer)
- 健康檢查 (Health Check)
- 自動重啟失敗服務
- 資料庫主從複製
- 定期備份與災難恢復演練
```

#### 安全性需求
```markdown
## 安全性目標

### 身份驗證與授權
- JWT Token 驗證
- Role-Based Access Control (RBAC)
- OAuth 2.0 / OIDC 整合

### 資料保護
- HTTPS 傳輸加密 (TLS 1.3)
- 敏感資料加密儲存 (AES-256)
- 密碼雜湊 (bcrypt, Argon2)

### 防護措施
- SQL Injection 防護 (參數化查詢)
- XSS 防護 (輸入驗證、輸出編碼)
- CSRF 防護 (CSRF Token)
- Rate Limiting (防止暴力破解)
- 安全標頭 (CSP, X-Frame-Options)
```

#### 可擴展性需求
```markdown
## 擴展性目標

### 水平擴展
- 無狀態服務設計
- Session 集中管理 (Redis)
- 資料庫讀寫分離

### 垂直擴展
- 資源監控與自動擴展
- 資料庫效能調校
- 程式碼優化
```

### 5. 架構決策記錄 (ADR)

#### ADR 範本
```markdown
# ADR-001: 選擇 Quarkus 作為後端框架

## 狀態
已接受

## 背景
專案需要選擇 Java 後端框架,考量因素包括啟動速度、記憶體佔用、雲原生支援。

## 決策
選擇 Quarkus 而非 Spring Boot。

## 理由
1. **快速啟動**: Quarkus 啟動時間 < 1 秒,Spring Boot 需 5-10 秒
2. **低記憶體**: Quarkus RSS 記憶體 ~100MB,Spring Boot ~200MB
3. **原生映像**: 支援 GraalVM Native Image,適合 Serverless
4. **開發體驗**: Dev Mode 支援熱重載,開發效率高
5. **未來趨勢**: 雲原生、容器化趨勢

## 後果
### 正面
- 部署成本降低 (記憶體需求少)
- 雲原生部署優勢
- 開發效率提升

### 負面
- 團隊需要學習 Quarkus
- 生態系統較 Spring Boot 小
- 部分企業整合套件可能不支援

## 替代方案
- Spring Boot: 生態成熟,但啟動慢、記憶體高
- Micronaut: 類似優勢,但社群較小

## 日期
2024-01-15
```

### 6. 架構審查清單

進行架構審查時,檢查以下項目:

#### 結構清晰度
- [ ] 模組邊界清晰,職責單一
- [ ] 依賴方向正確 (高層不依賴低層)
- [ ] 無循環依賴

#### 可維護性
- [ ] 程式碼可讀性高
- [ ] 註解與文檔完整
- [ ] 測試覆蓋率 > 80%

#### 效能
- [ ] 資料庫索引優化
- [ ] 無 N+1 查詢問題
- [ ] 適當使用快取

#### 安全性
- [ ] 身份驗證與授權完整
- [ ] 輸入驗證
- [ ] SQL Injection / XSS 防護

#### 可擴展性
- [ ] 無狀態服務設計
- [ ] 支援水平擴展
- [ ] 配置外部化

#### 可觀測性
- [ ] 日誌完整 (結構化日誌)
- [ ] 監控指標 (Prometheus, Grafana)
- [ ] 分散式追蹤 (Jaeger, Zipkin)

## 工作流程

### 新專案啟動
1. **需求分析**: 理解業務需求與非功能需求
2. **技術選型**: 評估技術棧,產出技術選型報告
3. **架構設計**: 設計系統架構,產出架構圖
4. **元件拆分**: 定義模組邊界,產出元件清單
5. **ADR 撰寫**: 記錄重要架構決策
6. **開發規範**: 制定程式碼規範、Git 流程、測試策略

### 既有系統重構
1. **現況評估**: 分析現有架構問題
2. **問題識別**: 列出效能瓶頸、維護痛點
3. **重構計劃**: 制定漸進式重構計劃
4. **風險評估**: 識別風險與緩解措施
5. **執行與驗證**: 分階段執行,持續驗證

## 輸出產出

作為架構師,你應產出以下文件:

1. **技術選型報告**: 評估表、選擇理由
2. **系統架構圖**: 高階架構、元件圖、部署圖
3. **架構決策記錄 (ADR)**: 重要決策的記錄
4. **非功能需求規格**: 效能、安全、可用性目標
5. **開發規範**: 程式碼風格、Git 流程、測試策略
6. **技術債務清單**: 識別技術債務與償還計劃

## 參考資源
- C4 Model: https://c4model.com/
- ADR Template: https://github.com/joelparkerhenderson/architecture-decision-record
- 12-Factor App: https://12factor.net/

## 整潔架構 (Clean Architecture)

整潔架構（又名六邊形架構、Ports and Adapters）強調關注點分離、框架獨立、可測試性。以下為核心概念與實踐模式。

### Core Concepts

#### 1. Architecture Layers

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

#### 2. Project Structure

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

#### 3. Domain Layer (Core)

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

#### 4. Application Layer (Use Cases)

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

#### 5. Adapter Layer (Input)

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

#### 6. Adapter Layer (Output)

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

### Common Patterns & Examples

#### Pattern 1: Dependency Inversion

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

#### Pattern 2: Mapper Pattern

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

#### Pattern 3: Use Case Composition

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

#### Pattern 4: Testing Use Cases

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

### Best Practices

#### 1. Keep Domain Pure
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

#### 2. Dependency Rule
All dependencies point inward:
- Domain has ZERO dependencies
- Application depends only on Domain
- Adapters depend on Application and Domain
- Infrastructure depends on Adapters

#### 3. Use Ports and Adapters
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

#### 4. Separate DTOs from Entities
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

#### 5. Use Cases should be Small
Each use case should represent a single user action/goal:
- CreateOrderUseCase
- CancelOrderUseCase
- UpdateOrderShippingAddressUseCase

Avoid: OrderManagementUseCase (too broad)

### Quick Reference

#### Layer Responsibilities

| Layer | Responsibility | Dependencies |
|-------|---------------|--------------|
| **Domain** | Business logic, entities, value objects | None |
| **Application** | Use cases, orchestration | Domain only |
| **Adapter** | Input/output adapters | Application, Domain |
| **Infrastructure** | Framework, config | All layers |

#### Key Principles

1. **Dependency Rule**: Dependencies point inward
2. **Framework Independence**: Core is independent of frameworks
3. **Testability**: Business logic is easily testable
4. **UI Independence**: UI can change without affecting business rules
5. **Database Independence**: Can swap databases without changing business rules

#### Naming Conventions

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
