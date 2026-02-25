# 設計文件：Explorer + Dev-Team Skills

> 日期：2026-02-25
> 狀態：待實作
> 方案：方案 C（explorer 獨立 + dev-team 內嵌 explorer 作為前置步驟）

---

## 1. 概覽

新增兩個 skill 到 touchfish-skills：

| Skill | 定位 | 多工方式 |
|-------|------|---------|
| **explorer** | 專案偵察工具 | Leader (Opus) + sub-agents (Sonnet) 並行探索 |
| **dev-team** | 多角色開發團隊 | 大團隊流水線，混合 teammates + sub-agents |

兩者獨立運作，dev-team 可在 Phase 0 呼叫 explorer（先後執行，避免巢狀 Teams）。

### 多工策略：Teammates vs Sub-agents

| 面向 | Sub-agent（Task tool） | Teammate（Agent Teams） |
|------|----------------------|------------------------|
| 生命週期 | 短暫：做完就消失 | 持久：存活到團隊關閉 |
| 互相溝通 | 不行，完全隔離 | 可以，透過 SendMessage |
| 共享狀態 | 無，只能透過呼叫者中轉 | 有，共享 TaskList |
| 適合場景 | 獨立、無交互的平行任務 | 需要協作、有依賴的工作 |

**分組策略：**
- **explore 組**：worker 用 sub-agents（各自獨立探索，leader 負責交叉比對）
- **pg 組**：worker 用 teammates（前後端需要互相協調 API 細節）
- **qa 組**：worker 用 sub-agents（各自獨立審查，leader 負責彙整）

---

## 2. Explorer（探索者）

### 2.1 觸發條件

- 關鍵字：explore, 探索, 偵察, 專案地圖, project map, codebase, 了解專案, 專案結構, 探索專案, 分析架構

### 2.2 工作流程

```
Phase 0: 偵測專案類型（explore-leader 獨立執行）
  ├─ 讀取根目錄結構、build 檔、package.json 等
  ├─ 判定：單體 / 多模組 / monorepo+共用基礎設施
  └─ 向使用者確認判定結果與探索範圍

Phase 1: 派出探索 sub-agents
  ├─ 單體 → sub-agent: backend + sub-agent: frontend
  ├─ 多模組 → sub-agent: module-a + sub-agent: module-b + ...
  ├─ 混合型 → sub-agent: infra + sub-agent: module-* + ...
  └─ 所有 sub-agents 並行執行，各自回傳報告

Phase 2: 審視與交叉比對（explore-leader 核心職責）
  ├─ 收回所有 sub-agent 報告
  ├─ 交叉比對：
  │   ├─ 前後端 API 路徑是否一致？
  │   ├─ 型別定義是否對齊？
  │   ├─ 共用設定是否有矛盾？
  │   └─ 模組間依賴是否合理？
  ├─ 審視迴圈：
  │   ├─ 有明確疑問 → 再派 sub-agent 深入探索特定範圍
  │   ├─ 有不確定   → 暫停，詢問使用者協助 → 再派 sub-agent
  │   └─ 無疑慮     → 進入 Phase 3
  └─ 重複直到所有疑慮解除

Phase 3: 彙整產出
  ├─ 產出 PROJECT_MAP.md（存放位置由使用者決定）
  └─ 向使用者呈現摘要
```

### 2.3 智慧混合分工

```
偵測邏輯（偽碼）：

if 根目錄有 packages/ 或 modules/ 或多個 build 檔:
    type = "多模組" 或 "monorepo"
    if 有共用 infra/（docker, k8s, ci）:
        type = "混合型"
        agents = [infra-explorer] + [module-N-explorer for each module]
    else:
        agents = [module-N-explorer for each module]
elif 有明確前後端分離（src/main + src/webapp, 或 server/ + client/）:
    type = "單體"
    agents = [backend-explorer, frontend-explorer]
else:
    type = "單體（簡易）"
    agents = [single-explorer]  # 單一 sub-agent 即可
```

### 2.4 explore-leader 審視準則

explore-leader 使用 Opus，負責所有交叉比對與品質把關：

```
審視檢查清單：
  □ 各報告的技術棧描述是否一致？
  □ 前後端 API 端點路徑是否對齊？
  □ 共用型別/介面定義是否吻合？
  □ 設定檔之間是否有矛盾？
  □ 模組間依賴方向是否合理（無循環依賴）？
  □ 是否有報告遺漏重要目錄或檔案？

任何一項有疑慮 → 再派 sub-agent 或詢問使用者
全部通過 → 產出 PROJECT_MAP.md
```

### 2.5 PROJECT_MAP.md 結構

```markdown
# 專案地圖：<專案名>
> 產出時間：YYYY-MM-DD | 探索範圍：<根目錄>

## 專案概覽
- 類型：單體 / 多模組 / monorepo
- 主要語言：Java 17, TypeScript 5.x
- 框架：Spring Boot 3.2, Vue 3.4
- 建置工具：Gradle 8.5, Vite 5.x

## 架構概覽
（高層級的模組/層級關係描述）

## 模組清單（多模組時）
| 模組 | 路徑 | 職責 | 關鍵依賴 |
|------|------|------|----------|

## 後端結構
- 進入點：...
- 分層：Controller → Service → Repository
- 關鍵設定檔：...

## 前端結構
- 進入點：...
- 元件結構：...
- 狀態管理：...
- 路由：...

## 基礎設施
- CI/CD：...
- 容器化：...
- 環境設定：...

## 關鍵檔案索引
| 檔案 | 用途 | 備註 |
|------|------|------|

## 已知問題與注意事項
（交叉比對發現的不一致、潛在問題等）
```

---

## 3. Dev-Team（開發團隊）

### 3.1 觸發條件

- 關鍵字：dev-team, 開發團隊, team, 組隊開發, 多角色, 團隊協作, 全流程開發

### 3.2 團隊架構

```
Team Lead (Opus) — 充當 PM
├── explore-leader (Opus, teammate)
│   └── sub-agents × N (Sonnet) ← 可反覆派出直到無疑慮
├── pg-leader (Opus, teammate)
│   ├── pg-1 (Sonnet, teammate) — backend
│   ├── pg-2 (Sonnet, teammate) — backend
│   ├── pg-3 (Sonnet, teammate) — frontend
│   ├── pg-4 (Sonnet, teammate) — frontend
│   └── ... 動態調整，數量由 pg-leader 決定
└── qa-leader (Opus, teammate)
    └── sub-agents × N (Sonnet) ← 各自獨立審查
```

**結構原則：**
- **Leader 層**：全部 Opus（需要高判斷力：審視、協調、決策）
- **Worker 層**：預設 Sonnet（確保執行品質），Leader 可視情況降為 Haiku
- **pg 組 workers 用 teammates**（需要互相協調 API 細節、共用 Entity）
- **explore/qa 組 workers 用 sub-agents**（各自獨立，leader 負責彙整比對）

### 3.3 動態規模調整

- Worker 數量由 Leader 根據任務量動態決定，不是固定值
- 中途可追加 Worker（如發現需要額外處理 i18n → spawn pg-5）
- 小型功能可減少至最低配置（每組 1 worker）
- 大型功能可擴展（多個 backend worker 各負責不同 domain）

```
規模範例：

小型功能（1 API + 1 頁面）：
  pg-leader → pg-1 (backend) + pg-2 (frontend)

中型功能（5 API + 3 頁面）：
  pg-leader → pg-1 (backend-entity)
            → pg-2 (backend-api)
            → pg-3 (frontend-頁面A)
            → pg-4 (frontend-頁面B)

大型功能（多 domain）：
  pg-leader → pg-1 ~ pg-3 (backend 各負責不同 domain)
            → pg-4 ~ pg-6 (frontend 各負責不同頁面)
```

### 3.4 Model 選擇指引

| 層級 | 預設 Model | 說明 |
|------|-----------|------|
| Team Lead | Opus | 決策品質要高：需求分析、契約定義、全局協調 |
| Leader 層 | Opus | 需要高判斷力：審視交叉比對、任務拆分、Worker 管理、跨組協調 |
| Worker 層 | Sonnet | 預設 Sonnet 確保執行品質，Leader 可視情況降為 Haiku |

Leader 可在 spawn Worker 時透過 `model` 參數調整，但預設為 Sonnet。

### 3.5 工作流程

```
Phase 0: 專案偵察
  ├─ 若已安裝 explorer skill：
  │   ├─ 呼叫 explorer skill（獨立執行，不在 dev-team 內）
  │   ├─ explorer 完整走完 → 產出 PROJECT_MAP.md
  │   └─ 完成後才進入 Phase 1
  ├─ 若未安裝 explorer：
  │   └─ TL 自行做簡易專案掃描
  └─ 確認專案地圖

Phase 1: 需求分析與任務規劃（TL 獨立執行）
  ├─ 閱讀使用者需求
  ├─ 參考 PROJECT_MAP.md 了解專案現況
  ├─ 拆分為具體任務（TaskCreate）
  ├─ 定義任務依賴（blockedBy / blocks）
  └─ 向使用者確認任務清單 + 驗收標準

Phase 2: API 契約定義（TL 獨立執行）
  ├─ 根據需求產出 API_CONTRACT.md
  │   ├─ 每個端點：Method, Path, Request, Response, 狀態碼
  │   ├─ 共用型別定義
  │   └─ 錯誤回傳格式
  ├─ 向使用者確認契約
  └─ 契約變更必須經過 TL

Phase 3: 組建團隊（TeamCreate）
  ├─ TeamCreate: "dev-<專案名>-<功能名>"
  ├─ Spawn Leaders: explore-leader, pg-leader, qa-leader（全部 Opus）
  ├─ pg-leader 評估任務量後 spawn 適當數量的 pg workers（teammates, Sonnet）
  ├─ TL 將高層任務指派給各 Leader
  └─ pg-leader 拆分並指派給 workers

Phase 3+4: 流水線開發與審查（合併執行）
  ┌─ 流水線運作 ────────────────────────────┐
  │                                          │
  │  pg-leader 指派任務給 pg-1, pg-2...      │
  │                                          │
  │  pg-1 完成 Task A                        │
  │    → TaskUpdate completed                │
  │    → pg-leader 通知 qa-leader            │
  │    → qa-leader 派 sub-agent 審查 Task A  │
  │    → pg-1 繼續 Task B                    │
  │                                          │
  │  pg-2 完成 Task C                        │
  │    → qa-leader 派 sub-agent 審查 Task C  │
  │    → pg-2 繼續 Task D                    │
  │                                          │
  │  qa sub-agent 審查 Task A 發現問題       │
  │    → 回報 qa-leader                      │
  │    → qa-leader 通知 pg-leader            │
  │    → pg-leader 指派 pg-1 修正            │
  │                                          │
  │  中途追加：                               │
  │    pg-leader: "需要額外處理 i18n"         │
  │    → spawn pg-5 (Sonnet, teammate)       │
  │                                          │
  └──────────────────────────────────────────┘

Phase 5: 契約一致性驗證
  ├─ qa-leader 做最終的前後端對接檢查：
  │   ├─ 後端實際 API 是否符合契約？
  │   ├─ 前端呼叫是否符合契約？
  │   ├─ Request/Response 格式是否對齊？
  │   └─ 錯誤處理是否一致？
  ├─ 不一致 → 回到流水線修正
  └─ 一致 → 通過

Phase 6: 交付
  ├─ TL 彙整交付報告：
  │   ├─ 完成了什麼
  │   ├─ 修改的檔案清單
  │   ├─ QA 審查結果
  │   └─ 待注意事項
  ├─ 關閉團隊（TeamDelete）
  ├─ 向使用者呈現報告
  └─ （不自動 commit/push，交由使用者決定）
```

### 3.6 溝通協議

#### 溝通規則

```
允許的溝通路徑（僅限 teammates 之間）：
  ✅ TL ↔ explore-leader, pg-leader, qa-leader
  ✅ pg-leader ↔ pg-1, pg-2, pg-3, ...（leader 管理 workers）
  ✅ pg-leader ↔ qa-leader（跨組協調：任務完成通知、審查回饋）

禁止的溝通路徑：
  ❌ TL → pg-1（不越級指揮）
  ❌ pg-1 → qa-leader（worker 不跨組，透過 pg-leader 轉達）
  ❌ pg-1 → pg-2（同組 worker 透過 pg-leader 協調）

sub-agents 不在溝通網路中（由 leader 派出並收回結果）。
```

#### 每個 Agent 的 Spawn Prompt 必須包含

```
你是 <角色名稱>，隸屬於 <組名>。
- 你的上級：<leader 名稱>（只接受他的指令）
- 你的同事：<同組 worker 列表>（透過 leader 協調）
- 你能直接溝通的對象：<允許清單>
- 不在上述名單的人，不要 SendMessage
```

#### Worker 行為準則（嵌入每個 Worker 的 spawn prompt）

```
你是執行者，不是決策者。嚴格按照任務指示實作。
不確定的事情不要猜測，不要自作主張。

遇到以下情況必須立即 SendMessage 向你的 Leader 回報：
  ├─ 任務描述不清楚或有歧義
  ├─ 發現任務之間的衝突或矛盾
  ├─ 實作過程中遇到預期外的情況
  ├─ 需要修改其他 Worker 負責範圍的檔案
  ├─ 發現潛在的安全性或架構問題
  └─ 任何超出你被指派範圍的事情

寧可多問一次，也不要做錯一次。
回報時請說明：什麼問題、你的判斷、建議選項（如果有的話）。
```

#### 任務指派鏈

```
TL 建立高層任務：
  "實作員工請假功能 - 後端"  → owner: pg-leader
  "實作員工請假功能 - 前端"  → owner: pg-leader
  "探索專案現況"            → owner: explore-leader

pg-leader 拆分並指派：
  "實作 LeaveRequest Entity"     → owner: pg-1
  "實作請假 API endpoints"       → owner: pg-1
  "實作請假審批 Service"          → owner: pg-2
  "實作請假表單元件"              → owner: pg-3
  "實作請假列表頁面"              → owner: pg-3
  "實作請假審批頁面"              → owner: pg-4

qa-leader 派 sub-agent 審查已完成任務：
  Task(subagent) → 審查 "LeaveRequest Entity"
  Task(subagent) → 審查 "請假 API endpoints"
  （可並行派出多個 sub-agent 審查）
```

### 3.7 效率對比

```
原設計（順序 Phase）：
TL ████████░░░░░░░░░░░░░░░░████
BE ░░░░░░░░████████░░░░░░░░░░░░
FE ░░░░░░░░████████░░░░░░░░░░░░
QA ░░░░░░░░░░░░░░░░████████░░░░
                              總時間 ████████████████████████████

流水線大團隊（重疊 + 多 worker）：
TL   ████████░░░░░░░░░░░░████
PG-L ░░░░░░░████████████░░░░░
PG-1 ░░░░░░░░███████████░░░░░   ← backend
PG-2 ░░░░░░░░███████████░░░░░   ← backend
PG-3 ░░░░░░░░███████████░░░░░   ← frontend
PG-4 ░░░░░░░░███████████░░░░░   ← frontend
QA-L ░░░░░░░░░░█████████░░░░░
QA-* ░░░░░░░░░░░████████░░░░░   ← sub-agents 並行審查
                         總時間 █████████████████████████

████ = 工作    ░ = 等待/監控
```

---

## 4. 與現有 Skills 的關係

| | 關係 |
|---|---|
| **explorer ↔ 現有 5 skill** | 獨立，產出物（PROJECT_MAP.md）可被任何 skill 參考 |
| **dev-team ↔ 現有 5 skill** | 獨立運作，不依賴也不取代 |
| **dev-team ↔ explorer** | 軟依賴：Phase 0 可呼叫 explorer（先後執行），未安裝時自行掃描 |
| **explorer ↔ explorer** | 不可巢狀：dev-team 呼叫 explorer 時，explorer 跑完才建 dev-team |

---

## 5. 成本估算

### 單次中型功能開發（8 個開發任務 + 審查）

| 配置 | Agent 數 | 預估 Input | 預估 Output | 估計成本 |
|------|---------|-----------|-------------|---------|
| 精簡版 | 3 | ~200K | ~60K | ~$1.5 |
| 大團隊（Leaders Opus + Workers Sonnet） | 8-10 | ~400K | ~120K | ~$5-8 |

### Model 定價參考

| Model | Input | Output |
|-------|-------|--------|
| Opus | $15/M | $75/M |
| Sonnet | $3/M | $15/M |
| Haiku | $0.25/M | $1.25/M |

> 注意：Leader 全部升為 Opus 後，成本較先前估算提高。
> 實際成本取決於任務複雜度、worker 數量、審視迴圈次數。

---

## 6. Plugin 結構

```
plugins/
├── explorer/
│   ├── .claude-plugin/
│   │   └── plugin.json
│   └── skills/
│       └── explorer/
│           └── SKILL.md
└── dev-team/
    ├── .claude-plugin/
    │   └── plugin.json
    └── skills/
        └── dev-team/
            └── SKILL.md
```

marketplace.json 新增：
```json
{
  "name": "explorer",
  "source": "./plugins/explorer",
  "description": "專案探索者：Opus Leader 指揮 sub-agents 並行探索，交叉比對後產出專案地圖",
  "tags": ["explore", "project-map", "codebase", "architecture", "agent-teams"]
},
{
  "name": "dev-team",
  "source": "./plugins/dev-team",
  "description": "開發團隊：多角色流水線協作（PM/開發者/QA），動態規模，混合 teammates + sub-agents",
  "tags": ["team", "agent-teams", "pipeline", "pm", "qa", "development"]
}
```
