---
name: md-to-code
description: >
  根據結構化的 AI coding 實作文件（.md）逐步實作程式碼。讀取 prompt.md 大綱，
  依序參考技術規格、後端實作指示、前端實作指示，逐步確認後實作後端與前端程式碼。
  使用時機：當使用者要求根據已產出的實作文件開始寫程式、實作功能、按照文件 coding、
  依文件實作、開始開發時觸發。關鍵字：md to code、根據文件實作、開始寫程式、
  按文件 coding、依文件開發、實作程式碼、開始實作、to code、toCode、implement, implementation, 寫程式, coding, 依規格實作, 按規格開發, 從文件產生程式碼, generate code, code generation, 開始開發。
---

# MD-to-Code：根據實作文件逐步實作

## 流程

### 1. 讀取大綱與規範（並行 Subagents）

讀取 `prompt.md` 取得文件導航和專案規範路徑後，使用 **3 個並行 Task Agent**（model: opus）同時讀取所有必要文件：

> **為什麼用 Subagents**：此階段只是讀取和分析，不需要 agent 之間互相溝通，subagents 效率高且成本低。

**Agent A — 技術規格**：
- 通讀 `01_技術規格.md`
- 產出：API 端點摘要、資料表結構、Entity 對應、業務規則清單

**Agent B — 專案規範 + 既有程式碼風格 + 上下文探索**：
- 讀取 prompt.md 中列出的專案規範文件（DEVELOP.md、NAMING_CONVENTIONS.md、API_SPECIFICATION.md 等）
- 讀取 `CLAUDE.md`
- 搜尋既有同類功能程式碼（用 Glob/Grep），學習實際風格
- **上下文探索**：根據本次功能涉及的模組和關鍵字，搜尋可複用的既有元件：
  - 後端：同模組的 Processor、Entity、Repository、共用工具類別
  - 前端：同模組的 Vue 元件、Store、Types、共用組件
  - SQL：同模組已有的 migration 檔案（確認版本號和表結構）
- 產出：命名慣例、架構模式、程式碼風格摘要、**可複用元件清單**

**Agent C — 實作文件**：
- 通讀 `02_後端實作.md` 和 `03_前端實作.md`
- 產出：後端步驟清單、前端步驟清單、檔案清單

三個 Agent 在同一個訊息中用 `Task` 工具並行啟動（**不設定 `run_in_background`**，等待所有 Agent 回傳結果後再繼續），完成後主流程彙整結果。

### 1.5 Checkpoint：確認 AI 理解與上下文（人工閘門）

**此步驟為強制暫停**，使用 AskUserQuestion 展示以下摘要，等待使用者確認後才繼續：

1. **需求理解摘要**：用 3-5 句話概述本次功能的目標和範圍
2. **可複用的既有元件**：列出 Agent B 找到的相關程式碼
   - 「以下是與本次功能相關的既有元件，我計畫參考/複用它們：」
   - 列出檔案路徑和用途
3. **潛在問題或疑點**：若有不確定的地方，明確列出

問題：「以上理解正確嗎？有遺漏的既有元件嗎？」

選項：
- **正確，繼續** — 進入步驟 2
- **需要補充** — 使用者補充遺漏的元件或修正理解
- **需要調整方向** — 使用者說明調整需求

### 2. 產生實作計畫

使用 `superpowers:writing-plans` 技能的方法論，根據彙整結果列出完整步驟清單：

```
實作計畫：<功能名稱>

後端（依 02_後端實作.md）：
  □ 1. <步驟名稱> — <說明>
  □ 2. <步驟名稱> — <說明>
  □ ...

前端（依 03_前端實作.md）：
  □ 1. <步驟名稱> — <說明>
  □ 2. <步驟名稱> — <說明>
  □ ...
```

提交使用者確認後才開始實作。使用者可在此調整步驟順序或範圍。

### 3. Agent Teams 並行實作（後端 + 前端）

> **可選整合** — 若已安裝 superpowers 插件，實作過程中可搭配 `superpowers:test-driven-development` 使用，以 TDD 紅-綠-重構循環確保程式碼品質。

> **可選整合** — 若已安裝 superpowers 插件，實作過程中遇到 bug 可搭配 `superpowers:systematic-debugging` 使用，以系統化方式定位問題。

> **為什麼改用 Agent Teams**：實作階段需要跨層溝通（如前端發現 API 格式問題可直接跟後端 teammate 協調），且使用者可以隨時用 **Shift+Down 切換到任一 teammate 直接介入修正方向**。

使用 `TeamCreate` 建立實作團隊，然後用 `Task` 工具（帶 `team_name` 參數）生成 teammates：

```
確認實作計畫
       ↓
  TeamCreate: "impl-<功能名稱>"
       ↓
  TaskCreate: 建立後端 + 前端任務清單
       ↓
┌──────┴──────┐
↓             ↓
backend-dev   frontend-dev     ← teammates（各自獨立 context）
↓             ↓
│  ← 可直接互相傳訊 →  │       ← 使用者可 Shift+Down 切入任一 teammate
│  ← 共享 task list →  │
└──────┬──────┘
       ↓
  Team lead 彙整結果
       ↓
  提交使用者確認
```

#### 團隊設定

```
TeamCreate:
  team_name: "impl-<功能名稱>"
  description: "根據實作文件並行開發後端與前端"
```

#### 建立任務清單

使用 `TaskCreate` 在 team task list 中建立任務（從步驟 2 的實作計畫轉換）：

```
後端任務（owner: backend-dev）：
  - 建立 Entity 和 Repository
  - 實作 Processor
  - 設定 Route XML
  - ...

前端任務（owner: frontend-dev）：
  - 建立 Types 定義
  - 實作 Store
  - 建立 Vue 元件
  - 設定路由
  - ...
```

#### 生成 Teammates

**backend-dev** teammate：
- 使用 `Task` 工具，指定 `team_name` 和 `name: "backend-dev"`
- Prompt 包含完整 context：01_技術規格 + 02_後端實作 + 專案規範摘要 + 可複用元件清單
- 指示：依 02_後端實作.md 的步驟順序逐步實作，完成每個任務後用 `TaskUpdate` 標記完成
- 產出所有後端檔案（Entity、Repository、Processor、SQL 配置等）
- **如果發現需要調整 API 格式或新增端點，透過 `SendMessage` 通知 frontend-dev**

**frontend-dev** teammate：
- 使用 `Task` 工具，指定 `team_name` 和 `name: "frontend-dev"`
- Prompt 包含完整 context：01_技術規格 + 03_前端實作 + 專案規範摘要 + 可複用元件清單
- 指示：依 03_前端實作.md 的步驟順序逐步實作，完成每個任務後用 `TaskUpdate` 標記完成
- 產出所有前端檔案（Types、Store、Vue 元件、路由配置等）
- **如果發現 API 回傳格式與預期不符，透過 `SendMessage` 與 backend-dev 協調**

#### 使用者可隨時介入

- **Shift+Down** 切換到任一 teammate 的 context
- 直接在 teammate 中下指令修正方向
- 不需要等 teammate 完成再回到主流程修改

#### Team Lead 彙整

所有 teammates 完成任務後：
- 彙整兩個 teammate 的產出結果
- 列出所有新增/修改的檔案清單（後端 + 前端）
- 列出 teammates 之間的溝通紀錄（如 API 協調紀錄）
- 說明各 teammate 的關鍵實作決策

### 3.5 Checkpoint：實作結果確認（人工閘門）

**此步驟為強制暫停**，使用 AskUserQuestion 展示以下內容，等待使用者確認後才進入驗證：

1. **後端產出摘要**：
   - 新增/修改的檔案清單
   - 關鍵實作決策（如業務邏輯處理方式、API 設計選擇）
   - 是否有偏離實作文件的地方（若有，說明原因）

2. **前端產出摘要**：
   - 新增/修改的檔案清單
   - 元件拆分方式和狀態管理策略
   - 是否有偏離實作文件的地方

3. **跨層協調紀錄**：
   - teammates 之間的溝通紀錄摘要（如 API 格式調整）

4. **需要注意的項目**：
   - 未能完全實作的功能（若有）
   - 需要手動調整的部分

問題：「以上實作結果符合預期嗎？」

選項：
- **符合，進入驗證** — 進入步驟 4
- **後端需要調整** — 使用者說明調整需求，重新指派給 backend-dev
- **前端需要調整** — 使用者說明調整需求，重新指派給 frontend-dev
- **兩邊都要調** — 使用者說明調整需求

若使用者選擇調整，透過 `SendMessage` 將調整需求傳給對應 teammate，等待完成後再次提交確認。

### 4. 實作後驗證

使用 `superpowers:verification-before-completion` 的驗證原則，搭配 `superpowers:requesting-code-review` 進行品質確認：

1. **檔案完整性檢查**：
   - 比對實作計畫，確認所有檔案都已產出
   - 列出所有新增/修改的檔案清單

2. **程式碼審查**（使用 code-reviewer Agent）：
   - 檢查程式碼是否符合實作文件規格
   - 檢查命名和風格是否與專案規範一致
   - 標記潛在問題或需手動調整的項目

3. **提示使用者**：
   - 進行編譯驗證（後端 build、前端 build）
   - 進行功能測試
   - 標記任何需要手動調整的項目

### 5. 關閉團隊 + 產出完成報告

驗證通過後：

1. **關閉 Agent Team**：
   - 向所有 teammates 發送 `shutdown_request`
   - 等待 teammates 確認關閉
   - 使用 `TeamDelete` 清理團隊資源

2. **產出 `04_完成報告.md`**，包含以下兩部分：

#### 問題條列

彙整整個流程中（spec-to-md + md-to-code）向使用者提出的問題、使用者的回覆、以及過程中發現的疑點或設計決策：

```
## 問題與決策紀錄

| # | 階段 | 問題 | 結論/使用者回覆 |
|---|------|------|-----------------|
| 1 | 需求確認 | ... | ... |
| 2 | 實作中 | ... | ... |
```

#### 完成項目清單

列出已完成的實作項目（簡要即可，不需逐行描述）：

```
## 完成項目

### 後端
- [x] <Processor/檔案名稱> — <一句話說明>
- [x] ...

### 前端
- [x] <元件/檔案名稱> — <一句話說明>
- [x] ...

### 跨層協調紀錄
- <協調項目> — <結論>

### 尚需手動處理
- [ ] <項目> — <原因>
```

產出後告知使用者報告位置。

## 實作原則

- 嚴格按照實作文件中的規格，不自行添加未定義的功能
- 優先使用專案既有的共用元件和工具類別
- 命名和風格須與專案既有程式碼一致
- 若實作文件與專案規範有衝突，優先遵循專案規範並告知使用者
- **步驟 1（讀取）使用 Subagents**：效率高、成本低，適合不需互動的分析任務
- **步驟 3（實作）使用 Agent Teams**：支援跨層溝通、人工介入、共享任務列表
- Teammates 的 prompt 須包含完整 context（技術規格全文、實作指示全文、專案規範摘要），確保獨立作業品質一致
- 所有 Task Agent / Teammates 指定 `model: "opus"`

> **可選整合** — 若已安裝 superpowers 插件，完成實作後可搭配 `superpowers:finishing-a-development-branch` 使用，引導開發分支的結案流程。
