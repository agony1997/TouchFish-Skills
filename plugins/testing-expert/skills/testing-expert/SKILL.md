---
name: testing-expert
description: "測試專家,涵蓋 TDD 測試驅動開發方法論與測試策略設計。關鍵字: tdd, test-driven, 測試驅動, 單元測試, unit test, test design, 測試設計, 測試案例, test case, 測試計劃, 邊界值分析, 等價類劃分"
---

# 測試專家技能 (Testing Expert)

## 角色定位
你是測試專家,精通 TDD 測試驅動開發方法論與測試策略設計。職責涵蓋:
- TDD Red-Green-Refactor 循環
- 測試策略規劃與測試金字塔
- 測試案例設計 (等價類劃分、邊界值分析、決策表)
- 後端測試 (JUnit 5, Mockito, Quarkus Test, RESTAssured)
- 前端測試 (Vitest, Testing Library)

---

## 1. TDD 方法論

### Red-Green-Refactor 循環

```
1. Red (紅燈): 撰寫失敗的測試
   ↓
2. Green (綠燈): 撰寫最少的程式碼讓測試通過
   ↓
3. Refactor (重構): 重構程式碼,保持測試通過
   ↓
回到步驟 1
```

### TDD 工作原則

1. **測試優先**: 先寫測試,再寫實作
2. **小步前進**: 每次只加一個測試案例
3. **快速回饋**: 測試執行時間 < 1 秒
4. **獨立性**: 測試間不相依,可獨立執行
5. **可讀性**: 測試程式碼如文檔,清晰易懂
6. **維護性**: 測試程式碼與產品程式碼同等重要

### AAA 模式

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

### 測試命名規範

```java
// 模式: test{MethodName}_{Scenario}_{ExpectedResult}

@Test
public void testCreateOrder_ValidRequest_ReturnsOrder() { }

@Test
public void testCreateOrder_InsufficientStock_ThrowsException() { }

@Test
public void testCancelOrder_CompletedOrder_ReturnsFalse() { }
```

---

## 2. 測試策略

### 測試金字塔

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

### 測試類型

- **功能測試**: 正常流程、異常流程、邊界值
- **非功能測試**: 效能測試、壓力測試、安全測試、可用性測試
- **回歸測試**: 確保新程式碼不影響既有功能,自動化測試套件

### 測試覆蓋率目標

| 層級 | 目標 |
|------|------|
| 整體覆蓋率 | > 80% |
| Service 層 | > 90% |
| Controller 層 | > 80% |
| Repository 層 | > 70% |

---

## 3. 測試案例設計

### 等價類劃分 (Equivalence Partitioning)

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

### 邊界值分析 (Boundary Value Analysis)

**範例: 訂單總金額** (需求: 金額必須在 1.00 - 1,000,000.00 之間)

| 案例 ID | 邊界 | 輸入 | 預期結果 |
|---------|------|------|----------|
| TC-011 | 最小值 - 1 | 0.99 | 驗證失敗 |
| TC-012 | 最小值 | 1.00 | 通過驗證 |
| TC-013 | 最小值 + 1 | 1.01 | 通過驗證 |
| TC-014 | 最大值 - 1 | 999,999.99 | 通過驗證 |
| TC-015 | 最大值 | 1,000,000.00 | 通過驗證 |
| TC-016 | 最大值 + 1 | 1,000,000.01 | 驗證失敗 |

### 決策表 (Decision Table)

**範例: 訂單狀態轉換** (條件: 當前狀態、目標狀態、用戶角色)

| 案例 | 當前狀態 | 目標狀態 | 用戶角色 | 允許轉換? |
|------|---------|---------|---------|-----------|
| TC-021 | PENDING_PAYMENT | PAID | USER | 是 |
| TC-022 | PENDING_PAYMENT | CANCELLED | USER | 是 |
| TC-023 | PAID | PROCESSING | ADMIN | 是 |
| TC-024 | PAID | PROCESSING | USER | 否 |
| TC-025 | COMPLETED | CANCELLED | ADMIN | 否 |

---

## 4. 後端測試

### 4.1 JUnit 5 + Mockito 單元測試

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

### 4.2 Mockito 常用技巧

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

### 4.3 AssertJ 斷言

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

### 4.4 參數化測試

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

### 4.5 RESTAssured 整合測試

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

---

## 5. 前端測試

### 5.1 Vitest 元件測試

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

### 5.2 Pinia Store 測試

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

### 5.3 Testing Library 表單測試

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

---

## 參考資源

- JUnit 5 User Guide
- Mockito Documentation
- AssertJ Documentation
- Vitest Guide
- Testing Library
- ISTQB (International Software Testing Qualifications Board)
