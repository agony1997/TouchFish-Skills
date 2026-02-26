# dev-team v2 改進設計

> 日期：2026-02-26 | 基於：收貨領域實作 Session 回饋報告

---

## 1. 問題摘要

根據 Mock-ODS-VUE 收貨領域實作的 SESSION_ANALYSIS.md，dev-team v1.0.0 有以下核心問題：

| # | 問題 | 嚴重度 |
|---|------|--------|
| 1 | pg-leader 自己做完 15/19 任務，完全沒 spawn workers | 高 |
| 2 | TL 催促 pg-leader spawn 被忽略（2 次） | 高 |
| 3 | QA 流水線未實現（qa-leader 從頭等到尾） | 高 |
| 4 | 使用者不確定單一規格 vs 多規格的使用方式 | 中 |
| 5 | 並行利用率僅 ~20% | 中 |

**根因：** dev-team 的核心假設「Leader 會自覺分派工作給 Workers」在實務中不成立。Agent 天然傾向直接執行而非協調。

---

## 2. 設計決策

| 問題 | 決定 | 理由 |
|------|------|------|
| spawn 控制權 | **混合：TL spawn workers，pg-leader 管理指派** | TL 保有控制權，pg-leader 專注管理 |
| 任務範圍 | **智慧判斷：TL 分析依賴後決定並行或序列** | 兩種模式都需要支援 |
| Leader model | **保持 Opus** | 協調和審查需要高判斷力 |
| QA 觸發 | **TL 主動通知 qa-leader** | 不依賴 pg-leader 轉達 |

---

## 3. 改進設計

### 3.1 Phase 1 改進：多規格書智慧判斷

**新增「輸入模式判斷」步驟：**

1. TL 接收使用者輸入（可能是 1 份或多份規格書/需求）

2. 如果是多份規格書，TL 分析：
   - 領域間是否有依賴？（如：收貨依賴訂貨）
   - 是否共用資料庫表/Entity？
   - 是否共用 API 路徑前綴？

3. 使用 AskUserQuestion 向使用者確認執行策略：

   - **並行模式**：「這 N 個領域互相獨立，建議並行開發」
     → 每個領域分配獨立 worker 組
     → pg-leader 協調跨領域共用部分

   - **序列模式**：「領域 A 依賴領域 B，建議先做 B 再做 A」
     → 按順序規劃任務

   - **單一專精模式**：「建議每次專注一個領域，做完再做下一個」
     → 給使用者明確建議

4. 無論哪種模式，TL 都必須向使用者說明理由，讓使用者做最終決定。

### 3.2 Phase 3 重構：TL spawn workers，pg-leader 管理

**原本流程：**
```
TL spawn leaders → leaders spawn workers
```

**新流程：**

1. TL spawn leaders（pg-leader, qa-leader, 可選 explore-leader）

2. TL 根據任務清單決定 worker 規模：
   ```
   規則：
   - ≤3 個實作任務 → 不 spawn workers，pg-leader 自己做
   - 4-8 個實作任務 → 至少 2 workers（1 backend + 1 frontend 或 2 backend）
   - ≥9 個實作任務 → 至少 3 workers
   - 有前端+後端 → 必須至少各 1 worker
   ```

3. TL 直接 spawn workers（帶 team_name），worker prompt 中指定上級為 pg-leader

4. TL 透過 TaskUpdate 將高層任務指派給 pg-leader。pg-leader 負責：
   - 拆分細粒度子任務
   - 指派給 workers（TaskUpdate owner）
   - 監督進度、協調依賴
   - 可向 TL 請求追加 workers（但不能自己 spawn）

5. pg-leader prompt 加入硬約束：
   ```
   你是協調者和管理者。你有 workers 可以指派任務。
   你不應該自己寫實作程式碼，除非任務數量 ≤3 且 TL 未派 workers。
   如果你有 workers，你必須把任務指派給他們。
   ```

### 3.3 Phase 4 重構：TL 主導 QA 觸發

**原本流程：**
```
worker 完成 → pg-leader → qa-leader
```

**新流程：**
```
worker 完成 → pg-leader 收到通知 → pg-leader 回報 TL
→ TL 通知 qa-leader 審查
```

**TL 的 Phase 4 職責增加：**
- 監控 TaskList 狀態
- 收到 pg-leader 完成回報時，主動 SendMessage 通知 qa-leader
- 不依賴任何 agent 轉達

**pg-leader prompt 加入：**
```
每完成一個任務（或收到 worker 完成通知），你必須 SendMessage 向 TL 回報進度，
包含：完成了什麼、下一步計畫、是否有阻塞。
```

**qa-leader prompt 調整：**
```
你的審查觸發來自 TL 的通知。
當 TL 通知你某任務已完成，立即派 sub-agent 審查。
你也應該每個 turn 開頭主動 TaskList 檢查是否有漏掉的完成任務。
```

### 3.4 通訊紀律強化

**所有 agent prompt 加入訊息響應約束：**

```
溝通紀律：
- 收到上級訊息後，你必須在回應的開頭先處理該訊息
- 如果是指令：確認收到 → 說明執行計畫
- 如果你不同意：必須說明理由，不可沉默忽略
- 沉默忽略上級訊息是不可接受的行為

定期回報：
- 每完成一個 Phase 的任務，暫停並 SendMessage 向上級回報
- 回報內容：已完成什麼、下一步、是否需要支援
```

---

## 4. 架構變更對照

### 4.1 團隊架構變更

```
v1:
TL (Opus)
├── explore-leader (Opus) → sub-agents
├── pg-leader (Opus) → pg-leader 自行 spawn workers
└── qa-leader (Opus) → sub-agents

v2:
TL (Opus) ← spawn 權集中在 TL
├── explore-leader (Opus) → sub-agents
├── pg-leader (Opus) ← 只做管理，不 spawn
│   ├── pg-1 (Sonnet) ← TL spawn，交由 pg-leader 管理
│   ├── pg-2 (Sonnet) ← TL spawn，交由 pg-leader 管理
│   └── ...
└── qa-leader (Opus) ← TL 主動通知觸發
    └── sub-agents
```

### 4.2 溝通路徑變更

```
v1 QA 觸發：
  pg-leader → qa-leader（跨組通知）

v2 QA 觸發：
  pg-leader → TL → qa-leader（TL 中繼，確保通知發生）
```

### 4.3 Phase 流程變更

| Phase | v1 | v2 變更 |
|-------|----|----|
| Phase 1 | 單一需求分析 | 新增多規格書智慧判斷 |
| Phase 3 | pg-leader spawn workers | TL spawn workers，pg-leader 管理 |
| Phase 4 | pg-leader 通知 qa-leader | TL 主動通知 qa-leader |
| 全局 | 無通訊約束 | 加入通訊紀律和定期回報 |

---

## 5. 需要修改的檔案

| 檔案 | 變更內容 |
|------|---------|
| `plugins/dev-team/skills/dev-team/SKILL.md` | Phase 1/3/4 重構 + 通訊紀律 + prompt 模板更新 |
| `plugins/dev-team/.claude-plugin/plugin.json` | 版本升至 1.1.0 |
