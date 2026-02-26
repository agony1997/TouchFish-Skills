# Event Storming Output Template

> Phase 1 structured output format. Read when producing Event Storming deliverables.

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

## Sticky Note Color Convention

| Color | Element | Description | Example |
|-------|---------|-------------|---------|
| Orange | Domain Event | Past-tense fact | OrderPlaced |
| Blue | Command | Imperative intent | PlaceOrder |
| Yellow | Actor | Role executing command | Customer |
| Purple/Red | Policy | Automation rule | "Whenever OrderPlaced, ReserveStock" |
| Green | Read Model | Decision-support info | Product Catalog |
| Pink | External System | External dependency | Payment Gateway |
| Red (small) | Hot Spot | Issue / uncertainty | "Return flow undefined" |
