# ddd-core 技能使用指南

> 版本：1.1.0 | 最後更新：2026-02-26

## 這是什麼？

ddd-core 是一個 DDD 端到端交付技能，涵蓋從理論基礎到實作規劃的完整流程。它引導 AI 助手以 Domain-Driven Design 方法論，系統化地完成領域探索、分析、設計與實作規劃。

**涵蓋範圍：**
- DDD 理論基礎（戰略設計、戰術設計、CQRS、Event Sourcing）
- Event Storming 領域探索
- SA（系統分析）領域分析
- SD（系統設計）戰術設計
- 實作規劃（TDD 任務清單）

## 使用流程

```
┌─────────────────────────────────────────────────────┐
│                  ddd-core 交付流程                     │
├─────────────────────────────────────────────────────┤
│                                                     │
│  Phase 0: DDD 理論基礎（按需載入）                     │
│    │  使用者不熟 DDD 時才載入                          │
│    ▼                                                │
│  Phase 1: Event Storming 領域探索                     │
│    │  Big Picture → Process Modeling → Software Design│
│    │  產出：事件、命令、聚合、策略、讀模型               │
│    ▼                                                │
│  Phase 2: SA 領域分析                                 │
│    │  限界上下文 → Use Case 映射 → 驗收標準             │
│    │  產出：分析文件（上下文定義、UC、Given/When/Then）  │
│    ▼                                                │
│  Phase 3: SD 戰術設計                                 │
│    │  聚合結構 → API 規格 → 套件佈局 → 介面定義         │
│    │  產出：設計文件（聚合、API、序列圖）                │
│    ▼                                                │
│  Phase 4: 實作規劃                                    │
│       任務拆解 → TDD 排序 → 逐檔規格 → 依賴圖          │
│       產出：實作計畫（任務清單、檔案規格）                │
│                                                     │
└─────────────────────────────────────────────────────┘
```

## 各階段詳解

### Phase 0: DDD 理論基礎（按需載入）

**什麼時候需要？**
- 你或團隊成員不熟悉 DDD 概念
- 需要了解 Bounded Context、Aggregate、Entity、Value Object 等基本概念
- 想要看 Java 程式碼範例

**涵蓋內容：**
- **Strategic Design（戰略設計）**
  - Bounded Context（限界上下文）
  - Context Mapping（上下文映射）：Shared Kernel、Anti-Corruption Layer
- **Tactical Design（戰術設計）**
  - Entity vs Value Object
  - Aggregate and Aggregate Root
  - Domain Service
  - Domain Events
  - Repository Pattern
  - CQRS Pattern
- **Best Practices**
  - Ubiquitous Language
  - 保持聚合精簡
  - 保護不變條件
  - 建模真實業務規則
- **Quick Reference** — Building Blocks 速查表

**使用方式：** 直接告訴 AI「我想了解 DDD 的基本概念」或「幫我解釋 Aggregate 是什麼」。

### Phase 1: Event Storming 領域探索

**輸入：** 業務需求描述或領域專家知識

**AI 會引導你完成三個階段：**

1. **Big Picture（全貌探索）**
   - 發散探索所有領域事件
   - 時間線排序
   - 識別 Pivotal Events（關鍵事件）
   - 標記 Hot Spots（熱點問題）

2. **Process Modeling（流程建模）**
   - 識別 Commands（命令）、Actors（參與者）
   - 識別 Policies（策略規則）：「每當...就...」
   - 識別 Read Models（讀模型）、External Systems（外部系統）

3. **Software Design（軟體設計）**
   - 識別 Aggregates（聚合）及其邊界
   - 定義 Bounded Contexts（限界上下文）
   - 建立 Context Mapping（上下文映射）

**產出：** 結構化 Event Storming 文件，包含詞彙表、事件清單、命令清單、聚合清單、策略清單、讀模型清單、上下文圖、Hot Spots。

**便利貼顏色約定：**
| 顏色 | 元素 | 說明 |
|------|------|------|
| 橘色 | Domain Event | 已發生的事實（過去式） |
| 藍色 | Command | 意圖/動作（祈使句） |
| 黃色 | Actor | 執行命令的角色 |
| 紫色/紅色 | Policy | 自動化規則 |
| 綠色 | Read Model | 決策所需的資訊 |
| 粉紅色 | External System | 外部系統 |
| 紅色（小） | Hot Spot | 問題/爭議/待確認 |

### Phase 2: SA 領域分析

**輸入：** Phase 1 的 Event Storming 產出，或直接提供業務需求

**AI 會進行以下分析：**

1. **限界上下文定義** — 劃定上下文邊界與職責
2. **Use Case → 聚合映射** — 每個 UC 對應到具體的命令、聚合、事件
3. **驗收標準** — Given/When/Then 格式，涵蓋 Happy Path、替代路徑、錯誤路徑
4. **通用語言詞彙表** — 確保術語在同一上下文內唯一且明確

**分析原則：**
- 以領域概念驅動分析，非以功能或畫面驅動
- 每個 Use Case 都要對應到具體聚合的命令
- Given/When/Then 確保可轉為自動化測試
- 產出需足夠具體，讓 SD 能直接據此設計

### Phase 3: SD 戰術設計

**輸入：** Phase 2 的 SA 分析文件

**AI 會設計以下內容：**

1. **聚合結構** — Entity、Value Object、聚合根、不變條件
2. **API 規格** — RESTful API 對齊聚合邊界
3. **套件結構** — Spring Boot + JPA + DDD 標準套件佈局
4. **介面定義** — Repository、Application Service、Domain Service
5. **序列圖** — 關鍵流程的 Mermaid 序列圖
6. **DTO 定義** — Command、Query、Response

**設計原則：**
- API 對齊聚合：一個聚合根 = 一組 RESTful 端點
- 領域邏輯內聚：業務邏輯在 Domain Layer
- 依賴反轉：Domain Layer 不依賴框架
- DTO 隔離：Controller 不暴露 Domain Entity

### Phase 4: 實作規劃

**輸入：** Phase 3 的 SD 設計文件

**AI 會產出以下計畫：**

1. **任務拆解** — 由內而外：Domain → Application → Infrastructure → Presentation → Frontend
2. **TDD 排序** — 每個實作任務前有對應的測試任務
3. **逐檔規格** — 每個任務包含檔案路徑、類型、依賴、介面定義、測試案例、驗收標準
4. **依賴圖** — 任務間的先後順序

**拆解順序（每個聚合）：**
1. Domain Layer（Value Object → Domain Event → Entity → Aggregate Root → Repository Interface → Domain Service）
2. Application Layer（Command/DTO → Application Service → Event Handler）
3. Infrastructure Layer（JPA Mapping → Repository Implementation → DB Migration）
4. Presentation Layer（REST Controller → Request/Response DTO → Exception Handler）
5. Frontend（API Client → Store → Component → Page/Route）

## 使用範例

### 範例 1：從零開始完整流程

```
使用者：我想用 DDD 方法設計一個電商訂單系統

AI 會引導你依序完成：
  Phase 0（若需要）→ Phase 1 → Phase 2 → Phase 3 → Phase 4
```

### 範例 2：只需要 Event Storming

```
使用者：幫我對「線上預約系統」做 Event Storming

AI 會執行 Phase 1，產出結構化的 Event Storming 文件
```

### 範例 3：從 SA 開始

```
使用者：我已經有 Event Storming 產出，幫我做 SA 分析
（提供 Event Storming 文件）

AI 會從 Phase 2 開始
```

### 範例 4：補充 DDD 知識

```
使用者：什麼是 Aggregate？跟 Entity 有什麼差別？

AI 會載入 DDD 理論基礎，用 Java 程式碼範例說明
```

## 可選整合

ddd-core 可與 superpowers 插件搭配使用：

| 時機 | superpowers 技能 | 用途 |
|------|----------------|------|
| Phase 1 之前 | `brainstorming` | 探索需求意圖、釐清業務範圍 |
| Phase 4 實作時 | `test-driven-development` | TDD 紅-綠-重構循環 |
| Phase 4 實作時 | `writing-plans` | 更結構化的實作計畫格式 |
| 實作階段 | `using-git-worktrees` | 在獨立 worktree 中隔離開發 |
| 交付前 | `verification-before-completion` | 確保產出完整性 |

## 檔案結構

```
plugins/ddd-core/
├── .claude-plugin/
│   └── plugin.json                         ← 插件設定（v1.1.0）
├── skills/ddd-core/
│   ├── SKILL.md                            ← AI 指令（英文，~150 行）
│   └── references/                         ← 按需載入的參考資料
│       ├── ddd-theory.md                   ← DDD 理論基礎 + Java 範例
│       ├── event-storming-template.md      ← Phase 1 產出模板
│       ├── sa-template.md                  ← Phase 2 產出模板
│       ├── sd-template.md                  ← Phase 3 產出模板（含套件佈局）
│       └── impl-plan-template.md           ← Phase 4 產出模板
└── docs/
    └── GUIDE.zh-TW.md                     ← 本文件（中文使用指南）
```

**設計理念：**
- **SKILL.md**（~150 行）：精簡的英文指令，AI 遵從度高
- **references/**：按需載入，減少 context 佔用
- **GUIDE.zh-TW.md**：中文人類文件，方便理解和查閱

## 常見問題

### Q: 一定要從 Phase 1 開始嗎？
不一定。如果你已經有部分產出（例如 Event Storming 文件或 SA 分析），可以直接從對應的 Phase 開始。AI 會確認你的輸入是否足夠。

### Q: 支援哪些技術棧？
Phase 3 預設以 Spring Boot + JPA + PostgreSQL 為範例，但設計原則是通用的。你可以告訴 AI 你使用的技術棧，它會適配產出。

### Q: DDD 理論什麼時候會載入？
只有當你明確問 DDD 概念，或 AI 判斷你需要背景知識時才會載入。正常流程中不會自動載入，以節省 context。

### Q: 可以只用部分功能嗎？
可以。例如只做 Event Storming、只做 SA 分析、只需要 DDD 理論解說，都是合法的使用方式。

### Q: 跟 superpowers 插件怎麼配合？
ddd-core 負責 DDD 方法論的流程和產出，superpowers 提供通用的開發最佳實踐（TDD、驗證、計畫撰寫）。兩者互補，非必須搭配。

### Q: 產出的文件格式可以自訂嗎？
模板是建議格式。你可以告訴 AI 你偏好的格式，它會在保持必要資訊的前提下調整。

### Q: 前端設計也涵蓋嗎？
Phase 4 的實作規劃包含前端任務拆解（API Client → Store → Component → Page），但不涉及具體的前端設計。前端 UI/UX 設計不在此技能範圍內。
