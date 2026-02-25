---
name: spec-to-md
description: >
  讀取規格文件轉換為 AI coding 實作文件（.md），產出技術規格、後端實作指示、前端實作指示等結構化文件。
  使用時機：當使用者提供規格文件並要求產生實作文件、轉換規格為 coding 文件時觸發。
  關鍵字：規格轉換、spec to md、產生實作文件、讀取規格、分析需求、coding 文件、開發文件、
  to md、toMD、toMd、TOMD、specification, 規格文件, 需求文件, 需求轉換, 實作指示, 實作文件, 技術規格, 文件生成, 文件產生, 前端實作, 後端實作, convert, 轉換。
---

# Spec-to-MD：規格文件轉 AI Coding 實作文件

## 流程

### 1. 確認輸入文件

觸發後，**先向使用者確認以下資訊**（使用 AskUserQuestion）：

1. **功能專屬規格文件**：有幾份？路徑為何？（如需求書、畫面規格、SQL 等）
2. **專案規範文件**：有哪些？路徑為何？（如 DEVELOP.md、NAMING_CONVENTIONS.md、API_SPECIFICATION.md 等）
3. **共通組件文件**：有哪些？路徑為何？（如元件文件目錄、共用元件原始碼路徑等）
4. **產出目錄**：文件要產出到哪裡？（預設 `Docs/<功能編號>/`）

等使用者提供完整資訊後才進入下一步。

### 2. 讀取與分析（並行 Subagents）

> **為什麼用 Subagents**：此階段只是讀取和分析，不需要 agent 之間互相溝通，subagents 效率高且成本低。

使用 **3 個並行 Task Agent** 同時讀取和分析：

**Agent A — 功能規格分析**：
- 讀取所有功能規格文件
- 產出：功能名稱、編號、前後端需求、業務規則、畫面結構、欄位說明、操作流程

**Agent B — 專案規範學習**：
- 讀取所有專案規範文件（DEVELOP.md、NAMING_CONVENTIONS.md、API_SPECIFICATION.md 等）
- 產出：命名慣例、架構模式、API 格式、回應格式、Entity 模式、錯誤碼規範

**Agent C — 既有程式碼風格學習**：
- 讀取共通組件文件
- 搜尋既有同類功能的程式碼（用 Glob/Grep）
- 產出：實際程式碼風格、可用共用元件清單、Processor/Store/Vue 元件的寫法模式

三個 Agent 使用 `Task` 工具並行啟動（在同一個訊息中發送多個 Task 呼叫，每個 Task 指定 `model: "opus"`，**不設定 `run_in_background`**，等待所有 Agent 回傳結果後再繼續），完成後主流程彙整結果。

大型專案中 Docs 下可能有大量文件，**只讀取使用者指定的文件**，不主動掃描整個目錄。若需要額外文件，向使用者確認。

### 3. 確認需求理解

使用 `superpowers:brainstorming` 的方法論探索需求和設計意圖。彙整 3 個 Agent 的結果後，**向使用者提交需求理解摘要**，確認再繼續：

- 功能名稱與編號
- 前端需求摘要（幾個頁面、主要操作流程、關鍵元件）
- 後端需求摘要（幾個 API、主要業務邏輯、關鍵驗證規則）
- 涉及的資料表與 Entity
- 預計產出的 Processor 數量和 Vue 元件數量
- 有疑問或不確定的地方（明確列出）

**等使用者確認理解正確後**才開始撰寫文件。

### 4. 產出文件

#### 第一階段：`prompt.md` — AI 實作大綱（主流程產出）

使用 `superpowers:writing-plans` 的方法論來規劃實作大綱。給 AI（md-to-code skill）看的總指引，內容為：
- 功能概述（一段話說明功能目的、執行時機、前提條件）
- 實作範圍摘要（後端幾個 Processor、前端幾個元件）
- 指向詳細文件的引用清單，說明每份文件的用途和閱讀順序
- 實作順序建議（先後端再前端，或依相依性排序）
- 專案規範文件引用路徑（讓實作時能參考）

**prompt.md 本身不包含技術細節**，只做導航和統籌。

→ 產出後向使用者說明重點，等待確認。

#### 第二階段：`01_技術規格.md`（主流程產出）

API、資料庫、Entity 的全部技術細節，語氣為規格說明式：
- API 端點清單（表格：端點、方法、權限、說明）
- 每個 API 的請求參數、回應格式、錯誤碼
- 資料表結構與關聯（ER 圖）
- Entity 欄位對應
- 業務規則定義
- 相關 SQL 參考

→ 產出後向使用者說明關鍵設計決策（如權限命名、API 路徑），等待確認。

#### 第三階段（Agent Teams 並行）：`02_後端實作.md` + `03_前端實作.md`

> **可選整合** — 若已安裝 superpowers 插件，可搭配 `superpowers:dispatching-parallel-agents` 使用，以其並行任務調度方法論管理 Agent Teams。

> **為什麼改用 Agent Teams**：後端和前端實作文件需要保持一致性（API 端點名稱、參數格式、回應結構），teammates 可以**直接互相校對**確保文件一致，使用者也可以**隨時切入任一 teammate 修正方向**。

01_技術規格確認後，建立 Agent Team 並行產出：

```
01_技術規格.md 確認
       ↓
  TeamCreate: "spec-<功能名稱>"
       ↓
┌──────┴──────┐
↓             ↓
backend-spec  frontend-spec    ← teammates（各自獨立 context）
↓             ↓
│  ← 互相校對一致性 →  │       ← 使用者可 Shift+Down 切入
└──────┬──────┘
       ↓
  Team lead 彙整結果
```

**團隊設定**：

```
TeamCreate:
  team_name: "spec-<功能名稱>"
  description: "並行產出後端與前端實作文件"
```

**backend-spec** teammate：
- Prompt 包含：功能規格摘要、專案規範摘要、01_技術規格完整內容
- 產出 `02_後端實作.md`（混合式：規格 + 指令）：
  - 檔案清單（路徑 + 說明）
  - 每個 Processor 的實作指示：
    - 「建立 `XxxProcessor.java`，繼承 `ApiRouteProcessor`」
    - 實作 `getTemplateParams()` 回傳值
    - `processBusinessLogic()` 的業務邏輯步驟
    - 參數提取方式（指定使用 JsonUtil）
    - 驗證邏輯描述
    - 關鍵程式碼片段
  - Route XML 配置指示
  - Entity 方法清單（若需新增方法則給出簽名和邏輯描述）
- **完成後透過 `SendMessage` 將 API 端點清單和回應格式發送給 frontend-spec 供校對**

**frontend-spec** teammate：
- Prompt 包含：功能規格摘要、專案規範摘要、01_技術規格完整內容
- 產出 `03_前端實作.md`（混合式：規格 + 指令）：
  - 檔案清單（路徑 + 說明）
  - Types 定義（完整 interface + 常數 + 工具函數）
  - Store 實作指示：
    - State 欄位定義
    - Getters 清單與計算邏輯
    - Actions 的 API 呼叫與狀態管理邏輯
  - 主頁面結構（組件樹 + 各區塊說明）
    - 畫面設計描述（排版、欄位配置）
    - 按鈕狀態邏輯（啟用/禁用條件）
    - 對話框與通知邏輯
  - 子元件的 Props / Events / 特性
  - 路由配置
  - 使用的共用組件對應表
- **收到 backend-spec 的 API 端點清單後，校對 Types 和 Store Actions 的 API 呼叫是否一致**
- **如果發現不一致，透過 `SendMessage` 與 backend-spec 協調修正**

→ **兩份都完成後，team lead 彙整並同時提交使用者確認**：分別說明後端 Processor 職責劃分和前端元件拆分邏輯，以及跨層一致性校對結果。

### 5. 關閉團隊 + 最終確認

使用 `superpowers:verification-before-completion` 的驗證原則，在宣告完成前進行最終檢查：

1. **關閉 Agent Team**：
   - 向所有 teammates 發送 `shutdown_request`
   - 等待 teammates 確認關閉
   - 使用 `TeamDelete` 清理團隊資源

2. **完整性驗證**：比對步驟 3 確認的需求摘要，確認所有需求都已涵蓋在文件中

3. **一致性檢查**：確認 02/03 文件中的 API 端點、Entity 欄位、元件名稱與 01_技術規格一致
   - 列出跨層校對紀錄（teammates 之間的溝通和修正項目）

4. **自足性確認**：確認 02/03 文件各自包含足夠 context，不需頻繁交叉引用

全部驗證通過後，列出完整清單供使用者做最終檢視：
- 各文件一句話摘要
- 驗證結果（是否有遺漏或不一致）
- 跨層一致性校對結果
- 提醒使用者可使用 md-to-code skill 開始實作

## 注意事項

- 程式碼片段須符合專案既有風格（從讀取的規範和既有程式碼學習）
- 02/03 文件應自帶足夠 context，實作時不需頻繁交叉引用 01
- 每份文件控制在合理長度，避免超出 Claude Code 的有效處理範圍
- **步驟 2（讀取分析）使用 Subagents**：效率高、成本低，適合不需互動的分析任務
- **步驟 4 第三階段（文件產出）使用 Agent Teams**：支援跨層校對一致性、人工介入
- Teammates 的 prompt 須包含完整 context（功能規格摘要、專案規範摘要、01_技術規格完整內容），確保獨立作業品質一致
