# TouchFish-Skills 技能專業審查報告

> **審查日期**：2026-03-03
> **審查者**：Claude Opus 4.6（Claude Skill 建立者 / 系統架構師視角）
> **專案版本**：marketplace v5.0.0（7 個工作流型插件）

---

## 專案總覽

7 個工作流型 Claude Code 技能插件，覆蓋從需求到交付的完整軟體開發生命週期：

| Skill | 版本 | 行數 | refs | prompts | 定位 |
|-------|------|------|------|---------|------|
| ddd-core | 1.1.0 | 149 | 5 | — | 方法論（DDD 理論→實作規劃） |
| git-nanny | 1.2.0 | 141 | 4 | — | 操作流程（Git 全生命週期） |
| reviewer | 1.2.0 | 140 | 2 | 1 | 審查+萃取（雙工作流） |
| spec-to-md | 1.2.0 | 135 | 1 | 2 | 轉換（規格→結構化文件） |
| md-to-code | 1.2.0 | 144 | 1 | 2 | 實作（文件→程式碼） |
| explorer | 1.2.0 | 109 | 1 | 1 | 偵察（並行探索→PROJECT_MAP） |
| dev-team | 3.0.0 | 160 | 4 | 5 | 團隊協作（多角色流水線）— **已拆分至 [TouchFish-DevTeam](https://github.com/agony1997/TouchFish-DevTeam)** |

---

## 架構評價：優點

### 1. 統一架構模式 — 非常成熟

所有 7 個 skill 遵循相同結構：

```
SKILL.md（英文、始終載入、60-160 行）
├── references/（模板、理論、按需 Glob+Read）
├── prompts/（Agent spawn 模板、按需載入填變數）
└── docs/GUIDE.zh-TW.md（中文人類指南）
```

**英文給 AI + 中文給人** 的雙語策略是經過驗證的最佳實踐 — AI 對英文指令遵從度高，中文指南讓使用者易讀。始終載入量僅 ~912 行（v1.0 的 30%），context 管理意識強。

### 2. 工作流定位清晰 — 哲學正確

「只保留有明確工作流的技能，純知識交給 AI 本身」— 這正是 skill 設計的核心原則。每個 skill 都是**可執行的工作流程**，不是參考手冊。

### 3. Agent 協作分層精準

```
Sub-agents (Explore) = 讀取/分析（低成本、無通訊、用完即棄）
Agent Teams (teammates) = 協作開發（跨層通訊、共享任務列表、人機互動）
```

spec-to-md、md-to-code、dev-team 精確地在讀取階段用 sub-agents、在開發階段用 Agent Teams，成本效益最佳化做得好。

### 4. 人機閘門設計完善

每個涉及 Agent Teams 的 skill 都有 **mandatory gates**（`AskUserQuestion`）在關鍵決策點暫停。md-to-code 的 Step 1.5 和 3.5 是典範 — 確認理解後才動工，實作後確認結果才進驗證。

### 5. dev-team v3.0 的設計水準最高

- **分離測試**：test-agent（Opus）先寫測試 → Worker 只管讓測試通過，避免 worker 自己寫「永遠通過」的測試
- **三方交叉驗證 QA**：需求↔測試、測試↔程式碼、需求↔程式碼
- **嚴格 scope enforcement**：ALLOWED/READONLY/FORBIDDEN 三級檔案權限
- **LLM-native 格式**：`[TYPE] key=value | key=value` 比 Markdown 表格更省 token
- **Worker prompt** 品質極高：任務明確、通訊紀律嚴格、scope 違規後果清楚

### 6. 可選整合不造成硬相依

所有 superpowers 整合點都用 `> **Optional integration**` 標記，缺少 superpowers 時 skill 仍可獨立運作。

---

## 架構評價：可改善之處

### 1. 跨 Skill 交接（Handoff）缺乏明確機制

README 的 pipeline 圖很清楚：

```
ddd-core → spec-to-md → md-to-code → git-nanny
```

但**沒有任何 skill 內部有明確的交接指示**。例如：

- ddd-core Phase 4 結束後，沒有提示「輸出格式相容 spec-to-md 輸入」
- md-to-code Step 5 完成後，沒有建議「使用 git-nanny 提交」
- spec-to-md 的 prompt.md 是 md-to-code 的入口，但 spec-to-md 沒有明確保證輸出格式與 md-to-code 的 `Expected input format` 相容

**建議**：在每個 skill 的末尾加入 `## Next Step` 提示下游 skill，並在 references 中定義跨 skill 的介面契約。

### 2. 中斷恢復（Resume）策略缺失

所有 skill 都是「從頭到尾」的流程設計。如果 context window 用盡、session 中斷、或使用者隔天繼續：

- dev-team 有 file-as-memory（PLAN/CONTRACT/logs），理論上可恢復，但沒有明確的「resume from Phase N」指示
- spec-to-md 和 md-to-code 沒有恢復機制 — Agent Teams 一旦中斷，所有 teammate 狀態丟失
- explorer 的 Phase 2 review loop 中斷後無法接續

**建議**：至少在 dev-team 和 md-to-code 中加入「Resume Protocol」section，說明如何從文件狀態推斷進度並繼續。

### 3. README 版本號與 plugin.json 不一致

README 列 reviewer 為 1.1.0，但 `plugin.json` 和 SKILL.md 都標示 1.2.0。marketplace.json 版本為 5.0.0，與各插件版本的關係未說明。

### 4. ddd-core 與 dev-team 的定位重疊區域

ddd-core Phase 4（Implementation Planning）產出 TDD 任務列表 + 依賴圖 + 每檔案規格。dev-team Phase 1 也做需求分析 + 任務拆解 + 檔案 scope。

當使用者走完整 DDD 流程後用 dev-team 開發，Phase 1 會重新分析已經在 ddd-core Phase 4 完成的工作。**兩者銜接點不明確**。

**建議**：在 dev-team Phase 1 加入「如果已有 ddd-core Phase 4 輸出，直接轉換為 PLAN.md，跳過重複分析」的快速通道。

### 5. 模型選擇策略可以更精確

目前的模型分配：

- Opus：TL、test-agent、qa-global、explorer leader
- Sonnet：Worker、qa-task、delivery-sub、explore sub-agents

基本正確，但缺少成本估算指引。使用者可能不清楚一次 dev-team 全流程大約消耗多少 token/多少成本。考量到 dev-team 可能 spawn 10+ agents，**成本可見性**對使用者決策很重要。

### 6. Prompt 模板的變數驗證

所有 prompts/ 都用 `{variable}` 佔位符，由 TL/Leader 填入。但如果某個變數為空或路徑不存在，沒有 fallback 機制。例如 worker.md 的 `{contract_path}` 標注 `(skip if "none")`，但其他變數沒有類似的空值處理。

### 7. explorer 的敏感檔案保護可更完整

explorer 列了 `.env`, `.pem`, `.key` 等，但沒有涵蓋 `.npmrc`（可能含 auth token）、`docker-compose.yml`（可能含 DB 密碼）、`.env.local`、`*.pfx` 等。建議擴充或改用 pattern-based 排除。

---

## 各 Skill 個別評分

| Skill | 工作流設計 | Prompt 品質 | 可維護性 | Agent 協作 | 完整度 | 綜合 |
|-------|-----------|------------|---------|-----------|--------|------|
| ddd-core | A | A | A | N/A | A- | **A** |
| git-nanny | A | A | A | N/A | A | **A** |
| reviewer | A | A | A | A- | A | **A** |
| spec-to-md | A | A | A- | A | A- | **A** |
| md-to-code | A | A | A- | A | A- | **A** |
| explorer | A | A | A | A- | A | **A** |
| dev-team | A+ | A+ | A- | A+ | A- | **A+** — **已拆分至 [TouchFish-DevTeam](https://github.com/agony1997/TouchFish-DevTeam)** |

---

## 綜合結論

這是一套**設計成熟度非常高**的 Claude Code skill 集合。核心設計決策都是正確的：

- 工作流型（非知識型）
- 英文指令 + 中文指南
- 按需載入降低 context 成本
- Sub-agent vs Agent Teams 精準分層
- 人機閘門保護關鍵決策

**最大亮點**是 dev-team v3.0 — 分離測試 + 三方 QA + scope enforcement + LLM-native 文件格式，這套設計解決了 agent 自我驗證不可靠的核心問題。

**最需要改善的**是跨 skill 的銜接契約和中斷恢復機制。目前 7 個 skill 各自封裝良好，但作為 pipeline 串接時缺少介面定義，使用者需要自己理解輸出格式是否相容下游輸入。

整體而言，作為 Claude Code skill marketplace 的作品，這套工具在架構設計、prompt engineering 和 context 管理上都處於**業界領先水準**。

---

## 建議改善優先順序

| 優先級 | 改善項目 | 影響範圍 | 預估工作量 |
|--------|---------|---------|-----------|
| P1 | 跨 Skill 交接契約（Handoff） | 全 pipeline | 中（每個 skill 加 Next Step + 介面定義） |
| P1 | 中斷恢復（Resume Protocol） | dev-team, md-to-code | 中（加 resume section + 狀態推斷邏輯） |
| P2 | ddd-core → dev-team 快速通道 | ddd-core, dev-team | 小（dev-team P1 加條件分支） |
| P2 | Prompt 變數空值 fallback | 所有 prompts/ | 小（每個變數加空值說明） |
| P3 | README 版本號修正 | README.md | 極小 |
| P3 | explorer 敏感檔案清單擴充 | explorer | 極小 |
| P3 | 成本估算指引 | dev-team, md-to-code | 小（加 cost note） |
