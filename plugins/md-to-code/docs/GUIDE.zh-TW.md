# md-to-code 技能使用指南

> 版本：1.2.0 | 最後更新：2026-02-26

## 這是什麼？

md-to-code 是一個**根據結構化實作文件自動產生程式碼**的技能。你提供由 spec-to-md 產出的實作文件（技術規格、後端實作、前端實作），它會組建 Agent Teams 並行開發後端與前端程式碼。

核心特色：
- **並行開發**：後端和前端由兩個 teammate 同時進行
- **跨層溝通**：後端發現 API 變更可直接通知前端
- **人工閘門**：關鍵節點強制暫停，等你確認後才繼續
- **你可以隨時介入**：Shift+Down 切換到任一 teammate 直接修正

---

## 前置條件

需要以下實作文件（通常由 spec-to-md 產出）：

| 文件 | 內容 |
|------|------|
| `prompt.md` | 導航大綱：列出所有文件路徑和專案規範位置 |
| `01_技術規格.md` | API 端點、資料表、Entity、業務規則 |
| `02_後端實作.md` | 後端逐步實作指示 |
| `03_前端實作.md` | 前端逐步實作指示 |

---

## 使用流程

```
步驟 1          1.5           2            3           3.5          4           5
讀取文件  ───→  確認理解  ───→  產生計畫  ───→  並行實作  ───→  確認結果  ───→  驗證  ───→  報告
(3 subagents)  (人工閘門)     (寫計畫)     (Agent Teams) (人工閘門)     (審查)     (關閉)
```

### 步驟 1：讀取大綱與規範

AI 使用 3 個並行 Subagent（Opus）同時讀取所有文件：

| Agent | 負責內容 | 產出 |
|-------|---------|------|
| A | 01_技術規格.md | API 摘要、DB 結構、業務規則 |
| B | 專案規範 + 既有程式碼探索 | 命名慣例、架構模式、**可複用元件清單** |
| C | 02_後端實作 + 03_前端實作 | 步驟清單、檔案清單 |

> **為什麼用 Subagents？** 此階段只是讀取分析，不需要 agent 之間互相溝通，subagents 效率高且成本低。

### 步驟 1.5：確認 AI 理解（人工閘門）

AI 會展示：
1. 需求理解摘要（3-5 句）
2. 找到的可複用既有元件
3. 未確定的問題或疑點

**你需要回覆**：正確繼續 / 需要補充 / 需要調整方向

### 步驟 2：產生實作計畫

AI 根據彙整結果，使用 superpowers:writing-plans 方法論列出步驟清單：

```
實作計畫：<功能名稱>

後端（依 02_後端實作.md）：
  [] 1. 建立 Entity 和 Repository
  [] 2. 實作 Processor
  [] 3. ...

前端（依 03_前端實作.md）：
  [] 1. 建立 Types 定義
  [] 2. 實作 Store
  [] 3. ...
```

你可以調整步驟順序或範圍。

### 步驟 3：Agent Teams 並行實作

AI 建立團隊，生成兩個 teammate 並行開發：

```
       Team Lead (你的對話)
       ┌──────┴──────┐
       │             │
  backend-dev   frontend-dev
       │             │
       │← 可直接互相傳訊 →│
       │← 共享 task list →│
```

| Teammate | 負責 | 輸入 |
|----------|------|------|
| backend-dev | 所有後端檔案 | 01_技術規格 + 02_後端實作 + 專案規範 + 可複用元件 |
| frontend-dev | 所有前端檔案 | 01_技術規格 + 03_前端實作 + 專案規範 + 可複用元件 |

**跨層溝通**：
- backend-dev 調整 API → 自動通知 frontend-dev
- frontend-dev 發現 API 不符 → 直接跟 backend-dev 協調

**你可以隨時介入**：按 Shift+Down 切換到任一 teammate，直接下指令修正。

### 步驟 3.5：實作結果確認（人工閘門）

AI 展示：
1. 後端產出摘要（檔案清單、關鍵決策、偏離說明）
2. 前端產出摘要（檔案清單、元件拆分、偏離說明）
3. 跨層協調紀錄
4. 需注意項目

**你需要回覆**：符合預期 / 後端需調整 / 前端需調整 / 兩邊都要調

### 步驟 4：實作後驗證

AI 進行：
1. 檔案完整性檢查（對照實作計畫）
2. 程式碼審查（如有安裝 reviewer 技能則使用，否則 AI 自行審查）
3. 提示你進行 build 和功能測試

### 步驟 5：關閉團隊 + 完成報告

1. 關閉所有 teammates，清理團隊資源
2. 產出 `04_完成報告.md`，包含：
   - 問題與決策紀錄（整個流程中的 Q&A）
   - 完成項目清單（後端 + 前端）
   - 跨層協調紀錄
   - 尚需手動處理的項目

---

## 使用範例

### 基本使用

```
你：「用 md-to-code 根據 docs/receive/ 的實作文件開始寫程式」
→ AI 讀取 prompt.md 找到所有文件路徑
→ 走完步驟 1-5
→ 產出完整程式碼 + 04_完成報告.md
```

### 搭配 spec-to-md

```
你：「先用 spec-to-md 產出實作文件，再用 md-to-code 實作」
→ spec-to-md 產出 01~03 文件
→ md-to-code 根據文件實作程式碼
```

### 只做後端或前端

```
你：「用 md-to-code 只實作後端」
→ AI 偵測 prompt.md 中只有 02_後端實作（或你明確要求）
→ 只生成 backend-dev teammate，跳過前端相關步驟
```

---

## 可選整合（非必要，未安裝也能正常運作）

| 插件 | 用途 | 時機 |
|------|------|------|
| superpowers:writing-plans | 計畫撰寫方法論 | 步驟 2 |
| superpowers:test-driven-development | TDD 紅-綠-重構 | 步驟 3 實作中 |
| superpowers:systematic-debugging | 系統化除錯 | 步驟 3 遇 bug 時 |
| superpowers:verification-before-completion | 驗證原則 | 步驟 4 |
| superpowers:requesting-code-review | 程式碼審查 | 步驟 4 |
| superpowers:finishing-a-development-branch | 分支結案 | 步驟 5 後 |

---

## 檔案結構

```
plugins/md-to-code/
├── .claude-plugin/plugin.json              ← 插件元資料 (v1.2.0)
├── skills/md-to-code/
│   ├── SKILL.md                            ← AI 核心指令（英文，始終載入）
│   ├── prompts/                            ← Spawn 模板（按需載入）
│   │   ├── backend-dev.md                  ← 後端 teammate
│   │   └── frontend-dev.md                 ← 前端 teammate
│   └── references/
│       └── completion-report-template.md   ← 完成報告格式
└── docs/
    └── GUIDE.zh-TW.md                     ← 本文件（中文，給人讀的）
```

---

## 常見問題

**Q: 需要先跑 spec-to-md 嗎？**
A: 不一定。只要你有結構化的實作文件（prompt.md + 01~03），就可以直接用 md-to-code。

**Q: 可以只做後端或只做前端嗎？**
A: 可以。告訴 AI 只需要哪一端，它會只生成對應的 teammate。

**Q: teammate 之間怎麼溝通？**
A: 透過 SendMessage 直接互傳。例如 backend-dev 改了 API 格式，會自動通知 frontend-dev。

**Q: 我可以中途介入 teammate 嗎？**
A: 可以。按 Shift+Down 切換到任一 teammate 的 context，直接下指令。

**Q: 步驟 1 為什麼用 Subagents 不用 Agent Teams？**
A: 步驟 1 只是讀取分析，不需要 agent 間互動。Subagents 效率高、成本低。步驟 3 才需要跨層溝通，所以用 Agent Teams。

**Q: 完成報告會包含什麼？**
A: 問題與決策紀錄、完成項目清單（後端+前端）、跨層協調紀錄、尚需手動處理的項目。
