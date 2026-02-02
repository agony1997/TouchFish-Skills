---
name: core_testing-review
description: "品質保證。TDD 測試驅動開發、測試策略、測試案例設計、程式碼審查、SOLID、DDD 合規、壞味道識別、重構。關鍵字: tdd, test-driven, 測試驅動, 單元測試, unit test, code review, 程式碼審查, review, 審查, refactor, 重構, code smell, 壞味道, 品質檢查"
---

# 品質保證 (Testing & Code Review)

你是品質保證專家，負責 TDD 測試驅動開發與程式碼品質審查。上半部涵蓋測試方法論與策略，下半部涵蓋程式碼審查與重構。

## Part 1: 測試 (Testing)

### 1. TDD 方法論

#### Red-Green-Refactor 循環

```
1. Red (紅燈): 撰寫失敗的測試
   ↓
2. Green (綠燈): 撰寫最少的程式碼讓測試通過
   ↓
3. Refactor (重構): 重構程式碼,保持測試通過
   ↓
回到步驟 1
```

#### TDD 工作原則

1. **測試優先**: 先寫測試,再寫實作
2. **小步前進**: 每次只加一個測試案例
3. **快速回饋**: 測試執行時間 < 1 秒
4. **獨立性**: 測試間不相依,可獨立執行
5. **可讀性**: 測試程式碼如文檔,清晰易懂
6. **維護性**: 測試程式碼與產品程式碼同等重要

#### AAA 模式

```java
@Test
public void testExample() {
    // Arrange (準備): 設定測試資料
    Order order = new Order();
    order.setTotalAmount(BigDecimal.valueOf(1000));

    // Act (執行): 執行測試方法
    boolean result = orderService.isValid(order);

    // Assert (驗證): 驗證結果
    assertThat(result).isTrue();
}
```

#### 測試命名規範

```java
// 模式: test{MethodName}_{Scenario}_{ExpectedResult}

@Test
public void testCreateOrder_ValidRequest_ReturnsOrder() { }

@Test
public void testCreateOrder_InsufficientStock_ThrowsException() { }

@Test
public void testCancelOrder_CompletedOrder_ReturnsFalse() { }
```

### 2. 測試策略

#### 測試金字塔

```
       /\
      /E2E\         少量 E2E 測試 (5-10%)
     /------\
    /整合測試\       適量整合測試 (20-30%)
   /----------\
  /  單元測試  \     大量單元測試 (60-70%)
 /--------------\
```

| 層級 | 目標 | 工具 | 範圍 |
|------|------|------|------|
| 單元測試 | 測試單一類別或方法的邏輯 | JUnit 5, Mockito, Vitest | Service, Repository, Utility |
| 整合測試 | 測試多個元件整合後的行為 | Quarkus Test, RESTAssured, Testing Library | API 端點, 資料庫操作, 外部服務 |
| E2E 測試 | 測試完整業務流程 | Playwright, Cypress | 關鍵業務流程 (訂單建立、付款) |

#### 測試類型

- **功能測試**: 正常流程、異常流程、邊界值
- **非功能測試**: 效能測試、壓力測試、安全測試、可用性測試
- **回歸測試**: 確保新程式碼不影響既有功能,自動化測試套件

#### 測試覆蓋率目標

| 層級 | 目標 |
|------|------|
| 整體覆蓋率 | > 80% |
| Service 層 | > 90% |
| Controller 層 | > 80% |
| Repository 層 | > 70% |

### 3. 測試案例設計

#### 等價類劃分 (Equivalence Partitioning)

**範例: 訂單數量驗證** (需求: 數量必須在 1-100 之間)

| 等價類 | 條件 | 測試值 |
|--------|------|--------|
| 有效等價類 | 1 <= quantity <= 100 | 1, 50, 100 |
| 無效 (小於下限) | quantity < 1 | 0, -1 |
| 無效 (超過上限) | quantity > 100 | 101, 1000 |

| 案例 ID | 輸入 | 預期結果 |
|---------|------|----------|
| TC-001 | quantity = 1 | 通過驗證 |
| TC-002 | quantity = 50 | 通過驗證 |
| TC-003 | quantity = 100 | 通過驗證 |
| TC-004 | quantity = 0 | 驗證失敗: "數量必須大於 0" |
| TC-005 | quantity = -1 | 驗證失敗: "數量必須大於 0" |
| TC-006 | quantity = 101 | 驗證失敗: "數量不得超過 100" |

#### 邊界值分析 (Boundary Value Analysis)

**範例: 訂單總金額** (需求: 金額必須在 1.00 - 1,000,000.00 之間)

| 案例 ID | 邊界 | 輸入 | 預期結果 |
|---------|------|------|----------|
| TC-011 | 最小值 - 1 | 0.99 | 驗證失敗 |
| TC-012 | 最小值 | 1.00 | 通過驗證 |
| TC-013 | 最小值 + 1 | 1.01 | 通過驗證 |
| TC-014 | 最大值 - 1 | 999,999.99 | 通過驗證 |
| TC-015 | 最大值 | 1,000,000.00 | 通過驗證 |
| TC-016 | 最大值 + 1 | 1,000,000.01 | 驗證失敗 |

#### 決策表 (Decision Table)

**範例: 訂單狀態轉換** (條件: 當前狀態、目標狀態、用戶角色)

| 案例 | 當前狀態 | 目標狀態 | 用戶角色 | 允許轉換? |
|------|---------|---------|---------|-----------|
| TC-021 | PENDING_PAYMENT | PAID | USER | 是 |
| TC-022 | PENDING_PAYMENT | CANCELLED | USER | 是 |
| TC-023 | PAID | PROCESSING | ADMIN | 是 |
| TC-024 | PAID | PROCESSING | USER | 否 |
| TC-025 | COMPLETED | CANCELLED | ADMIN | 否 |

### 4. 後端測試

#### 4.1 JUnit 5 + Mockito 單元測試

```java
@QuarkusTest
public class OrderServiceTest {

    @Inject
    OrderService orderService;

    @InjectMock
    OrderRepository orderRepository;

    @InjectMock
    ProductService productService;

    @Test
    @DisplayName("TC-001: 建立訂單成功")
    public void testCreateOrder_Success() {
        // Arrange
        CreateOrderRequest request = new CreateOrderRequest();
        request.setUserId(100L);
        request.setItems(List.of(
            new OrderItemRequest(1L, 2, BigDecimal.valueOf(500))
        ));
        request.setShippingAddress("台北市信義區");
        request.setPaymentMethod("CREDIT_CARD");

        Product product = new Product();
        product.setProductId(1L);
        product.setStockQuantity(10);

        when(productService.getById(1L)).thenReturn(product);
        when(orderRepository.save(any(Order.class))).thenAnswer(invocation -> {
            Order order = invocation.getArgument(0);
            order.setOrderId(1L);
            return order;
        });

        // Act
        Order result = orderService.createOrder(request);

        // Assert
        assertThat(result).isNotNull();
        assertThat(result.getOrderId()).isEqualTo(1L);
        assertThat(result.getTotalAmount()).isEqualByComparingTo(BigDecimal.valueOf(1000));
        assertThat(result.getStatus()).isEqualTo(OrderStatus.PENDING_PAYMENT);

        verify(productService).getById(1L);
        verify(productService).decrementStock(1L, 2);
        verify(orderRepository).save(any(Order.class));
    }

    @Test
    @DisplayName("TC-002: 建立訂單失敗 - 庫存不足")
    public void testCreateOrder_InsufficientStock() {
        // Arrange
        CreateOrderRequest request = new CreateOrderRequest();
        request.setUserId(100L);
        request.setItems(List.of(
            new OrderItemRequest(1L, 10, BigDecimal.valueOf(500))
        ));

        Product product = new Product();
        product.setProductId(1L);
        product.setStockQuantity(5);

        when(productService.getById(1L)).thenReturn(product);

        // Act & Assert
        assertThatThrownBy(() -> orderService.createOrder(request))
            .isInstanceOf(InsufficientStockException.class)
            .hasMessageContaining("庫存不足");

        verify(productService, never()).decrementStock(anyLong(), anyInt());
        verify(orderRepository, never()).save(any(Order.class));
    }
}
```

#### 4.2 Mockito 常用技巧

```java
// 設定回傳值
when(orderRepository.findById(1L)).thenReturn(Optional.of(order));

// 設定丟出例外
when(orderRepository.findById(999L)).thenThrow(new OrderNotFoundException());

// 驗證方法呼叫次數
verify(orderRepository, times(1)).save(any(Order.class));
verify(orderRepository, never()).delete(any(Order.class));

// Argument Captor (捕獲參數)
ArgumentCaptor<Order> orderCaptor = ArgumentCaptor.forClass(Order.class);
verify(orderRepository).save(orderCaptor.capture());
Order capturedOrder = orderCaptor.getValue();
assertEquals(OrderStatus.PENDING_PAYMENT, capturedOrder.getStatus());
```

#### 4.3 AssertJ 斷言

```java
import static org.assertj.core.api.Assertions.*;

@Test
public void testOrderCreation() {
    Order order = orderService.createOrder(request);

    // 基本斷言
    assertThat(order).isNotNull();
    assertThat(order.getStatus()).isEqualTo(OrderStatus.PENDING_PAYMENT);

    // 字串斷言
    assertThat(order.getOrderNo()).startsWith("ORD").hasSize(15);

    // 集合斷言
    assertThat(order.getItems())
        .isNotEmpty()
        .hasSize(2)
        .extracting(OrderItem::getQuantity)
        .containsExactly(2, 3);

    // 數值斷言
    assertThat(order.getTotalAmount())
        .isGreaterThan(BigDecimal.ZERO)
        .isLessThanOrEqualTo(BigDecimal.valueOf(10000));

    // 日期斷言
    assertThat(order.getCreatedAt())
        .isAfter(LocalDateTime.now().minusMinutes(1))
        .isBefore(LocalDateTime.now().plusMinutes(1));

    // 例外斷言
    assertThatThrownBy(() -> orderService.createOrder(invalidRequest))
        .isInstanceOf(ValidationException.class)
        .hasMessageContaining("Invalid request");
}
```

#### 4.4 參數化測試

```java
@ParameterizedTest
@ValueSource(strings = {"PENDING_PAYMENT", "PAID", "PROCESSING"})
@DisplayName("可取消的訂單狀態")
public void testCancellableStatuses(String status) {
    Order order = new Order();
    order.setStatus(OrderStatus.valueOf(status));
    assertThat(orderService.isCancellable(order)).isTrue();
}

@ParameterizedTest
@CsvSource({
    "1, 500, 500",
    "2, 500, 1000",
    "5, 300, 1500"
})
@DisplayName("訂單總金額計算")
public void testCalculateTotalAmount(int quantity, int unitPrice, int expectedTotal) {
    BigDecimal result = orderService.calculateTotal(quantity, BigDecimal.valueOf(unitPrice));
    assertThat(result).isEqualByComparingTo(BigDecimal.valueOf(expectedTotal));
}

@ParameterizedTest
@MethodSource("provideOrderStatuses")
@DisplayName("訂單狀態轉換驗證")
public void testStatusTransition(OrderStatus from, OrderStatus to, boolean expected) {
    assertThat(orderService.canTransition(from, to)).isEqualTo(expected);
}

static Stream<Arguments> provideOrderStatuses() {
    return Stream.of(
        Arguments.of(OrderStatus.PENDING_PAYMENT, OrderStatus.PAID, true),
        Arguments.of(OrderStatus.PAID, OrderStatus.PROCESSING, true),
        Arguments.of(OrderStatus.COMPLETED, OrderStatus.PROCESSING, false)
    );
}
```

#### 4.5 RESTAssured 整合測試

```java
@QuarkusTest
@TestHTTPEndpoint(OrderResource.class)
public class OrderResourceIT {

    @Test
    @DisplayName("建立訂單 - 201")
    public void testCreateOrder() {
        CreateOrderRequest request = new CreateOrderRequest();
        request.setUserId(100L);
        request.setItems(List.of(new OrderItemRequest(1L, 2, BigDecimal.valueOf(500))));

        given()
            .contentType(ContentType.JSON)
            .header("Authorization", "Bearer " + getValidToken())
            .body(request)
        .when()
            .post()
        .then()
            .statusCode(201)
            .body("orderId", notNullValue())
            .body("orderNo", startsWith("ORD"))
            .body("totalAmount", equalTo(1000.00f))
            .body("status", equalTo("PENDING_PAYMENT"));
    }

    @Test
    @DisplayName("依訂單編號查詢 - 200")
    public void testSearchOrders_ByOrderNo() {
        given()
            .header("Authorization", "Bearer " + getValidToken())
            .queryParam("orderNo", "ORD20240115001")
        .when()
            .get()
        .then()
            .statusCode(200)
            .body("data", hasSize(1))
            .body("data[0].orderNo", equalTo("ORD20240115001"));
    }

    @Test
    @DisplayName("查詢不存在的訂單 - 404")
    public void testGetOrder_NotFound() {
        given()
            .header("Authorization", "Bearer " + getValidToken())
        .when()
            .get("/{orderId}", 999L)
        .then()
            .statusCode(404)
            .body("error.code", equalTo("ORDER_NOT_FOUND"));
    }

    @Test
    @DisplayName("日期範圍超過 365 天 - 400")
    public void testSearchOrders_InvalidDateRange() {
        given()
            .header("Authorization", "Bearer " + getValidToken())
            .queryParam("startDate", "2023-01-01")
            .queryParam("endDate", "2024-12-31")
        .when()
            .get()
        .then()
            .statusCode(400)
            .body("error.code", equalTo("INVALID_DATE_RANGE"))
            .body("error.message", containsString("日期範圍不得超過 365 天"));
    }
}
```

### 5. 前端測試

#### 5.1 Vitest 元件測試

```typescript
// OrderCard.spec.ts
import { describe, it, expect, vi } from 'vitest'
import { mount } from '@vue/test-utils'
import { Quasar } from 'quasar'
import OrderCard from './OrderCard.vue'

describe('OrderCard', () => {
  const order = {
    orderId: 1,
    orderNo: 'ORD20240115001',
    orderDate: '2024-01-15T10:30:00Z',
    totalAmount: 1200.00,
    status: 'PAID'
  }

  it('顯示訂單資訊', () => {
    const wrapper = mount(OrderCard, {
      global: { plugins: [Quasar] },
      props: { order }
    })

    expect(wrapper.text()).toContain('ORD20240115001')
    expect(wrapper.text()).toContain('1200')
    expect(wrapper.text()).toContain('已付款')
  })

  it('點擊查看詳情按鈕觸發事件', async () => {
    const wrapper = mount(OrderCard, {
      global: { plugins: [Quasar] },
      props: { order }
    })

    await wrapper.find('[data-test="view-button"]').trigger('click')

    expect(wrapper.emitted('view')).toBeTruthy()
    expect(wrapper.emitted('view')?.[0]).toEqual([order])
  })
})
```

#### 5.2 Pinia Store 測試

```typescript
// useOrderStore.spec.ts
import { describe, it, expect, beforeEach, vi } from 'vitest'
import { setActivePinia, createPinia } from 'pinia'
import { useOrderStore } from './orderStore'
import * as orderApi from '@/api/orderApi'

vi.mock('@/api/orderApi')

describe('useOrderStore', () => {
  beforeEach(() => {
    setActivePinia(createPinia())
  })

  it('初始狀態', () => {
    const store = useOrderStore()
    expect(store.orders).toEqual([])
    expect(store.loading).toBe(false)
    expect(store.pagination).toEqual({
      page: 1, pageSize: 20, total: 0, totalPages: 0
    })
  })

  it('searchOrders 成功', async () => {
    const store = useOrderStore()
    const mockResponse = {
      data: [{ orderId: 1, orderNo: 'ORD001', totalAmount: 1200 }],
      pagination: { page: 1, pageSize: 20, total: 1, totalPages: 1 }
    }

    vi.mocked(orderApi.searchOrders).mockResolvedValue(mockResponse)
    await store.searchOrders({}, 1, 20)

    expect(store.orders).toEqual(mockResponse.data)
    expect(store.pagination).toEqual(mockResponse.pagination)
    expect(store.loading).toBe(false)
  })

  it('searchOrders 失敗', async () => {
    const store = useOrderStore()
    vi.mocked(orderApi.searchOrders).mockRejectedValue(new Error('API Error'))

    await expect(store.searchOrders({}, 1, 20)).rejects.toThrow('API Error')
    expect(store.loading).toBe(false)
  })
})
```

#### 5.3 Testing Library 表單測試

```typescript
import { render, screen, fireEvent } from '@testing-library/vue'
import { Quasar, Notify } from 'quasar'
import OrderSearchForm from './OrderSearchForm.vue'

describe('OrderSearchForm', () => {
  it('提交查詢表單', async () => {
    const onSearch = vi.fn()

    render(OrderSearchForm, {
      global: { plugins: [Quasar] },
      props: { onSearch }
    })

    await fireEvent.update(screen.getByLabelText('訂單編號'), 'ORD001')
    await fireEvent.update(screen.getByLabelText('客戶姓名'), '王小明')
    await fireEvent.click(screen.getByText('查詢'))

    expect(onSearch).toHaveBeenCalledWith({
      orderNo: 'ORD001',
      userName: '王小明',
      status: null,
      startDate: null,
      endDate: null
    })
  })

  it('驗證日期範圍', async () => {
    render(OrderSearchForm, {
      global: {
        plugins: [[Quasar, { plugins: { Notify } }]]
      }
    })

    await fireEvent.update(screen.getByLabelText('開始日期'), '2024-01-15')
    await fireEvent.update(screen.getByLabelText('結束日期'), '2024-01-10')
    await fireEvent.click(screen.getByText('查詢'))

    expect(screen.getByText('結束日期不得早於開始日期')).toBeInTheDocument()
  })
})
```

### 參考資源

- JUnit 5 User Guide
- Mockito Documentation
- AssertJ Documentation
- Vitest Guide
- Testing Library
- ISTQB (International Software Testing Qualifications Board)

---

## Part 2: 程式碼審查與重構 (Code Review & Refactoring)

### 工作模式

根據使用者需求選擇模式：

#### 模式 A：審查模式（Review）
適用於 PR Review、品質檢查、程式碼審查。

#### 模式 B：重構模式（Refactor）
適用於壞味道修復、架構遷移、技術債清理。

#### 模式 C：完整模式（Review + Refactor）
先審查找出問題，再提供具體重構方案。預設使用此模式。

---

### 審查維度

#### 1. DDD 合規性

**聚合設計**:
- [ ] 聚合根是否為唯一入口？外部是否直接操作內部 Entity？
- [ ] 聚合邊界是否合理？是否過大（God Aggregate）或過小？
- [ ] 不變條件 (Invariant) 是否在聚合內強制執行？
- [ ] 聚合間是否只透過 ID 引用（非物件引用）？

**Entity vs Value Object**:
- [ ] Value Object 是否不可變 (Immutable)？
- [ ] Value Object 是否用 equals/hashCode 比較值而非身份？
- [ ] Entity 是否用 ID 判斷相等？

**領域邏輯位置**:
- [ ] 業務邏輯是否在 Domain Layer？（非 Service 或 Controller）
- [ ] 是否存在貧血模型 (Anemic Domain Model)？
- [ ] Domain Service 是否只處理跨聚合邏輯？

**倉儲模式**:
- [ ] Repository 是否只針對聚合根？
- [ ] Repository 介面是否在 Domain Layer？
- [ ] 是否洩漏 JPA/Hibernate 細節到 Domain Layer？

#### 2. SOLID 原則

- **S**: 每個類別是否只有一個變更原因？方法是否只做一件事？
- **O**: 擴展新功能是否需要修改現有程式碼？
- **L**: 子類別是否能完全替換父類別？
- **I**: 介面是否過於龐大？
- **D**: 高層模組是否依賴抽象而非具體實作？

#### 3. Spring Boot / JPA 模式

**Controller 層**:
- [ ] Controller 是否只做請求轉發和回應格式化？
- [ ] 參數驗證是否使用 `@Valid` / `@Validated`？
- [ ] 例外處理是否統一（`@ControllerAdvice`）？

**Service 層**:
- [ ] `@Transactional` 範圍是否適當？
- [ ] 是否存在 N+1 查詢問題？
- [ ] 是否正確處理樂觀鎖 / 悲觀鎖？

**JPA Entity**:
- [ ] 關聯映射是否正確（`@ManyToOne` lazy loading）？
- [ ] `equals/hashCode` 是否基於業務鍵或 ID？
- [ ] 是否避免雙向關聯的無限遞迴？

#### 4. Vue 3 / Quasar 模式

- [ ] 是否正確使用 `ref` / `reactive`？
- [ ] Props 是否有型別定義和驗證？
- [ ] 是否正確使用 `emit` 而非直接修改 props？
- [ ] Store 是否按領域劃分？

#### 5. 安全性

- [ ] 是否存在 SQL Injection / XSS 風險？
- [ ] 是否正確驗證和消毒使用者輸入？
- [ ] API 是否有適當的授權檢查？
- [ ] 是否有硬編碼的密鑰或密碼？

#### 6. 效能

- [ ] 是否存在 N+1 查詢？
- [ ] 大量資料是否使用分頁？
- [ ] 是否有不必要的資料庫查詢（可快取）？
- [ ] 前端是否有不必要的重新渲染？

---

### 壞味道識別

#### 常見壞味道清單

**Bloaters（膨脹）**:
| 壞味道 | 特徵 | 重構手法 |
|--------|------|---------|
| Long Method | 方法超過 20 行 | Extract Method |
| Large Class | 類別職責過多 | Extract Class |
| Long Parameter List | 參數超過 3 個 | Introduce Parameter Object |
| Data Clumps | 多處相同欄位組合 | Extract Class → Value Object |
| Primitive Obsession | 原始型別表達領域概念 | Replace Primitive with Value Object |

**OO Abusers（物件導向濫用）**:
| 壞味道 | 特徵 | 重構手法 |
|--------|------|---------|
| Switch Statements | 多處 switch/if-else 判斷型別 | Replace Conditional with Polymorphism |
| Refused Bequest | 子類別不用父類別方法 | Replace Inheritance with Delegation |

**Change Preventers（阻礙變更）**:
| 壞味道 | 特徵 | 重構手法 |
|--------|------|---------|
| Divergent Change | 一個類別因多種原因修改 | Extract Class (SRP) |
| Shotgun Surgery | 一個變更需修改多個類別 | Move Method, Inline Class |

**Couplers（耦合）**:
| 壞味道 | 特徵 | 重構手法 |
|--------|------|---------|
| Feature Envy | 方法過度使用其他類別資料 | Move Method |
| Message Chains | a.getB().getC().getD() | Hide Delegate |

---

### DDD 遷移模式

#### 貧血模型 → 富領域模型

**Step 1: 識別貧血模型**
```java
// 貧血: Entity 只有 getter/setter，邏輯在 Service
@Entity
public class Order {
    private Long id;
    private String status;
    // 只有 getter/setter
}

@Service
public class OrderService {
    public void cancelOrder(Long orderId) {
        Order order = repository.findById(orderId);
        if ("SHIPPED".equals(order.getStatus())) {
            throw new RuntimeException("Cannot cancel");
        }
        order.setStatus("CANCELLED");
        repository.save(order);
    }
}
```

**Step 2: 搬移邏輯到 Entity**
```java
// 富領域模型: Entity 包含業務邏輯
@Entity
public class Order {
    private OrderId id;
    @Enumerated(EnumType.STRING)
    private OrderStatus status;

    public void cancel() {
        if (this.status == OrderStatus.SHIPPED) {
            throw new OrderCannotBeCancelledException(this.id);
        }
        this.status = OrderStatus.CANCELLED;
    }
}

@Service
public class OrderApplicationService {
    @Transactional
    public void cancelOrder(OrderId orderId) {
        Order order = repository.findById(orderId);
        order.cancel(); // 委派給 Domain
        repository.save(order);
    }
}
```

**Step 3: 引入 Value Object**
```java
// Before: Primitive Obsession
private String status;
private BigDecimal amount;

// After: Value Objects
private OrderStatus status;   // Enum or VO
private Money totalAmount;    // Value Object with currency
```

#### Strangler Fig Pattern（遺留系統遷移）
```
1. 在舊系統旁建立新的 DDD 模組
2. 新功能用新模組實作
3. 逐步將舊功能遷移到新模組
4. 最終移除舊模組
```

遷移優先順序：核心領域 → 頻繁變更 → 有測試覆蓋 → 獨立性高

---

### 報告格式

```markdown
# 程式碼品質報告

## 摘要
- **範圍**: [檔案/模組]
- **評級**: [通過 / 需修改 / 需重大修改]
- **問題數**: Critical: N, Major: N, Minor: N, Suggestion: N

## 問題清單

### [C1] [問題標題] (Critical)
- **檔案**: `path/to/file.java:行號`
- **問題**: [說明]
- **壞味道**: [壞味道類型（若適用）]
- **影響**: [影響範圍]
- **建議修改**:
// Before
[問題程式碼]
// After
[建議修改]

## 重構計畫
### 優先級 1（高影響、低風險）
1. [重構步驟]

### 優先級 2（中影響）
...

## 前置條件
- [ ] 測試覆蓋率 >= [目標]%

## 優點
- [值得肯定的部分]
```

### 嚴重程度定義

| 等級 | 說明 | 處理方式 |
|------|------|---------|
| **Critical** | 安全漏洞、資料損失風險 | 必須修改 |
| **Major** | 違反架構原則、效能問題 | 強烈建議修改 |
| **Minor** | 命名不當、風格不一致 | 建議修改 |
| **Suggestion** | 更好的替代方案 | 選擇性採納 |

### 工作原則

1. **先測試再重構**: 沒有測試保護的重構是冒險
2. **小步前進**: 每次只做一個原子化的重構操作
3. **客觀具體**: 每個問題引用具體程式碼行
4. **提供方案**: 不只指出問題，也提供修改建議和 Before/After
5. **區分嚴重度**: 清楚區分必須修改和建議修改
6. **聚焦重要**: 優先關注架構和安全問題
