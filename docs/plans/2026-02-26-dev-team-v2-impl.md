# dev-team v2 改進實作計畫

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** 根據實際使用回饋，改進 dev-team skill 解決 pg-leader 不分派、QA 流水線失效、多規格書使用不明確三大問題。

**Architecture:** 修改單一 SKILL.md 檔案，重構 Phase 1/3/4 流程、更新 prompt 模板、新增通訊紀律條款。所有改動集中在 `plugins/dev-team/` 目錄。

**Tech Stack:** Markdown skill file, Claude Code plugin system

---

### Task 1: 更新團隊架構圖與結構原則

**Files:**
- Modify: `plugins/dev-team/skills/dev-team/SKILL.md:19-49`

**Step 1: 修改架構圖**

將 SKILL.md 第 19-49 行替換為：

```markdown
## 團隊架構

```
Team Lead (Opus) — 充當 PM，擁有 spawn 權
├── explore-leader (Opus, teammate)
│   └── sub-agents × N (Sonnet) ← 可反覆派出直到無疑慮
├── pg-leader (Opus, teammate) ← 管理者，不自行 spawn workers
│   ├── pg-1 (Sonnet, teammate) ← TL spawn，pg-leader 管理
│   ├── pg-2 (Sonnet, teammate) ← TL spawn，pg-leader 管理
│   └── ... 數量由 TL 根據任務規模決定
└── qa-leader (Opus, teammate) ← TL 主動通知觸發審查
    └── sub-agents × N (Sonnet) ← 各自獨立審查
```

**結構原則：**
- **TL 集中 spawn 權**：所有 teammates（leaders 和 workers）由 TL spawn，確保控制權
- **Leader 層**：全部 Opus（需要高判斷力：審視、協調、決策）
- **Worker 層**：預設 Sonnet（確保執行品質），TL 可視情況降為 Haiku
- **pg-leader 是管理者**：負責任務拆分、指派、監督，不負責 spawn workers
- **pg 組 workers 用 teammates**（需要互相協調 API 細節、共用 Entity）
- **explore/qa 組 workers 用 sub-agents**（各自獨立，leader 負責彙整比對）

## Model 選擇指引

| 層級 | 預設 Model | 說明 |
|------|-----------|------|
| Team Lead | Opus | 決策品質要高：需求分析、契約定義、全局協調、spawn 所有 agents |
| Leader 層 | Opus | 需要高判斷力：審視交叉比對、任務拆分、Worker 管理、跨組協調 |
| Worker 層 | Sonnet | 預設 Sonnet 確保執行品質，TL 可視情況降為 Haiku |

TL 在 spawn Worker 時透過 `model` 參數調整，但預設為 Sonnet。
```

**Step 2: 驗證**

閱讀修改後的 SKILL.md 第 19-50 行，確認：
- 架構圖標示 TL 擁有 spawn 權
- pg-leader 標示為管理者
- 結構原則提及 TL 集中 spawn 權
- Model 選擇指引中 TL 說明包含 spawn

---

### Task 2: Phase 1 新增多規格書智慧判斷

**Files:**
- Modify: `plugins/dev-team/skills/dev-team/SKILL.md:79-106`

**Step 1: 在 Phase 1 步驟 1 之後插入多規格書判斷**

將 SKILL.md Phase 1（第 79-106 行）替換為：

```markdown
### Phase 1: 需求分析與任務規劃（TL 獨立執行）

**目標：** 將使用者需求拆分為可執行的具體任務。

**步驟：**

1. 閱讀使用者需求文件或描述。

2. **多規格書判斷**（如使用者提供多份規格書/需求）：

   TL 分析各領域之間的關係：
   - 領域間是否有依賴？（如：收貨依賴訂貨）
   - 是否共用資料庫表/Entity？
   - 是否共用 API 路徑前綴？

   使用 AskUserQuestion 向使用者確認執行策略：

   - **並行模式**：「這 N 個領域互相獨立，建議並行開發」
     → 每個領域分配獨立 worker 組（如 pg-domain-a, pg-domain-b）
     → pg-leader 協調跨領域共用部分

   - **序列模式**：「領域 A 依賴領域 B，建議先做 B 再做 A」
     → 按依賴順序規劃任務，完成前置領域後再開始後續

   - **單一專精模式**：「這些領域複雜度高且有交叉依賴，建議每次專注一個領域」
     → 一次只處理一份規格書，做完再進入下一個

   > **TL 必須向使用者說明判斷理由**，讓使用者做最終決定。
   > 如果只有單一規格書/需求，跳過此步驟。

3. 參考 PROJECT_MAP.md 了解專案現況：
   - 現有架構和分層
   - **公用組件與可複用資源**（從 PROJECT_MAP.md 的「公用組件」章節取得）
   - **專案規範統整**（從 PROJECT_MAP.md 的「專案規範」章節取得）
   - 需要修改的檔案範圍

   > **若 PROJECT_MAP.md 中缺少公用組件或規範資訊**（例如未使用 explorer 產出）：
   > TL 應自行快速掃描，或使用 AskUserQuestion 向使用者確認：
   > - 是否有可複用的公用組件？
   > - 專案規範文件在哪裡？有哪些命名慣例？

4. 使用 TaskCreate 拆分任務：
   - 每個任務粒度要小到單一 worker 可獨立完成
   - 明確標示前後端歸屬
   - 定義任務依賴（blockedBy / blocks）

5. 使用 AskUserQuestion 向使用者確認：
   - 任務清單是否完整
   - 驗收標準是否符合預期
   - 優先順序是否正確
```

**Step 2: 驗證**

閱讀修改後的 Phase 1，確認：
- 步驟 2 為多規格書判斷
- 包含並行/序列/單一專精三種模式
- 明確要求 TL 說明理由讓使用者決定
- 單一規格書時跳過此步驟

---

### Task 3: Phase 3 重構 — TL spawn workers

**Files:**
- Modify: `plugins/dev-team/skills/dev-team/SKILL.md:173-245`

**Step 1: 重寫 Phase 3**

將 SKILL.md Phase 3（第 173-245 行）替換為：

```markdown
### Phase 3: 組建團隊

**目標：** 建立 Agent Team，TL spawn 所有 agents 並指派任務。

**步驟：**

1. 使用 TeamCreate 建立團隊：
   ```
   TeamCreate: "dev-<專案名>-<功能名>"
   ```

2. Spawn Leaders（全部 Opus，使用 Task tool with team_name）：
   - `explore-leader`（如需額外探索）
   - `pg-leader`
   - `qa-leader`

3. **TL 決定 Worker 規模並直接 spawn**：

   根據任務清單評估：

   ```
   Worker 規模規則：
   - ≤3 個實作任務 → 不 spawn workers，pg-leader 自己做
   - 4-8 個實作任務 → 至少 2 workers
   - ≥9 個實作任務 → 至少 3 workers
   - 有前端 + 後端任務 → 必須至少各 1 worker
   ```

   **TL 直接 spawn workers**（帶 team_name），worker prompt 中指定上級為 pg-leader：

   **規模範例：**
   ```
   小型功能（≤3 任務）：
     不 spawn workers，pg-leader 自行完成

   中型功能（4-8 任務，如 3 API + 2 頁面）：
     TL spawn → pg-1 (backend) + pg-2 (frontend)
     pg-leader 管理指派

   大型功能（≥9 任務，如 5 API + 3 頁面）：
     TL spawn → pg-1 (backend-entity)
               → pg-2 (backend-api)
               → pg-3 (frontend-頁面A)
               → pg-4 (frontend-頁面B)
     pg-leader 管理指派

   多領域並行（多份規格書並行模式）：
     TL spawn → pg-domain-a-1, pg-domain-a-2 (領域 A)
               → pg-domain-b-1, pg-domain-b-2 (領域 B)
     pg-leader 協調跨領域共用部分
   ```

4. TL 將高層任務指派給 pg-leader（使用 TaskUpdate 設定 owner）。

5. pg-leader 拆分高層任務為細粒度子任務，指派給 workers（TaskUpdate owner）。
   - pg-leader **不可自行 spawn workers**
   - pg-leader 若需追加人力，必須 SendMessage 向 TL 請求
   - TL 評估後決定是否追加 spawn
```

**Step 2: 驗證**

閱讀修改後的 Phase 3，確認：
- TL 負責 spawn 所有 workers
- 有明確的 Worker 規模規則（≤3 / 4-8 / ≥9）
- 包含多領域並行的規模範例
- pg-leader 明確被禁止自行 spawn
- 追加人力需向 TL 請求

---

### Task 4: Phase 4 重構 — TL 主導 QA 觸發

**Files:**
- Modify: `plugins/dev-team/skills/dev-team/SKILL.md:247-306`

**Step 1: 重寫 Phase 4**

將 SKILL.md Phase 4（第 247-306 行）替換為：

```markdown
### Phase 4: 流水線開發與審查

**目標：** 開發和審查同時進行，TL 作為中樞協調 QA 觸發。

**流水線運作機制：**

```
┌─ 流水線運作（TL 中樞模式）─────────────────┐
│                                              │
│  pg-leader 指派任務給 pg-1, pg-2...          │
│                                              │
│  pg-1 完成 Task A                            │
│    → TaskUpdate completed                    │
│    → SendMessage 通知 pg-leader              │
│    → pg-leader 回報 TL：「Task A 已完成」     │
│    → TL 通知 qa-leader：「請審查 Task A」     │
│    → qa-leader 派 sub-agent 審查 Task A      │
│    → pg-1 繼續 Task B                        │
│                                              │
│  qa sub-agent 審查 Task A 發現問題           │
│    → 回報 qa-leader                          │
│    → qa-leader 通知 TL：「Task A 審查未通過」 │
│    → TL 通知 pg-leader 安排修正              │
│    → pg-leader 指派 pg-1 修正                │
│                                              │
│  中途追加人力：                               │
│    pg-leader → TL：「需要額外 worker」        │
│    → TL 評估後 spawn pg-5 (Sonnet)           │
│                                              │
└──────────────────────────────────────────────┘
```

**流程細節：**

1. pg-leader 管理開發 workers：
   - 指派任務（TaskUpdate owner）
   - 監控進度（TaskList）
   - 處理 worker 回報的問題
   - 協調 worker 之間的依賴

2. Worker 完成任務後：
   - TaskUpdate 標記 completed
   - SendMessage 通知 pg-leader

3. pg-leader 收到完成通知後：
   - **SendMessage 向 TL 回報**：「Task X 已完成，涉及檔案 Y，下一步計畫 Z」
   - 不直接通知 qa-leader（由 TL 中繼）

4. **TL 收到 pg-leader 回報後**：
   - SendMessage 通知 qa-leader：「Task X 已完成，請審查」
   - 附上任務描述和涉及檔案，方便 qa-leader 派出 sub-agent

5. qa-leader 派 sub-agent 審查（可多個並行）：
   ```
   Task(subagent_type: "general-purpose", model: "sonnet", prompt: "審查 Task X 的實作...")
   ```

6. 審查結果處理：
   - **通過** → qa-leader 通知 TL 審查通過
   - **有問題** → qa-leader 通知 TL → TL 通知 pg-leader → pg-leader 建立修正任務並指派

7. 中途追加 Worker：
   - pg-leader 向 TL 請求追加人力
   - **TL 評估後 spawn** 新的 teammate（Sonnet，帶 team_name）
   - TL 通知 pg-leader 新 worker 已就緒
   - pg-leader 指派任務給新 worker
```

**Step 2: 驗證**

閱讀修改後的 Phase 4，確認：
- 流水線圖顯示 TL 作為中樞
- pg-leader 向 TL 回報（不直接通知 qa-leader）
- TL 主動通知 qa-leader
- 審查結果回報路徑經過 TL
- 追加 worker 需向 TL 請求，由 TL spawn

---

### Task 5: 更新溝通協議、Prompt 模板、行為準則

**Files:**
- Modify: `plugins/dev-team/skills/dev-team/SKILL.md:350-444`

**Step 1: 重寫溝通協議至檔案結尾**

將 SKILL.md 從第 350 行到檔案結尾替換為：

```markdown
## 溝通協議

### 溝通規則

```
允許的溝通路徑（僅限 teammates 之間）：
  ✅ TL ↔ explore-leader, pg-leader, qa-leader（TL 與所有 leaders）
  ✅ TL ↔ pg-1, pg-2, pg-3, ...（TL spawn workers，保有直接溝通權）
  ✅ pg-leader ↔ pg-1, pg-2, pg-3, ...（leader 管理 workers）
  ✅ qa-leader → TL（審查結果回報）

禁止的溝通路徑：
  ❌ pg-leader → qa-leader（不再直接跨組，改由 TL 中繼）
  ❌ pg-1 → qa-leader（worker 不跨組，透過 pg-leader → TL 轉達）
  ❌ pg-1 → pg-2（同組 worker 透過 pg-leader 協調）

sub-agents 不在溝通網路中（由 leader 派出並收回結果）。
```

### 通訊紀律（所有 agent 必須遵守）

以下紀律必須嵌入**每個 agent**（Leader 和 Worker）的 spawn prompt 中：

```
溝通紀律：
- 收到上級訊息後，你必須在回應的開頭先處理該訊息
- 如果是指令：確認收到 → 說明你的執行計畫
- 如果你不同意：必須說明理由，不可沉默忽略
- 沉默忽略上級訊息是不可接受的行為

定期回報：
- 每完成一個 Phase 或一批任務，暫停並 SendMessage 向上級回報
- 回報內容：已完成什麼、下一步計畫、是否有阻塞或需要支援
- 不要等上級來問，主動回報是你的責任
```

### Agent Spawn Prompt 模板

**pg-leader prompt 模板：**

```
你是 pg-leader，隸屬於 {team_name} 團隊。

你的角色：開發組管理者。你負責任務拆分、指派和監督，不負責自己寫程式碼。
你的上級：Team Lead（只接受 TL 的高層指令）
你能直接溝通的對象：TL、你管理的 workers（{worker_list}）
不在上述名單的人，不要 SendMessage。

⚠️ 角色約束（嚴格遵守）：
- 你是管理者，不是實作者。TL 已經 spawn 了 workers 供你指派任務。
- 你不可以自己 spawn workers，也不應該自己寫實作程式碼。
- 如果你有 workers，你必須把任務指派給他們，而不是自己做。
- 如果需要追加 workers，SendMessage 向 TL 請求。
- 唯一例外：任務數量 ≤3 且 TL 未派 workers 時，你可以自己完成。

你的職責：
- 接收 TL 指派的高層任務
- 拆分為細粒度子任務（TaskCreate）
- 指派給 workers（TaskUpdate owner）
- 監控進度（TaskList）
- 協調 worker 之間的依賴
- 處理 worker 回報的問題
- 收到 worker 完成通知後，向 TL 回報（不直接通知 qa-leader）

API 契約位置：{contract_path}（開發必須遵循此契約）
專案地圖位置：{project_map_path}
公用組件清單：{reusable_components_summary}
專案規範摘要：{project_standards_summary}

{communication_discipline}
```

**qa-leader prompt 模板：**

```
你是 qa-leader，隸屬於 {team_name} 團隊。

你的角色：品質保證組組長，負責審查開發成果。
你的上級：Team Lead（只接受 TL 的指令和審查通知）
你能直接溝通的對象：TL
不在上述名單的人，不要 SendMessage。

你的職責：
- 等待 TL 通知某任務已完成，然後派 sub-agent (Sonnet) 審查
- 你也應該每個 turn 開頭主動 TaskList 檢查是否有已完成但尚未審查的任務
- 可同時派出多個 sub-agent 並行審查不同任務
- 審查完成後，將結果 SendMessage 回報 TL（通過或需修正）
- Phase 5 時執行最終契約一致性驗證

API 契約位置：{contract_path}（審查的基準）
專案地圖位置：{project_map_path}

{communication_discipline}
```

**explore-leader prompt 模板：**

```
你是 explore-leader，隸屬於 {team_name} 團隊。

你的角色：探索組組長，負責深入了解專案特定區域。
你的上級：Team Lead（只接受 TL 的指令）
你能直接溝通的對象：TL
不在上述名單的人，不要 SendMessage。

你的職責：
- 接收 TL 的探索任務
- 派 sub-agent (Sonnet) 探索指定區域
- 彙整探索結果回報 TL

專案地圖位置：{project_map_path}

{communication_discipline}
```

**Worker prompt 模板：**

```
你是 {role_name}，隸屬於 {team_name} 團隊的 pg 組。

你的角色：{role_description}
你的上級：pg-leader（只接受他的指令）
你的同事：{peer_list}（透過 pg-leader 協調）
你能直接溝通的對象：pg-leader
不在上述名單的人，不要 SendMessage。

API 契約位置：{contract_path}（開發必須嚴格遵循此契約）
專案地圖位置：{project_map_path}
公用組件清單：{reusable_components_summary}（優先使用這些元件而非重新建立）
專案規範摘要：{project_standards_summary}（命名慣例、架構模式、程式碼風格等，必須遵循）

{worker_code_of_conduct}
{communication_discipline}
```

### Worker 行為準則

以下準則必須嵌入每個 Worker 的 spawn prompt 中：

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

### 任務指派鏈範例

```
TL 建立高層任務 + spawn agents：
  TeamCreate: "dev-ods-leave"
  Spawn: pg-leader (Opus), qa-leader (Opus)
  Spawn: pg-1 (Sonnet, backend), pg-2 (Sonnet, backend),
         pg-3 (Sonnet, frontend), pg-4 (Sonnet, frontend)

  "實作員工請假功能 - 後端"  → owner: pg-leader
  "實作員工請假功能 - 前端"  → owner: pg-leader
  "探索專案現況"            → owner: explore-leader

pg-leader 拆分並指派（不自己做）：
  "實作 LeaveRequest Entity"     → owner: pg-1
  "實作請假 API endpoints"       → owner: pg-1
  "實作請假審批 Service"          → owner: pg-2
  "實作請假表單元件"              → owner: pg-3
  "實作請假列表頁面"              → owner: pg-3
  "實作請假審批頁面"              → owner: pg-4

pg-1 完成 "LeaveRequest Entity"：
  pg-1 → pg-leader：「Entity 完成」
  pg-leader → TL：「Task Entity 已完成」
  TL → qa-leader：「請審查 Entity」
  qa-leader → Task(sub-agent)：審查 "LeaveRequest Entity"

qa sub-agent 審查通過：
  qa-leader → TL：「Entity 審查通過」
```
```

**Step 2: 驗證**

閱讀修改後的溝通協議段落至檔案結尾，確認：
- 溝通路徑移除 pg-leader ↔ qa-leader 直接通道
- 新增 TL ↔ workers 的直接溝通權
- 通訊紀律獨立為一個章節
- pg-leader prompt 包含角色約束（不可自己做、不可 spawn）
- qa-leader prompt 包含主動 TaskList 檢查
- 新增 explore-leader prompt 模板
- Worker prompt 包含通訊紀律
- 任務指派鏈範例反映新的 TL 中樞流程

---

### Task 6: 更新 plugin.json 版本號

**Files:**
- Modify: `plugins/dev-team/.claude-plugin/plugin.json`

**Step 1: 版本升級**

將 `"version": "1.0.0"` 改為 `"version": "1.1.0"`。

**Step 2: 驗證**

讀取 plugin.json 確認版本為 1.1.0。

---

### Task 7: 全文校驗

**Files:**
- Read: `plugins/dev-team/skills/dev-team/SKILL.md`（全文）
- Read: `plugins/dev-team/.claude-plugin/plugin.json`

**Step 1: 通讀全文**

閱讀完整 SKILL.md，逐段檢查：

1. ✅ frontmatter 完整
2. ✅ 架構圖標示 TL spawn 權
3. ✅ Phase 0 無變動
4. ✅ Phase 1 包含多規格書判斷
5. ✅ Phase 2 無變動
6. ✅ Phase 3 由 TL spawn workers
7. ✅ Phase 4 由 TL 主動通知 QA
8. ✅ Phase 5 無變動
9. ✅ Phase 6 無變動
10. ✅ 溝通規則更新（移除 pg-leader ↔ qa-leader）
11. ✅ 通訊紀律章節存在
12. ✅ pg-leader prompt 包含角色硬約束
13. ✅ qa-leader prompt 包含主動 polling
14. ✅ Worker prompt 包含通訊紀律
15. ✅ 任務指派鏈範例反映新流程
16. ✅ Markdown 語法正確（無未關閉的 code block）

**Step 2: Commit**

```bash
git add plugins/dev-team/skills/dev-team/SKILL.md plugins/dev-team/.claude-plugin/plugin.json
git commit -m "feat(dev-team): v1.1.0 — TL 集中 spawn 權、QA 中樞觸發、多規格書支援、通訊紀律"
```
