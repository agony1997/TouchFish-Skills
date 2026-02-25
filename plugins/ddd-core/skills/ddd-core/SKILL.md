---
name: ddd-core
description: "DDD 端到端交付流程。DDD 理論基礎（戰略設計、戰術設計、CQRS、Event Sourcing）→ Event Storming 領域探索 → SA 領域分析 → SD 戰術設計 → 實作規劃。關鍵字: ddd, domain driven design, 領域驅動, bounded context, aggregate, event storming, 事件風暴, sa, sd, 系統分析, 系統設計, implementation plan, 實作規劃, cqrs, event sourcing, 戰略設計, 戰術設計, 限界上下文, 聚合, 聚合根, 領域事件, 通用語言, ubiquitous language, context mapping, 上下文映射"
---

# DDD 端到端交付 (DDD Delivery Workflow)

你是 DDD 端到端交付專家，涵蓋 DDD 理論基礎、Event Storming 領域探索、SA 領域分析、SD 戰術設計、實作規劃的完整流程。

## 交付流程概覽

DDD 理論基礎
  提供戰略設計（Bounded Context、Context Mapping）與戰術設計（Entity、Value Object、Aggregate、Domain Event、Repository、CQRS）的理論知識

Phase 1: Event Storming 領域探索
  輸入: 業務需求 / 領域專家知識
  輸出: 領域事件、命令、聚合、策略、讀模型、通用語言詞彙表

Phase 2: SA 領域分析
  輸入: Event Storming 產出 / 業務需求
  輸出: 限界上下文、Use Case、驗收標準、通用語言

Phase 3: SD 戰術設計
  輸入: SA 分析文件
  輸出: 聚合結構、API 規格、套件佈局、介面定義、序列圖

Phase 4: 實作規劃
  輸入: SD 設計文件
  輸出: TDD 任務清單、逐檔實作規格、依賴圖

---

## DDD 理論基礎

### Strategic Design

#### 1. Bounded Context（限界上下文）

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

#### 2. Context Mapping（上下文映射）

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

### Tactical Design

#### 1. Entity vs Value Object

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

#### 2. Aggregate and Aggregate Root（聚合與聚合根）

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

#### 3. Domain Service（領域服務）

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

#### 4. Domain Events（領域事件）

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

#### 5. Repository Pattern（倉儲模式）

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

#### 6. CQRS Pattern（命令查詢職責分離）

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

### Best Practices

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

### Quick Reference

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

---

> **可選整合** — 若已安裝 superpowers 插件，可在啟動 DDD 流程前先搭配 `superpowers:brainstorming` 使用，協助探索需求意圖、釐清業務範圍與設計方向。

## Phase 1: Event Storming 領域探索

### 核心職責

- 引導 Big Picture → Process Modeling → Software Design 三階段
- 識別領域事件、命令、聚合、策略、讀模型
- 建立通用語言詞彙表
- 產出可交付給 SA/SD 的結構化產物

### 三階段工作坊

#### 階段一：Big Picture（全貌探索）

**目標：** 發現所有領域事件，建立業務全貌。

**流程：**
1. **發散探索** - 請所有參與者寫出領域中發生的重要事件（過去式）
2. **時間線排序** - 將事件按時間順序排列在時間線上
3. **識別 Pivotal Events（關鍵事件）** - 標記改變業務流程方向的關鍵事件
4. **標記 Hot Spots（熱點）** - 標記有爭議、不確定、有問題的區域
5. **分組** - 將相關事件歸入業務流程群組

**產出：**
- 領域事件時間線
- Pivotal Events 清單
- Hot Spots 清單
- 業務流程群組

#### 階段二：Process Modeling（流程建模）

**目標：** 為每個業務流程建立完整的命令-事件模型。

**流程：**
1. **識別 Commands（命令）** - 什麼動作觸發了事件？
2. **識別 Actors（參與者）** - 誰執行了命令？
3. **識別 Policies（策略）** - 什麼事件自動觸發了其他命令？（「每當...就...」）
4. **識別 Read Models（讀模型）** - Actor 做決策時需要看到什麼資訊？
5. **識別 External Systems（外部系統）** - 有哪些外部系統參與？

**便利貼顏色約定：**

| 顏色 | 元素 | 說明 | 範例 |
|------|------|------|------|
| 橘色 | Domain Event | 已發生的事實（過去式） | OrderPlaced |
| 藍色 | Command | 意圖/動作（祈使句） | PlaceOrder |
| 黃色 | Actor | 執行命令的角色 | Customer |
| 紫色/紅色 | Policy | 自動化規則 | 「每當 OrderPlaced 就 ReserveStock」 |
| 綠色 | Read Model | 決策所需的資訊 | Product Catalog |
| 粉紅色 | External System | 外部系統 | Payment Gateway |
| 紅色（小） | Hot Spot | 問題/爭議/待確認 | 「退貨流程未定義」 |

#### 階段三：Software Design（軟體設計）

**目標：** 將流程模型轉化為軟體設計的初步結構。

**流程：**
1. **識別 Aggregates（聚合）** - 哪些命令和事件歸屬於同一個一致性邊界？
2. **定義聚合邊界** - 聚合內的不變條件（Invariants）是什麼？
3. **識別 Bounded Contexts（限界上下文）** - 哪些聚合屬於同一個語義邊界？
4. **Context Mapping（上下文映射）** - 上下文之間的關係是什麼？（上下游、共享核心、防腐層等）

**聚合識別原則：**
- 需要在同一交易中保持一致的資料 → 同一聚合
- 命令作用的範圍 → 聚合邊界的線索
- 不變條件（Invariant）的範圍 → 聚合邊界的依據

### 結構化產出總覽

以下是 Event Storming 完成後的標準產出模板：

```markdown
# Event Storming 產出: [專案/功能名稱]

## 1. 通用語言詞彙表 (Ubiquitous Language)
| 中文術語 | 英文術語 | 定義 | 所屬上下文 | 備註 |
|---------|---------|------|-----------|------|
| [術語] | [Term] | [精確定義] | [Context] | [備註] |

## 2. 領域事件清單 (Domain Events)
| 編號 | 事件名稱 | 觸發條件 | 所屬聚合 | 是否為 Pivotal Event | 備註 |
|------|---------|---------|---------|-------------------|------|
| E001 | [EventName] | [何時觸發] | [Aggregate] | 是/否 | [備註] |

## 3. 命令清單 (Commands)
| 編號 | 命令名稱 | 觸發者(Actor) | 目標聚合 | 產生事件 | 前置條件 |
|------|---------|-------------|---------|---------|---------|
| C001 | [CommandName] | [Actor] | [Aggregate] | [Event] | [條件] |

## 4. 聚合清單 (Aggregates)
| 編號 | 聚合名稱 | 職責 | 包含命令 | 包含事件 | 不變條件 |
|------|---------|------|---------|---------|---------|
| A001 | [AggregateName] | [職責描述] | C001, C002 | E001, E002 | [Invariants] |

## 5. 策略清單 (Policies)
| 編號 | 策略名稱 | 觸發事件 | 產生命令 | 規則描述 |
|------|---------|---------|---------|---------|
| P001 | [PolicyName] | [Event] | [Command] | 「每當...就...」 |

## 6. 讀模型清單 (Read Models)
| 編號 | 讀模型名稱 | 使用者 | 用途 | 包含資料 |
|------|-----------|-------|------|---------|
| R001 | [ReadModelName] | [Actor] | [決策用途] | [資料欄位] |

## 7. 限界上下文圖 (Bounded Context Map)
| 上下文名稱 | 包含聚合 | 上游依賴 | 下游服務 | 整合模式 |
|-----------|---------|---------|---------|---------|
| [ContextName] | A001, A002 | [依賴的上下文] | [服務的上下文] | [模式] |

## 8. 痛點與待釐清事項 (Hot Spots)
| 編號 | 描述 | 優先級 | 建議行動 |
|------|------|-------|---------|
| H001 | [問題描述] | 高/中/低 | [建議] |
```

### 引導原則

1. **事件優先**: 先識別「已經發生了什麼」，再追溯「是什麼觸發的」
2. **過去式命名**: 領域事件一律使用過去式（OrderPlaced, PaymentReceived）
3. **避免技術語言**: 使用業務語言而非技術術語（「訂單已成立」而非「INSERT INTO orders」）
4. **擁抱混亂**: Big Picture 階段允許重複和混亂，後續再整理
5. **視覺化優先**: 盡量用便利貼/圖形呈現，降低溝通成本
6. **迭代精煉**: 三個階段可反覆進行，逐步深入細節

---

## Phase 2: SA 領域分析

### 核心職責

- 接收 Event Storming 產出,進行領域分析
- 定義限界上下文 (Bounded Context) 邊界與職責
- 建立 Use Case → 聚合 (Aggregate) 映射
- 撰寫 Given/When/Then 驗收標準 (Acceptance Criteria)
- 維護通用語言 (Ubiquitous Language) 詞彙表
- 產出可交付給 SD (系統設計師) 的結構化分析文件

### 分析流程

#### 步驟一：確認輸入

向使用者確認可用的輸入：
1. **Event Storming 產出**（優先）: 領域事件、命令、聚合、策略、讀模型
2. **業務需求描述**: 口頭或文件形式的業務需求
3. **既有系統**: 是否有需要整合或遷移的現有系統

#### 步驟二：限界上下文定義

**分析原則**:
- 通用語言邊界：相同術語在不同上下文中意義不同 → 應拆分
- 業務能力邊界：不同業務能力 → 不同上下文
- 團隊邊界：不同團隊負責 → 不同上下文
- 資料一致性邊界：需要強一致的 → 同一上下文

**產出格式**:
```markdown
## 限界上下文定義

### 上下文: [名稱] Context

**職責**: [一句話描述這個上下文的業務職責]

**核心聚合**:
- [聚合 1]: [職責]
- [聚合 2]: [職責]

**通用語言術語**:
| 術語 | 定義 | 備註 |
|------|------|------|

**對外提供**:
- 事件: [發布的領域事件]
- 查詢: [提供的查詢介面]

**依賴**:
- 消費事件: [訂閱的領域事件]
- 呼叫介面: [依賴的外部介面]
```

#### 步驟三：Use Case → 聚合映射

**產出格式**:
```markdown
## Use Case 清單

### UC-001: [Use Case 名稱]

**Actor**: [使用者角色]
**觸發條件**: [什麼情境下觸發]
**所屬上下文**: [Bounded Context 名稱]
**涉及聚合**: [聚合名稱]

**主要流程**:
1. [步驟 1]
2. [步驟 2]
3. [步驟 3]

**替代流程**:
- [條件]: [替代步驟]

**命令→聚合→事件映射**:
| 步驟 | 命令 (Command) | 目標聚合 | 產生事件 (Event) |
|------|---------------|---------|-----------------|

**業務規則**:
- BR-001: [規則描述]
- BR-002: [規則描述]
```

#### 步驟四：驗收標準 (Given/When/Then)

為每個 Use Case 撰寫結構化驗收標準：

```markdown
## 驗收標準

### UC-001: [Use Case 名稱]

#### AC-001: [場景名稱]（Happy Path）
```gherkin
Given [前置條件]
  And [額外條件]
When [使用者執行的動作]
Then [預期結果]
  And [額外預期結果]
```

#### AC-002: [場景名稱]（替代路徑）
```gherkin
Given [前置條件]
When [使用者執行的動作]
Then [預期的替代結果]
```

#### AC-003: [場景名稱]（錯誤路徑）
```gherkin
Given [前置條件]
When [使用者嘗試不合法的動作]
Then [預期的錯誤處理]
```
```

#### 步驟五：通用語言詞彙表

```markdown
## 通用語言詞彙表 (Ubiquitous Language Glossary)

| 中文術語 | 英文術語 | 定義 | 所屬上下文 | 相關聚合 | 備註 |
|---------|---------|------|-----------|---------|------|
| [術語] | [Term] | [精確定義] | [Context] | [Aggregate] | [備註] |
```

### 完整產出範本

```markdown
# 領域分析文件: [專案/功能名稱]

## 1. 分析概覽
- **分析範圍**: [描述]
- **輸入來源**: Event Storming / 業務需求 / 既有系統
- **限界上下文數量**: N 個
- **Use Case 數量**: N 個

## 2. 限界上下文定義
[各上下文的定義]

## 3. 上下文映射 (Context Map)
[上下文間的關係]

## 4. Use Case 清單與聚合映射
[各 Use Case 的詳細描述]

## 5. 驗收標準 (Given/When/Then)
[各 Use Case 的驗收標準]

## 6. 通用語言詞彙表
[術語定義表]

## 7. 非功能需求 (NFR)
- 效能: [需求]
- 安全性: [需求]
- 可用性: [需求]

## 8. 待釐清事項
- [Issue 1]
- [Issue 2]
```

### 分析原則

1. **DDD 優先**: 以領域概念驅動分析,非以功能或畫面驅動
2. **聚合為核心**: 每個 Use Case 都要對應到具體聚合的命令
3. **驗收可測試**: Given/When/Then 格式確保可轉為自動化測試
4. **通用語言一致**: 同一上下文內術語必須唯一且明確
5. **邊界明確**: 上下文邊界要明確,不同上下文可有相同名稱但不同定義的概念
6. **向下交付**: 產出需足夠具體,讓 SD 能直接據此設計聚合結構和 API

---

## Phase 3: SD 戰術設計

### 核心職責

- 聚合 (Aggregate) 內部結構設計：Entity、Value Object、聚合根
- API 規格設計：RESTful API 對齊聚合邊界
- Spring Boot + JPA 套件結構設計
- 序列圖繪製（關鍵流程）
- 介面定義（Repository、Domain Service、Application Service）
- DTO / Command / Event 定義

### 設計流程

#### 步驟一：確認輸入

向使用者確認可用的 SA 產出：
1. **限界上下文定義**: 上下文職責和聚合清單
2. **Use Case → 聚合映射**: 命令、事件、業務規則
3. **驗收標準**: Given/When/Then
4. **通用語言詞彙表**: 術語定義

#### 步驟二：聚合結構設計

**產出格式**:
```markdown
## 聚合設計: [聚合名稱]

### 聚合根: [AggregateRoot 名稱]
```java
@Entity
@Table(name = "table_name")
public class AggregateRoot {
    @EmbeddedId
    private AggregateRootId id;

    // 內部 Entity（透過聚合根存取）
    @OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
    private List<InternalEntity> items;

    // Value Objects
    @Embedded
    private SomeValueObject valueObject;

    // 狀態
    @Enumerated(EnumType.STRING)
    private Status status;

    // === 命令方法 ===
    public void commandMethod(Param param) {
        // 不變條件檢查
        // 狀態變更
        // 產生領域事件
    }
}
```

### 內部 Entity
```java
@Entity
@Table(name = "table_name")
public class InternalEntity {
    @Id @GeneratedValue
    private Long id;
    // 欄位和方法
}
```

### Value Objects
```java
@Embeddable
public class SomeValueObject {
    // 不可變欄位
    // equals/hashCode 基於值
    // 自我驗證
}
```

### 不變條件 (Invariants)
1. [不變條件 1]: 在哪個方法中強制執行
2. [不變條件 2]: 在哪個方法中強制執行

### 領域事件
- `EventName`: 在 [方法] 中產生, 攜帶 [資料]
```

#### 步驟三：API 規格設計

**設計原則**: API 對齊聚合邊界
- 一個聚合根 = 一組 RESTful 端點
- Command = POST/PUT/PATCH/DELETE
- Query = GET
- 聚合內部 Entity 透過巢狀路徑存取

**產出格式**:
```markdown
## API 規格: [聚合名稱]

### 基礎路徑: `/api/v1/[aggregate-roots]`

#### POST /api/v1/[aggregate-roots]
**對應命令**: Create[AggregateRoot]Command
**Request Body**:
```json
{
  "field1": "value",
  "field2": "value"
}
```
**Response 201**:
```json
{
  "id": "uuid",
  "field1": "value",
  "status": "CREATED"
}
```
**驗證規則**:
- field1: 必填, 最大長度 100
- field2: 必填, 正數

**錯誤回應**:
- 400: 驗證失敗
- 409: 業務規則衝突

---

#### PATCH /api/v1/[aggregate-roots]/{id}/[action]
**對應命令**: [Action]Command
...

#### GET /api/v1/[aggregate-roots]/{id}
**對應查詢**: 聚合根詳情
...

#### GET /api/v1/[aggregate-roots]
**對應查詢**: 聚合根列表（分頁）
**Query Parameters**: page, size, sort, [filter fields]
...
```

#### 步驟四：套件結構設計

**Spring Boot + JPA + DDD 標準套件佈局**:

```
src/main/java/com/example/[project]/
├── [context]/                          # 限界上下文
│   ├── domain/                         # 領域層
│   │   ├── model/                      # 聚合、Entity、VO
│   │   │   ├── [AggregateRoot].java
│   │   │   ├── [InternalEntity].java
│   │   │   ├── [ValueObject].java
│   │   │   └── [AggregateRootId].java
│   │   ├── event/                      # 領域事件
│   │   │   └── [DomainEvent].java
│   │   ├── repository/                 # 倉儲介面
│   │   │   └── [AggregateRoot]Repository.java
│   │   └── service/                    # 領域服務（跨聚合邏輯）
│   │       └── [DomainService].java
│   │
│   ├── application/                    # 應用層
│   │   ├── command/                    # 命令 DTO
│   │   │   └── [Command]Command.java
│   │   ├── query/                      # 查詢 DTO
│   │   │   └── [Query]Query.java
│   │   ├── dto/                        # 回應 DTO
│   │   │   └── [Response]Response.java
│   │   ├── service/                    # 應用服務
│   │   │   └── [AggregateRoot]ApplicationService.java
│   │   └── eventhandler/              # 事件處理器
│   │       └── [Event]Handler.java
│   │
│   ├── infrastructure/                 # 基礎設施層
│   │   ├── persistence/                # JPA 實作
│   │   │   ├── Jpa[AggregateRoot]Repository.java
│   │   │   └── SpringData[AggregateRoot]Repository.java
│   │   └── adapter/                    # 外部系統適配器
│   │       └── [ExternalSystem]Adapter.java
│   │
│   └── interfaces/                     # 介面層（展示層）
│       ├── rest/                       # REST Controller
│       │   └── [AggregateRoot]Controller.java
│       └── dto/                        # Request/Response 轉換
│           ├── [Request]Request.java
│           └── [Assembler].java
│
└── shared/                             # 共享核心
    ├── domain/                         # 共用 Value Object
    │   ├── Money.java
    │   └── AuditInfo.java
    └── infrastructure/                 # 共用基礎設施
        └── exception/
            └── GlobalExceptionHandler.java
```

#### 步驟五：介面定義

**產出格式**:
```markdown
## 介面定義

### Repository 介面
```java
public interface OrderRepository {
    Order findById(OrderId id);
    Optional<Order> findByIdOptional(OrderId id);
    void save(Order order);
    void delete(Order order);
    Page<Order> findAll(OrderSearchCriteria criteria, Pageable pageable);
}
```

### Application Service 介面
```java
@Service
public class OrderApplicationService {
    @Transactional
    public OrderId createOrder(CreateOrderCommand command);

    @Transactional
    public void cancelOrder(CancelOrderCommand command);

    @Transactional(readOnly = true)
    public OrderDetailResponse findById(OrderId id);

    @Transactional(readOnly = true)
    public Page<OrderSummaryResponse> search(OrderSearchQuery query, Pageable pageable);
}
```

### Domain Service 介面（若需要）
```java
public interface OrderPricingService {
    Money calculateTotal(Order order, DiscountPolicy policy);
}
```
```

#### 步驟六：序列圖

為關鍵流程繪製序列圖（Mermaid 格式）：

```markdown
## 序列圖: [流程名稱]

```mermaid
sequenceDiagram
    participant C as Client
    participant Ctrl as Controller
    participant App as ApplicationService
    participant Agg as AggregateRoot
    participant Repo as Repository
    participant DB as Database

    C->>Ctrl: POST /api/v1/orders
    Ctrl->>App: createOrder(command)
    App->>Agg: new Order(command)
    Agg->>Agg: validate invariants
    App->>Repo: save(order)
    Repo->>DB: INSERT
    App-->>Ctrl: OrderId
    Ctrl-->>C: 201 Created
```
```

### 完整產出範本

```markdown
# 系統設計文件: [功能/上下文名稱]

## 1. 設計概覽
- **限界上下文**: [Context 名稱]
- **涉及聚合**: [聚合列表]
- **API 端點數**: N 個
- **技術棧**: Spring Boot 3.x + JPA + PostgreSQL

## 2. 聚合結構設計
[各聚合的內部結構]

## 3. API 規格
[RESTful API 規格]

## 4. 套件結構
[套件佈局]

## 5. 介面定義
[Repository、Service 介面]

## 6. 序列圖
[關鍵流程序列圖]

## 7. DTO 定義
[Command、Query、Response DTO]

## 8. 錯誤處理
[錯誤碼定義和例外處理策略]

## 9. 設計決策記錄 (ADR)
[重要的設計決策和理由]
```

### 設計原則

1. **API 對齊聚合**: 每個 API 端點對應一個聚合命令或查詢
2. **領域邏輯內聚**: 業務邏輯在 Domain Layer,Application Layer 只做協調
3. **依賴反轉**: Domain Layer 不依賴任何框架,Infrastructure Layer 實作介面
4. **DTO 隔離**: Controller 層使用 Request/Response DTO,不暴露 Domain Entity
5. **Spring Boot 慣例**: 遵循 Spring Boot 最佳實踐（`@Transactional`、`@Valid`、`@ControllerAdvice`）
6. **JPA 對齊 DDD**: JPA Entity 即 Domain Entity,善用 `@Embedded`、`@Enumerated`、Cascade

---

## Phase 4: 實作規劃

### 核心職責

- 接收 SD 產出（聚合設計、API 規格、套件結構、介面定義）
- 拆解為最小可執行任務單元
- 按 TDD 先測試後實作排序
- 為每個檔案產出逐檔實作規格
- 標示任務間的依賴關係

### 輸入要求

開始規劃前，向使用者確認以下資訊：
1. **SD 設計產出**: 聚合結構、API 規格、套件佈局、介面定義
2. **技術棧**: 後端框架、ORM、前端框架（預設: Spring Boot + JPA + PostgreSQL + Vue/Quasar）
3. **專案結構**: 現有專案的套件結構（若為新專案則依 SD 產出建立）
4. **範圍**: 本次迭代要實作哪些 Use Case / 聚合

### 任務拆解策略

#### 拆解原則

1. **由內而外**: Domain Layer → Application Layer → Infrastructure Layer → Presentation Layer
2. **測試先行**: 每個實作任務前，先列出對應的測試任務
3. **最小單元**: 每個任務對應一個檔案或一個小型變更
4. **依賴排序**: 被依賴的先做，依賴別人的後做

> **可選整合** — 若已安裝 superpowers 插件，可搭配 `superpowers:test-driven-development` 使用，以其 TDD 紅-綠-重構循環來執行每個任務。

#### 拆解順序（每個聚合）

```
Phase 1: Domain Layer（領域層）
  1.1 Value Object（值對象）
  1.2 Domain Event（領域事件）
  1.3 Entity（實體）
  1.4 Aggregate Root（聚合根）— 包含不變條件
  1.5 Repository Interface（倉儲介面）
  1.6 Domain Service（領域服務）— 若有

Phase 2: Application Layer（應用層）
  2.1 Command / DTO 定義
  2.2 Application Service（應用服務）
  2.3 Event Handler（事件處理器）— 若有

Phase 3: Infrastructure Layer（基礎設施層）
  3.1 JPA Entity Mapping（若 Domain Entity ≠ JPA Entity）
  3.2 Repository Implementation（倉儲實作）
  3.3 DB Migration Script (Flyway)

Phase 4: Presentation Layer（展示層）
  4.1 REST Controller
  4.2 Request/Response DTO
  4.3 Exception Handler / Error Mapping

Phase 5: Frontend（前端）
  5.1 API Client / Composable
  5.2 Store (Pinia)
  5.3 Component
  5.4 Page / Route
```

### 逐檔實作規格格式

每個任務的規格如下：

```markdown
### Task [編號]: [任務名稱]

**檔案**: `src/main/java/com/example/[path]/[FileName].java`
**類型**: [Test | Implementation | Migration | Config]
**依賴**: Task [N], Task [M]
**對應測試**: Task [N] (若為 Implementation 類型)

#### 目的
[一句話說明這個檔案的職責]

#### 規格
- [具體實作要點 1]
- [具體實作要點 2]
- [具體實作要點 3]

#### 介面定義
```java
// 類別簽章、方法簽章（不含實作）
public class ClassName {
    public ReturnType methodName(ParamType param);
}
```

#### 測試案例 (若為 Test 類型)
- [ ] [測試情境 1]: Given [前置條件], When [動作], Then [預期結果]
- [ ] [測試情境 2]: Given [前置條件], When [動作], Then [預期結果]

#### 驗收標準
- [ ] [檢查項目 1]
- [ ] [檢查項目 2]
```

### 產出範例

```markdown
# 實作計畫: [功能名稱]

## 概覽
- **涉及聚合**: [聚合名稱列表]
- **總任務數**: N 個（測試 X 個 + 實作 Y 個 + 遷移 Z 個）
- **預估檔案數**: N 個

## 依賴圖
Task 1 (VO) → Task 3 (Entity)
Task 2 (Event) → Task 3 (Entity)
Task 3 (Entity) → Task 5 (Aggregate Root)
...

## 任務清單

### Phase 1: Domain Layer

#### Task 1: [Test] OrderId 值對象測試
**檔案**: `src/test/java/.../domain/OrderIdTest.java`
**類型**: Test
**依賴**: 無
...

#### Task 2: [Impl] OrderId 值對象
**檔案**: `src/main/java/.../domain/OrderId.java`
**類型**: Implementation
**對應測試**: Task 1
...
```

### 工作流程

1. **接收 SD 產出** → 確認聚合結構、API 規格、套件佈局
2. **識別實作範圍** → 確認本次迭代的聚合和 Use Case
3. **拆解任務** → 按 Phase 1-5 順序拆解
4. **標示依賴** → 建立任務間的依賴關係
5. **產出逐檔規格** → 每個任務包含完整規格
6. **確認與調整** → 與使用者確認後微調

### 注意事項

- 每個 Implementation 任務必須有對應的 Test 任務在前
- Migration Script 任務在所有 Domain 任務之後、Repository 實作之前
- 前端任務在所有後端 API 任務完成之後
- 若發現 SD 設計有遺漏，主動提出並建議補充

> **可選整合** — 若已安裝 superpowers 插件，可搭配 `superpowers:writing-plans` 使用，以其計畫格式產出更結構化的實作計畫文件。

> **可選整合** — 若已安裝 superpowers 插件，進入實作階段時可搭配 `superpowers:using-git-worktrees` 使用，在獨立的 worktree 中隔離開發。

> **可選整合** — 若已安裝 superpowers 插件，實作完成後可搭配 `superpowers:verification-before-completion` 使用，確保所有產出的完整性。
