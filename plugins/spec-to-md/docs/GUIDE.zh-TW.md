# spec-to-md 技能使用指南

> 版本：1.2.0 | 最後更新：2026-02-26

## 這是什麼？

spec-to-md 是一個**規格文件轉 AI Coding 實作文件**的技能。你提供規格文件（需求書、畫面規格、SQL 等），
它會產出結構化的實作文件，讓 AI（md-to-code skill）能直接依文件開發。

產出文件：
- `prompt.md` — AI 實作大綱（導航用）
- `01_技術規格.md` — API、資料庫、Entity 技術細節
- `02_後端實作.md` — 後端 Processor 實作指示
- `03_前端實作.md` — 前端元件實作指示

---

## 使用流程

```
步驟 1          步驟 2           步驟 3
確認輸入  ───→  讀取分析   ───→  確認理解
(問你檔案路徑)  (3 並行 agent)   (摘要確認)
                                    │
                                    ▼
步驟 4                           步驟 5
產出文件  ──────────────────────→ 關閉團隊
 ├─ 4a: prompt.md (主流程)         + 最終確認
 ├─ 4b: 01_技術規格.md (主流程)
 └─ 4c: 02 + 03 (Agent Teams 並行)
```

### 步驟 1: 確認輸入文件

技能觸發後會問你以下問題：

1. **功能專屬規格文件**：有幾份？路徑為何？（需求書、畫面規格、SQL 等）
2. **專案規範文件**：路徑為何？（如 DEVELOP.md、NAMING_CONVENTIONS.md、API_SPECIFICATION.md）
3. **共通組件文件**：路徑為何？（元件文件目錄、共用元件原始碼路徑）
4. **產出目錄**：文件要產出到哪裡？（預設 `Docs/<功能編號>/`）

提供完整資訊後才會繼續。

### 步驟 2: 讀取與分析

使用 3 個並行 Subagent（Opus）同時讀取分析：

| Agent | 任務 | 產出 |
|-------|------|------|
| **A — 功能規格** | 讀取所有功能規格文件 | 功能名稱、編號、前後端需求、業務規則、畫面結構、欄位說明、操作流程 |
| **B — 專案規範** | 讀取專案規範文件 | 命名慣例、架構模式、API 格式、回應格式、Entity 模式、錯誤碼規範 |
| **C — 程式碼風格** | 讀取共通組件 + 搜尋同類程式碼 | 實際程式碼風格、可用共用元件清單、Processor/Store/Vue 元件寫法模式 |

> 只讀取你指定的文件，不會主動掃描整個目錄。

### 步驟 3: 確認需求理解

彙整分析結果後，向你提交需求理解摘要：

- 功能名稱與編號
- 前端需求摘要（頁面數、操作流程、關鍵元件）
- 後端需求摘要（API 數量、業務邏輯、驗證規則）
- 涉及的資料表與 Entity
- 預計 Processor 數量和 Vue 元件數量
- 有疑問或不確定的地方

**等你確認後才開始撰寫文件。**

### 步驟 4: 產出文件

#### 4a: prompt.md — AI 實作大綱（主流程）

給 md-to-code skill 看的總指引：
- 功能概述
- 實作範圍摘要
- 詳細文件引用清單
- 實作順序建議
- 專案規範文件引用路徑

**prompt.md 不包含技術細節**，只做導航和統籌。產出後等你確認。

#### 4b: 01_技術規格.md（主流程）

API、資料庫、Entity 的全部技術細節：
- API 端點清單（表格）
- 請求參數、回應格式、錯誤碼
- 資料表結構與關聯
- Entity 欄位對應
- 業務規則定義
- 相關 SQL 參考

產出後等你確認關鍵設計決策。

#### 4c: 02_後端實作.md + 03_前端實作.md（Agent Teams 並行）

01 確認後，建立 Agent Team 並行產出：

```
01_技術規格.md 確認
       |
  TeamCreate: "spec-<功能名稱>"
       |
 ------+------
 |            |
backend-    frontend-
spec        spec        <-- teammates（各自獨立 context）
 |            |
 |  <-- 互相校對一致性 -->  |    <-- 你可以 Shift+Down 切入
 ------+------
       |
  Team lead 彙整結果
```

- **backend-spec**：產出 02_後端實作.md（檔案清單、Handler 實作指示、路由配置、Entity 方法）
- **frontend-spec**：產出 03_前端實作.md（檔案清單、Types、State Management、頁面結構、子元件、路由、共用組件）
- 兩者完成後由 **Team Lead 協調校對** API 端點、參數格式、回應結構的一致性（避免競態條件）

完成後 Team Lead 彙整並提交你確認。

### 步驟 5: 關閉團隊 + 最終確認

1. 關閉 Agent Team（shutdown + TeamDelete）
2. 完整性驗證：比對步驟 3 需求摘要，確認全部涵蓋
3. 一致性檢查：02/03 文件的 API 端點、Entity 欄位、元件名稱與 01 一致
4. 自足性確認：02/03 文件各自包含足夠 context

最後列出：
- 各文件一句話摘要
- 驗證結果
- 跨層一致性校對結果
- 提醒你可使用 md-to-code skill 開始實作

---

## 產出文件說明

| 文件 | 內容 | 閱讀者 |
|------|------|--------|
| `prompt.md` | 實作大綱、文件導航 | AI（md-to-code skill） |
| `01_技術規格.md` | API、DB、Entity 規格 | AI + 人類 |
| `02_後端實作.md` | 後端 Processor 實作指示 | AI（md-to-code skill） |
| `03_前端實作.md` | 前端元件實作指示 | AI（md-to-code skill） |

---

## 使用範例

### 基本用法

```
你：「用 spec-to-md 把收貨功能的規格轉成實作文件」
→ 技能問你檔案路徑
→ 你提供規格文件、專案規範、共用元件路徑
→ 技能產出 4 份文件
```

### 搭配 md-to-code

```
你：「規格文件已經用 spec-to-md 產出了，在 Docs/ODS001/ 下，用 md-to-code 開始實作」
→ md-to-code 讀取 prompt.md 開始依文件實作
```

---

## 檔案結構

```
plugins/spec-to-md/
├── .claude-plugin/plugin.json          ← 插件元資料 (v1.2.0)
├── skills/spec-to-md/
│   ├── SKILL.md                        ← AI 核心指令（英文，始終載入）
│   ├── prompts/                        ← Spawn 模板（按需載入）
│   │   ├── backend-spec.md             ← 後端實作 teammate
│   │   └── frontend-spec.md            ← 前端實作 teammate
│   └── references/
│       └── template-structure.md       ← 文件模板結構（254 行）
└── docs/
    └── GUIDE.zh-TW.md                  ← 本文件（中文，給人讀的）
```

---

## 常見問題

**Q: 一定要提供專案規範文件嗎？**
A: 建議提供。有了 DEVELOP.md、NAMING_CONVENTIONS.md 等，產出的文件才能貼合專案風格。沒有的話技能會用通用慣例。

**Q: 可以只產出後端或前端嗎？**
A: 建議一次產出全套（4 份文件）以確保跨層一致性。如果只需要其中一部分，可以在步驟 3 確認時說明，但請注意跨層校對將無法進行。

**Q: Agent Teams 的 backend-spec 和 frontend-spec 會做什麼？**
A: 它們各自獨立撰寫實作文件，完成後互相校對 API 一致性。你也可以用 Shift+Down 切入任一 teammate 修正方向。

**Q: 產出的文件格式遵循什麼模板？**
A: 遵循 `references/template-structure.md` 定義的結構。該文件定義了每份文件的章節和內容要求。

**Q: 可以搭配 superpowers 插件嗎？**
A: 可以但非必要。如果安裝了 superpowers 插件，步驟 3 會使用 brainstorming 探索需求，步驟 4a 使用 writing-plans 規劃大綱，步驟 5 使用 verification-before-completion 做最終驗證。沒有安裝也能正常運作。
