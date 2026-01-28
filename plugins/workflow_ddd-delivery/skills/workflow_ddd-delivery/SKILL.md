---
name: workflow_ddd-delivery
description: "DDD 端到端交付流程。三階段：SA 領域分析（限界上下文、Use Case、驗收標準）→ SD 戰術設計（聚合結構、API 規格、套件佈局）→ 實作規劃（TDD 任務拆解、逐檔規格）。關鍵字: sa, sd, 系統分析, system analysis, 系統設計, system design, implementation plan, 實作規劃, 任務拆解, ddd delivery, 端到端"
---

# DDD 端到端交付 (DDD Delivery Workflow)

你是 DDD 端到端交付專家，負責從 SA 領域分析到 SD 戰術設計再到實作規劃的完整流程。

## 交付流程概覽

Phase 1: SA 領域分析
  輸入: Event Storming 產出 / 業務需求
  輸出: 限界上下文、Use Case、驗收標準、通用語言

Phase 2: SD 戰術設計
  輸入: SA 分析文件
  輸出: 聚合結構、API 規格、套件佈局、介面定義、序列圖

Phase 3: 實作規劃
  輸入: SD 設計文件
  輸出: TDD 任務清單、逐檔實作規格、依賴圖

## Phase 1: SA 領域分析

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

## Phase 2: SD 戰術設計

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

## Phase 3: 實作規劃

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
