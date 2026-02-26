# TouchFish-Skills 技能審視報告

> 審視日期：2026-02-26
> 團隊配置：TL (Opus) + challenger x3 (Opus) + resolver x3 (Opus)
> 審視範圍：全部 7 個 Skill（ddd-core, git-nanny, spec-to-md, md-to-code, reviewer, explorer, dev-team）

---

## 一、跨技能系統性問題

以下 5 個問題影響多個技能，建議統一處理。

### S1. Context 炸彈（Spawn Prompt 嵌入完整文件內容）

| 項目 | 內容 |
|------|------|
| 影響技能 | spec-to-md, md-to-code |
| 問題描述 | Spawn prompt 以 `{tech_spec_content}` 等變數嵌入完整文件（500-1000 行），導致 teammate context window 被佔 30-50%。SKILL.md Principles 自己寫「Keep each document at reasonable length」，卻又要求「Teammate prompts must include full context」，自相矛盾。 |
| 建議方案 | 改為「寫入共享 context 檔案 + teammate 自行 Read」。Spawn prompt 只嵌入摘要 + 檔案路徑。對超過 ~200 行的文件，不嵌入全文。 |
| 實施成本 | Medium |
| 建議 | **修改** |

### S2. 技術棧硬編碼

| 項目 | 內容 |
|------|------|
| 影響技能 | ddd-core (Spring Boot + JPA + PostgreSQL), git-nanny (npm), spec-to-md (特定框架 API), dev-team (TypeScript 型別定義) |
| 問題描述 | 多個技能在 SKILL.md 和 reference 檔案中硬編碼特定技術棧，對非 Java/Node.js 專案使用者造成障礙。 |
| 建議方案 | (1) SKILL.md 加入技術棧確認步驟，(2) 模板改為通用結構 + 範例標記，(3) reference 中的具體命令改為多生態系範例或通用描述。 |
| 實施成本 | Low-Medium |
| 建議 | **修改** |

### S3. superpowers 依賴不一致

| 項目 | 內容 |
|------|------|
| 影響技能 | spec-to-md, md-to-code |
| 問題描述 | spec-to-md Step 3 以命令式語氣使用 `superpowers:brainstorming`（強制依賴），其他整合點標記為「Optional integration」。md-to-code 的 Step 2/Step 4 同樣把 superpowers 當必要方法論。若未安裝 superpowers，執行方式不明確。 |
| 建議方案 | 統一為 `> **Optional integration** — if superpowers plugin is installed, use superpowers:xxx.` 格式，並提供無 superpowers 時的替代做法。 |
| 實施成本 | Low |
| 建議 | **修改** |

### S4. On-demand 載入路徑策略不統一

| 項目 | 內容 |
|------|------|
| 影響技能 | ddd-core, git-nanny, explorer, dev-team |
| 問題描述 | Phase 0 (ddd-core) 使用 Glob pattern（`Glob **/ddd-core/**/references/ddd-theory.md`），Phase 1-4 使用相對路徑（`references/[template].md`）。各技能間載入方式混用。 |
| 建議方案 | 統一使用相對路徑 `references/xxx.md`（AI 執行 Skill 時已知 Skill 目錄位置，相對路徑更清楚）。 |
| 實施成本 | Low |
| 建議 | **修改** |

### S5. SKILL.md 缺版本號

| 項目 | 內容 |
|------|------|
| 影響技能 | 全部 7 個 |
| 問題描述 | 只有 GUIDE.zh-TW.md 頂部和 plugin.json 有版本號，SKILL.md 本身沒有版本標示，維護時容易混淆。 |
| 建議方案 | 可在 SKILL.md frontmatter 或頂部加入版本標記。 |
| 實施成本 | Low |
| 建議 | 可選修改（P3 等級） |

---

## 二、各技能審視結果

### 1. ddd-core

**總體評估**：四段式流程架構清晰，理論基礎紮實。主要問題是過度綁定 Java/Spring Boot，以及指令與模板之間的不一致。

#### P0 Critical

| # | 問題 | 影響檔案 | 方案 | 成本 | 建議 |
|---|------|---------|------|------|------|
| 1.1 | **Phase 3 技術棧硬綁 Spring Boot** — SKILL.md 第 100 行寫死 `Spring Boot + JPA + DDD standard layout`，sd-template.md 硬碼 `Spring Boot 3.x + JPA + PostgreSQL`。非 Java 專案完全無法適用。 | SKILL.md, sd-template.md | (1) Phase 3 新增「確認技術棧」步驟，(2) sd-template 改為 `[User's tech stack]` 佔位符，(3) package layout 改為範例標示。 | Low | **修改** |

#### P1 Major

| # | 問題 | 影響檔案 | 方案 | 成本 | 建議 |
|---|------|---------|------|------|------|
| 1.2 | **Phase 0 觸發條件不清** — 「User is unfamiliar with DDD」太模糊，AI 無法判斷何時觸發，且不清楚能否在其他 Phase 中途觸發。 | SKILL.md | 改為具體 trigger 條件列表（用戶明確問 DDD 概念 / 請求學習 / Phase 中途問術語），並聲明 Phase 0 為 non-blocking。 | Low | **修改** |
| 1.3 | **確認步驟過度摩擦** — Phase 1-4 每個都有 `Confirm [input] with user`，連續執行時多餘。 | SKILL.md | Phase 2-4 改為條件式：「if continuing from previous Phase, use its output directly; otherwise confirm with user」。 | Low | **修改** |
| 1.4 | **Event Sourcing 虛假宣傳** — frontmatter description 包含 `Event Sourcing`，但 SKILL.md 和 ddd-theory.md 都沒有 Event Sourcing 的內容。 | SKILL.md (frontmatter) | 從 description 和 keywords 移除 Event Sourcing。 | Low | **修改** |
| 1.5 | **Phase 間銜接缺乏驗證** — 沒有驗證前一階段產出是否完整的機制，不完整的 Event Storming 產出直接進入 SA 會有品質問題。 | SKILL.md | 各 Phase 入口加簡要的完整性檢查指引。 | Low | **修改** |
| 1.6 | **SA Template 缺少 NFR 指導** — sa-template.md 有 NFR 區塊，但 SKILL.md Phase 2 指令完全沒提到 NFR。 | SKILL.md | Phase 2 步驟中加入 NFR 處理指示。 | Low | **修改** |

#### P2 Minor

| # | 問題 | 影響檔案 | 方案 | 成本 | 建議 |
|---|------|---------|------|------|------|
| 1.7 | **載入路徑不一致** — Phase 0 用 Glob pattern，Phase 1-4 用相對路徑。 | SKILL.md | 統一為相對路徑 `references/xxx.md`。 | Low | **修改** |
| 1.8 | **sd-template 步驟不對齊** — 模板有第 8 節（錯誤處理）和第 9 節（ADR），但 Phase 3 步驟沒有對應指示。 | SKILL.md | Phase 3 補充步驟 8（錯誤處理策略）和 9（ADR 記錄）。 | Low | **修改** |
| 1.9 | **impl-plan 依賴圖格式過簡** — 純文字箭頭格式在 20+ 任務時無法閱讀。 | impl-plan-template.md | 改為 Mermaid graph 格式，加上大計畫分組提示。 | Low | **修改** |
| 1.10 | **ddd-theory.md 冗餘過長（596 行）** — Money class 重複出現三次，同時當教材和設計參考。 | ddd-theory.md | 去除重複內容。或拆分為概念速查 + 程式碼範例（可選，成本較高）。 | Medium | 可選修改 |
| 1.11 | **產出模板語言混用** — SKILL.md 指令英文，模板用中文欄位名。 | 各 template | 此為設計意圖（繁中產出），暫不修改。 | N/A | 不修改 |

#### P3 Cosmetic

| # | 問題 | 影響檔案 | 方案 | 成本 | 建議 |
|---|------|---------|------|------|------|
| 1.12 | **Factory 模式半殘** — Quick Reference 列出 Factory，但無範例和解釋。 | ddd-theory.md | 在 Quick Reference 加註 `(use static factory methods on Aggregate Root)` 即可。 | Low | 可選修改 |
| 1.13 | **缺少增量式 DDD 支援** — 整個流程假設 greenfield 專案，無 brownfield 重構指引。 | SKILL.md | 未來 feature request，不在當前修改範圍。 | N/A | 不修改 |
| 1.14 | **缺少版本化和變更追蹤** — 產出文件無版本欄位。 | 各 template | 未來 feature request。 | N/A | 不修改 |

---

### 2. git-nanny

**總體評估**：所有技能中設計最紮實的一個。安全規則清晰，Intent Detection 分工明確。問題主要集中在 reference 檔案的細節矛盾。

#### P0 Critical

| # | 問題 | 影響檔案 | 方案 | 成本 | 建議 |
|---|------|---------|------|------|------|
| 2.1 | **perf 版本號矛盾** — release-changelog.md 說 `perf → MINOR`，conventional-commits.md 的 Auto-Versioning 只列 feat/fix/BREAKING CHANGE，兩個文件矛盾。SemVer 官方 perf 應屬 PATCH。 | release-changelog.md, conventional-commits.md | 統一：(1) release-changelog.md 將 perf 改為 PATCH，(2) conventional-commits.md 擴充 Auto-Versioning 列出所有 commit type 對應的 bump。 | Low | **修改** |
| 2.2 | **force-with-lease 灰色地帶** — Safety Rules 只禁 `--force`，但 pr-template.md 有 `--force-with-lease`。雖然後者較安全但仍是覆蓋遠端歷史。 | SKILL.md, pr-template.md | Safety Rules 明確將 `--force-with-lease` 也納入需使用者確認的清單。pr-template.md 加確認步驟。 | Low | **修改** |

#### P1 Major

| # | 問題 | 影響檔案 | 方案 | 成本 | 建議 |
|---|------|---------|------|------|------|
| 2.3 | **Intent Detection 中文關鍵字過窄** — 只列了 commit/提交/PR/release 等，缺「合併請求」「版本號」「標籤」「代碼審查」等常見中文。複合場景（commit + PR 串接）無處理。 | SKILL.md | 擴充中文關鍵字清單，加入多 Section 串接指引。 | Low | **修改** |
| 2.4 | **Section 3 缺輸出格式** — Branch Strategy 的步驟沒有明確輸出格式要求，是四個 Section 中最弱的。 | SKILL.md | 為 Section 3 設計具體的輸出格式規範（推薦策略、理由、命名規範範例）。 | Low | **修改** |
| 2.5 | **release-changelog.md 硬編碼 npm** — `npm test && npm run build`、`npm version`、semantic-release 區塊全是 npm 專屬。 | release-changelog.md | 改為通用框架：提示使用者確認驗證命令，或提供多生態系範例。 | Medium | **修改** |
| 2.6 | **Section 3 缺少「分析現有專案」的步驟** — 說「Assess project context」但沒說明如何評估（git tag 歷史、CI 配置等）。 | SKILL.md | 加入具體分析步驟（`git tag -l`、檢查 CI 配置檔、README 等）。 | Low | **修改** |
| 2.7 | **Hotfix 流程假設 Git Flow 但放在通用位置** — release-changelog.md 的 Hotfix 明確使用 Git Flow 模式，但放在通用 Release 區塊。 | release-changelog.md | 移至 branch-strategies.md 的 Git Flow 部分，或加條件標記。 | Low | **修改** |
| 2.8 | **PR Review 與 reviewer 技能職責重疊** — pr-template.md 包含完整 Code Review Checklist，但專案已有獨立的 reviewer 技能。 | pr-template.md | 簡化 pr-template 中的 review 內容為基本 checklist，深度審查指向 reviewer 技能。 | Low | **修改** |

#### P2 Minor

| # | 問題 | 影響檔案 | 方案 | 成本 | 建議 |
|---|------|---------|------|------|------|
| 2.9 | **Quick Reference 含被 Safety Rules 禁止的命令** — `git commit --amend` 和 `git reset --soft HEAD~1` 列在 Quick Reference，但 Safety Rules 禁止未經確認的 amend。 | SKILL.md | 在這些命令旁加 `# Confirm first` 標記。 | Low | **修改** |
| 2.10 | **pr-template.md push 指令未加確認** — Step 4 有 `git push -u origin` 但未提醒先確認。 | pr-template.md | 加一句 `# First confirm with user before pushing`。 | Low | **修改** |

#### P3 Cosmetic

| # | 問題 | 影響檔案 | 方案 | 成本 | 建議 |
|---|------|---------|------|------|------|
| 2.11 | **Co-Authored-By 硬編碼模型版本** — 寫死 `Claude Opus 4.6`，模型更新後不準確。 | conventional-commits.md | 改為 `Claude <noreply@anthropic.com>`，移除版本號。 | Low | **修改** |
| 2.12 | **Safety Rules 與 Claude Code 內建行為高度重複** — ~24 行（SKILL.md ~17%）是純冗餘。 | SKILL.md | 精簡為「Follow Claude Code's built-in git safety rules」+ git-nanny 特有補充。 | Low | 可選修改 |
| 2.13 | **branch-strategies.md Hotfix Flow 有 push 未顯示確認** — reference 中直接有 `git push origin main develop --tags`。 | branch-strategies.md | 加一行注釋 `# Confirm with user before pushing`。 | Low | **修改** |

---

### 3. spec-to-md

**總體評估**：整個技能集中設計最複雜的一個（3 subagents + Agent Teams + 交叉校對）。架構有創意，但複雜度帶來更多潛在失敗點。

#### P0 Critical

| # | 問題 | 影響檔案 | 方案 | 成本 | 建議 |
|---|------|---------|------|------|------|
| 3.1 | **Spawn Prompt Context 過載** — `{tech_spec_content}` 嵌入完整 01_技術規格.md（500-1000 行），與 Principles「reasonable length」矛盾。 | SKILL.md, backend-spec.md, frontend-spec.md | 改為「寫入共享 context 檔案 `{output_dir}/context/spec-context.md` + teammate 自行 Read」。Principle 改為「reference shared context file, avoid embedding large documents」。 | Medium | **修改** |

#### P1 Major

| # | 問題 | 影響檔案 | 方案 | 成本 | 建議 |
|---|------|---------|------|------|------|
| 3.2 | **backend-spec / frontend-spec 競態條件** — backend-spec 完成後 SendMessage 給 frontend-spec，但 frontend-spec 可能已完成或未啟動。 | SKILL.md, backend-spec.md, frontend-spec.md | 改為 TL 協調模式：兩個 teammate 都向 TL 報告完成，TL 提取 API 清單發給 frontend-spec 驗證。 | Low | **修改** |
| 3.3 | **template-structure.md 高度客製化** — 包含 `ApiRouteProcessor`、`LogType.BIZ_XXX`、`@ActivateRequestContext` 等特定框架 API，作為通用插件發布有誤導性。 | template-structure.md, backend-spec.md | 方案 A（推薦）：模板泛化為通用結構（handler/controller/processor 等通稱），移除特定框架 class 名稱。 | Medium | **修改** |
| 3.4 | **superpowers 強制依賴無 fallback** — Step 3 用命令式 `Use superpowers:brainstorming`，不像 ddd-core 標記「Optional integration」。 | SKILL.md | 統一改為 `> Optional integration — if superpowers plugin is installed, use superpowers:brainstorming.` | Low | **修改** |

#### P2 Minor

| # | 問題 | 影響檔案 | 方案 | 成本 | 建議 |
|---|------|---------|------|------|------|
| 3.5 | **Consistency Check 太抽象** — Step 5 驗證只說「match on API endpoints, Entity fields, component names」，缺具體 checklist。 | SKILL.md | 加入 7 項具體 checklist（每個 API endpoint 有對應實作、field names 匹配、Types 匹配等）。 | Low | **修改** |
| 3.6 | **GUIDE 與 SKILL.md 不一致** — GUIDE 說「可以只產出部分功能」，但 SKILL.md 是線性流程，無分支邏輯。 | GUIDE.zh-TW.md | 修改 GUIDE 使之符合 SKILL.md（建議產出全套以確保一致性）。 | Low | **修改** |
| 3.7 | **Spawn prompt 缺模板格式指示** — backend-spec.md 和 frontend-spec.md 沒有指示 teammate 要遵照 template-structure.md 格式。 | prompts/backend-spec.md, prompts/frontend-spec.md | 加一行指向 template-structure.md 的指示。 | Low | **修改** |
| 3.8 | **Step 5 關閉流程不完整** — 沒有說明 teammate 拒絕 shutdown 時怎麼辦。 | SKILL.md | 加一句：「If teammate rejects shutdown, SendMessage asking them to wrap up current work first.」 | Low | **修改** |

#### P3 Cosmetic

| # | 問題 | 影響檔案 | 方案 | 成本 | 建議 |
|---|------|---------|------|------|------|
| 3.9 | **TeamCreate team_name 格式不明** — 沒說 feature-name 的格式限制。 | SKILL.md | 加說明 `(lowercase, no spaces, use hyphens)`。 | Low | 可選修改 |
| 3.10 | **Step 2 並行 Subagent 無失敗處理** — 任一 Agent 失敗可能導致流程卡住。 | SKILL.md | 加一行 fallback：「If any agent fails, AskUserQuestion to verify file path before retrying.」 | Low | 可選修改 |

---

### 4. md-to-code

**總體評估**：流程設計嚴謹，Gate 機制是亮點。與 spec-to-md 共享 context 炸彈問題，並有自己獨特的缺陷（code-reviewer 未定義、缺分支邏輯）。

#### P1 Major

| # | 問題 | 影響檔案 | 方案 | 成本 | 建議 |
|---|------|---------|------|------|------|
| 4.1 | **Context window 爆炸** — Spawn prompt 嵌入 tech spec + backend impl + standards + components，可能達 3000-5000 tokens。 | SKILL.md | 加入 context size 管理指引：超 ~200 行的文件改嵌摘要 + 檔案路徑讓 teammate 按需 Read。 | Low | **修改** |
| 4.2 | **Step 4 code-reviewer 未定義** — 「via code-reviewer agent」但沒說明是 reviewer 技能還是另一個 subagent。 | SKILL.md | 改為 `use reviewer skill if available`，明確引用。 | Low | **修改** |
| 4.3 | **prompt.md 位置不明** — Step 1 說 `Read prompt.md` 但沒說在哪裡找。 | SKILL.md | 加入搜尋邏輯：用戶提供路徑 → Glob `**/prompt.md` → 多結果時 AskUserQuestion。 | Low | **修改** |
| 4.4 | **{variables} 無映射表** — AI 不知道每個 spawn 模板變數對應 Step 1 的哪個產出。 | SKILL.md | 加入明確的變數映射表（5 個變數各自對應 Agent A/B/C 的哪個輸出）。 | Low | **修改** |
| 4.5 | **「只做後端」無分支邏輯** — GUIDE 承諾此功能但 SKILL.md 無實作。 | SKILL.md | 加入 scope detection：如果 prompt.md 只有 02（無 03），跳過 frontend，只 spawn backend-dev。 | Low | **修改** |
| 4.6 | **錯誤處理完全缺失** — Subagent 失敗、teammate 卡住、編譯失敗都無處理。 | SKILL.md | 在 Principles 區段集中加入 error handling 指引（retry、take over、present to user）。 | Low | **修改** |
| 4.7 | **Subagent 分工重疊風險** — Agent B（探索既有程式碼）和 Agent C（讀實作文件）產出可能重疊，無衝突解決規則。 | SKILL.md | 加入 conflict resolution：Naming/style → Agent B（專案現實）優先；Implementation steps → Agent C（spec docs）為準。 | Low | **修改** |
| 4.8 | **與 spec-to-md 緊耦合** — 依賴特定輸出格式（prompt.md + 01/02/03 檔名），但無版本相容性聲明。 | SKILL.md | 在 Principles 加入預期格式聲明 + 適應機制（讀 prompt.md 的 navigation section 取得實際路徑）。 | Low | **修改** |

#### P2 Minor

| # | 問題 | 影響檔案 | 方案 | 成本 | 建議 |
|---|------|---------|------|------|------|
| 4.9 | **superpowers 軟硬依賴不清** — Step 2 語氣是命令式，但 GUIDE 列為可選。 | SKILL.md | 明確標記 `> Optional integration`。 | Low | **修改** |
| 4.10 | **完成報告模板語言不一致** — 模板中文，但指示說「Write in the same language as spec docs」。 | completion-report-template.md | 模板改為英文（與其他 reference 一致），指示保持 follow spec docs language。 | Low | **修改** |
| 4.11 | **Agent B 用 Opus 可能不必要** — Agent B 做的是 Explore 型任務，Sonnet 更經濟。 | SKILL.md | 考慮 Agent B 改為 `model: "sonnet"`。 | Low | 可選修改 |

#### P3 Cosmetic

| # | 問題 | 影響檔案 | 方案 | 成本 | 建議 |
|---|------|---------|------|------|------|
| 4.12 | **半步編號風格獨特** — 1, 1.5, 2, 3, 3.5, 4, 5 在其他技能中未見。 | SKILL.md | 不修改——半步清楚表達「gate ≠ step」語意。 | N/A | 不修改 |
| 4.13 | **TeamCreate/TeamDelete API 存疑** — 質疑者認為不存在，但確認為真實 API。 | — | 不修改。 | N/A | 不修改 |
| 4.14 | **缺 Agent Metrics** — dev-team 有完整 Metrics，md-to-code 沒有。 | completion-report-template.md | 可選加入簡化版 Metrics 表格（Agent/Role/Tasks/Duration）。 | Low | 可選修改 |
| 4.15 | **無版本控制指引** — 屬 git-nanny 職責，md-to-code 不應重複。 | — | 不修改。可選在 Step 5 加一行提示用 git-nanny。 | N/A | 不修改 |
| 4.16 | **測試策略缺失** — 由 spec docs 決定，符合 Principles「do not add undefined features」。 | — | 不修改。 | N/A | 不修改 |

---

### 5. reviewer

**總體評估**：所有技能中最精簡的（65 行），設計哲學清晰（skill = 工作流，規範 = 用戶維護）。問題主要集中在實用性功能缺口。

#### P1 Major

| # | 問題 | 影響檔案 | 方案 | 成本 | 建議 |
|---|------|---------|------|------|------|
| 5.1 | **Progressive disclosure 矛盾** — Step 1 說「only load what is needed」，Notes 說「Multiple standards files are all loaded」，直接矛盾。 | SKILL.md | 移除「progressive disclosure」措辭。Standards 是審查依據，必須全部載入。改為「Load all located standards files — they are the review source of truth.」 | Low | **修改** |
| 5.2 | **沒有並行能力** — Step 3 序列審查所有檔案，大範圍審查時低效。 | SKILL.md | 加入：>5 檔案時 group by module/layer，launch parallel Task agents (Sonnet)。<=5 檔案直接審查。 | Low | **修改** |
| 5.3 | **沒有增量審查** — scope 有「recently modified」選項但無實作方式。 | SKILL.md | Step 2 加入 scope resolution：「recently modified」→ `git diff --name-only HEAD~1` 或 `--staged`。 | Low | **修改** |
| 5.4 | **審查結果沒有持久化** — 報告只 present 在對話中，下次對話消失。 | SKILL.md | Step 4 加：「Ask user: save report to file? If yes, write to `.standards/reviews/YYYY-MM-DD-<scope>.md`」。 | Low | **修改** |
| 5.5 | **缺嚴重度分級** — 不合規項目表格無 Severity 欄，命名不一致和架構違規混在一起。 | review-report-template.md | 表格加 Severity 欄（Critical / Major / Minor），Summary 加分級統計。 | Low | **修改** |
| 5.6 | **無自動修正橋接** — 審查後最常見需求是「幫我修」，但無銜接。 | SKILL.md | Step 4 加 fix workflow：Minor/Major 直接修；Critical 先說明計畫再確認；修完後 re-run 驗證。 | Low | **修改** |
| 5.7 | **大型專案效能未考慮** — 500+ 檔案無法逐一讀取。 | SKILL.md | 加 scale guard：>20 檔案時建議縮小範圍或用 parallel subagents + 抽樣審查。 | Low | **修改** |

#### P2 Minor

| # | 問題 | 影響檔案 | 方案 | 成本 | 建議 |
|---|------|---------|------|------|------|
| 5.8 | **Review dimensions 二義性** — 「adapt to actual standards」不清楚是追加還是替代。 | SKILL.md | 改為「use standards' own section structure if present; fall back to generic dimensions if not」。 | Low | **修改** |
| 5.9 | **review-report-template 一檔兩用** — Step 1 引用為 setup guide，Step 4 引用為 report template。 | SKILL.md | 引用時指明區段 `§ "Project Setup Guide"` 和 `§ "Review Report"`。 | Low | **修改** |
| 5.10 | **Standards 衝突處理缺失** — 多份 standards 可能矛盾。 | SKILL.md | Notes 加入：「If standards files contain conflicting rules, flag in report and ask user to clarify.」 | Low | **修改** |
| 5.11 | **Step 1 搜尋方式不明確** — 沒說用 Glob 還是直接 Read，無跨平台考量。 | SKILL.md | 明確指定 Glob 搜尋。 | Low | **修改** |
| 5.12 | **SKILL.md 和 GUIDE 語氣不一致** — SKILL.md 說「suggest」，GUIDE 說「會提示你建立」。 | GUIDE.zh-TW.md | 統一為「啟用時會自動搜尋規範文件，找不到時會詢問位置」。 | Low | **修改** |

#### P3 Cosmetic

| # | 問題 | 影響檔案 | 方案 | 成本 | 建議 |
|---|------|---------|------|------|------|
| 5.13 | **功能過於單薄** — 65 行，附加價值低。 | — | 不修改——設計意圖如此。透過其他功能增強（5.2-5.7）自然提升價值。 | N/A | 不修改 |
| 5.14 | **是否應作為獨立技能** — 有獨立使用場景（任何時候都可要求 code review），維持獨立。 | — | 不修改。 | N/A | 不修改 |

---

### 6. explorer

**總體評估**：設計理念最獨特（Phase 0 智慧偵測 + 並行探索 + 審視迴圈）。主要弱點是大型專案的 context 控制、偵測邏輯覆蓋不足、安全指示缺失。

#### P0 Critical

| # | 問題 | 影響檔案 | 方案 | 成本 | 建議 |
|---|------|---------|------|------|------|
| 6.1 | **偵測邏輯過於簡化** — 只有三個分支（multi-module、前後端分離、簡易單體），microservices、library/SDK、data/ML 專案都會誤判為簡易單體。 | SKILL.md | 改為五個 primary signals（+ microservices + library/SDK）+ secondary signals + 使用者確認。加 cap 10 上限。 | Low | **修改** |
| 6.2 | **敏感檔案洩漏** — explore-subagent.md 第 43 行明確列出 `.env` 作為搜尋目標，PROJECT_MAP.md 可能洩漏敏感資訊。 | explore-subagent.md, SKILL.md | (1) 加 SAFETY RULES：不報告 .env/credentials 實際值，只報告存在和用途。(2) Phase 3 寫入前加過濾步驟。(3) 排除清單加 `.env.local, *.pem, *.key`。 | Low | **修改** |

#### P1 Major

| # | 問題 | 影響檔案 | 方案 | 成本 | 建議 |
|---|------|---------|------|------|------|
| 6.3 | **審視迴圈無終止條件** — `Repeat until ALL pass` 無上限，專案本身的真實衝突會導致無限迴圈。 | SKILL.md | 加 `max 3 iterations`，三分類（Factual discrepancy → sub-agent / Contextual uncertainty → AskUser / True inconsistency → Known Issue），達上限後未解決項變 Known Issue。 | Low | **修改** |
| 6.4 | **Sub-agent 數量爆炸** — 50 個 packages 就是 50+ sub-agents，無上限或分批策略。 | SKILL.md, explore-subagent.md | cap 10 + 分組策略（按 naming prefix 或目錄結構分組），呈現給使用者確認。 | Low | **修改** |
| 6.5 | **Sub-agent 失敗處理缺失** — 可能派 5-10 個 sub-agents，至少一個失敗機率不低。 | SKILL.md | 加 retry once → Leader 手動探索 fallback → partial success 繼續進行。 | Low | **修改** |
| 6.6 | **Sub-agent 報告長度未控制** — 複雜模組可能產出 500+ 行報告，Leader context 爆炸。 | explore-subagent.md | 加 length guideline：「keep report under 500 lines, list top 10 important items per section, summarize rest.」 | Low | **修改** |

#### P2 Minor

| # | 問題 | 影響檔案 | 方案 | 成本 | 建議 |
|---|------|---------|------|------|------|
| 6.7 | **探索深度不清** — 只有排除清單，無正面深度定義。 | explore-subagent.md | 加 EXPLORATION RULES：4 層深度、500KB 大小限制、source files 只讀結構不讀全文。 | Low | **修改** |
| 6.8 | **缺少二進位/大檔案排除** — sub-agent 可能嘗試 Read 10MB SQL dump。 | explore-subagent.md | 加排除清單：binary files、generated files、data dumps（各有具體副檔名列表）。 | Low | **修改** |
| 6.9 | **唯讀安全指示缺失** — 雖然 `subagent_type: "Explore"` 沒有 Edit 權限，但明確聲明是 defense in depth。 | explore-subagent.md | 加「READ-ONLY exploration. Do NOT create, modify, or delete any files.」 | Low | **修改** |
| 6.10 | **PROJECT_MAP.md 模板偏向 Java + Vue** — `Controller -> Service -> Repository` 等特定結構。 | project-map-template.md | 改為通用描述（`<describe actual layering pattern>`），範例通用化。 | Low | **修改** |
| 6.11 | **「Clear doubt」vs「Uncertain」分界線不明** — AI 不確定何時派 sub-agent、何時問使用者。 | SKILL.md | 已併入 6.3 的方案：改用 Factual discrepancy / Contextual uncertainty / True inconsistency 三分類。 | — | 見 6.3 |

#### P3 Cosmetic

| # | 問題 | 影響檔案 | 方案 | 成本 | 建議 |
|---|------|---------|------|------|------|
| 6.12 | **Sub-agent 用 Sonnet 是否合適** — 設計合理（讀檔+報告是 Sonnet 甜蜜點，判斷在 Phase 2 Opus Leader 做）。 | — | 不修改。 | N/A | 不修改 |
| 6.13 | **無增量更新機制** — 每次完整重跑。 | — | 不修改——定位為一次性偵察，增量更新是未來 feature。 | N/A | 不修改 |
| 6.14 | **Phase 編號從 0 開始** — 有語義（偵測/準備），ddd-core 也是。 | — | 不修改。 | N/A | 不修改 |
| 6.15 | **GUIDE 時間估計過於樂觀** — 大型 monorepo 可能遠超 5 分鐘。 | GUIDE.zh-TW.md | 改為「取決於專案大小和複雜度」。 | Low | 可選修改 |

---

### 7. dev-team

**總體評估**：最有野心、最複雜的技能。任務池架構 + File Scope + QA pipeline + Metrics 展現深思熟慮。核心問題多為設計取捨而非缺陷，真正需要的是操作指引精細化。

#### P1 Major

| # | 問題 | 影響檔案 | 方案 | 成本 | 建議 |
|---|------|---------|------|------|------|
| 7.1 | **TL 單點瓶頸** — 所有通訊、文件維護、QA 管理集中在 TL。多 worker 同時完成時串行處理。 | SKILL.md | 加入 batch processing 指引：「If multiple workers complete near-simultaneously, TL MAY batch-process: update TRACE for all, then spawn QA sub-agents in parallel.」 | Low | **修改** |
| 7.2 | **Agent Metrics「tracked」資料空洞** — Workers/challenger 的 Token 欄是 `—`，只有 QA 有 exact 數據。Cost Breakdown 對主要角色基本空白。 | delivery-report-template.md | 調整模板：TL/worker/challenger 的 Token 和 Cost 改為 `n/a`，Cost Breakdown 更名為「QA Cost Breakdown (exact data only)」。誠實承認限制。 | Low | **修改** |
| 7.3 | **QA sub-agent 審查深度不足** — 一次性 Sonnet sub-agent 需完成 12 項 checklist，但 context 可能不足。 | SKILL.md | 擴充 QA spawn context：除原有的 file_list/contract/standards 外，加入 task description with acceptance criteria + relevant existing code context。 | Low | **修改** |
| 7.4 | **Process Log / TRACE 大量重疊** — 每次狀態變更 TL 要更新 2-3 個文件，加重瓶頸。 | SKILL.md, process-log-template.md | PROCESS_LOG 精簡為只記非常規事件（decisions, issues, contract amendments, team changes）；常規狀態轉換（task-completed, review-pass）由 TRACE 涵蓋即可。 | Low | **修改** |
| 7.5 | **Multi-spec 共用檔案衝突** — parallel 模式下不同 spec 的任務可能衝突，處理不明確。 | SKILL.md | 加入：TL MUST identify shared files across specs → assign to same worker OR serialize → 共用檔案 >3 個時建議 Sequential 模式。 | Low | **修改** |
| 7.6 | **Phase 5 契約驗證時機太晚** — 所有任務完成後才全局驗證，發現偏差修復成本高。 | SKILL.md | Phase 4 加早期預警：QA 偵測到 contract mismatch 時，TL 立即廣播給所有受影響 workers。 | Low | **修改** |
| 7.7 | **Worker 自取任務的 race condition** — 兩個 worker 可能同時看到同一 pending 任務並嘗試 claim。 | prompts/worker.md | 加入明確的驗證步驟：「TaskUpdate → set owner → TaskGet → confirm owner is yourself. If not, TaskList again.」 | Low | **修改** |

#### P2 Minor

| # | 問題 | 影響檔案 | 方案 | 成本 | 建議 |
|---|------|---------|------|------|------|
| 7.8 | **Challenger「proactively monitor」措辭不當** — Agent Teams 中 teammate 是消息驅動的，無法真正主動輪詢。 | prompts/challenger.md | 改為「When activated by TL, use TaskList to understand current progress before reviewing.」 | Low | **修改** |
| 7.9 | **Worker 數量計算邊界問題** — 極端情況（2pt total 要 2 workers、30pt total 超上限）未處理。 | SKILL.md | 加入 edge cases：`<= 3 pts → 1 worker may suffice`、`single L task → prefer splitting`。 | Low | **修改** |
| 7.10 | **STOP RULE 邊界模糊** — TL 在 acknowledgment 中夾帶指示時 worker 可能忽略。 | prompts/worker.md | 加例外說明：「if acknowledgment ALSO contains instruction, treat as instruction and respond.」 | Low | **修改** |
| 7.11 | **TRACE 狀態機不完整** — `fixed` 後再 QA fail 的循環無定義，一個需求對應多個任務無法表示。 | trace-template.md | 表格加 Fix Task 關聯欄位，狀態流註解更新。 | Low | **修改** |
| 7.12 | **API Contract Template 硬編碼 TypeScript** — 用 TS 語法定義 Shared Types，非 TS 專案不適用。 | api-contract-template.md | 改為語言無關：「Use the project's primary language (TypeScript, Java, Go, etc.)」 | Low | **修改** |
| 7.13 | **Phase 2 API Contract 對非 API 專案不適用** — 純後端 batch job、CLI 工具等不涉及 API。 | SKILL.md | 加條件：「If requirements contain no API interactions: skip Phase 2, note N/A in TRACE.」 | Low | **修改** |
| 7.14 | **複雜度評分 S/M/L 標準缺失** — 沒有定義 S/M/L 的判斷標準。 | SKILL.md | 加簡要標準（S: 單檔修改 / M: 2-3 檔跨層 / L: 4+ 檔或核心邏輯變更）。 | Low | **修改** |
| 7.15 | **Worker prompt 的 context 傳遞方式不一致** — 用路徑而非嵌入內容，但 worker 不知道要主動 Read。 | prompts/worker.md | 在 context 區段加指示：「Read these files at the start of your work session.」 | Low | **修改** |
| 7.16 | **Phase 0 explorer 偵測方法不明** — 「Check if explorer skill is available」但沒具體偵測方式。 | SKILL.md | 加入具體偵測邏輯（嘗試 Glob 找 explorer SKILL.md，找到即可用）。 | Low | **修改** |
| 7.17 | **Metrics 定價硬編碼** — Opus $15/$75、Sonnet $3/$15 寫死在 SKILL.md，定價可能變動。 | SKILL.md | 抽到 references/ 中的獨立文件，或標記為「pricing as of 2026-02」。 | Low | **修改** |
| 7.18 | **Challenger METRICS 計數不明確** — 「reviews={count of checkpoints you reviewed}」定義模糊。 | prompts/challenger.md | 改為 `reviews={checkpoints reviewed} | challenges={CHALLENGE count} | concerns={CONCERN count}`。 | Low | **修改** |

#### P3 Cosmetic

| # | 問題 | 影響檔案 | 方案 | 成本 | 建議 |
|---|------|---------|------|------|------|
| 7.19 | **File Scope 無技術強制** — 只有自然語言約束，依賴 AI 遵從度。 | prompts/worker.md | 可選加強警告語氣：「VIOLATION WARNING: Modifying files outside ALLOWED list WILL cause QA failure.」 | Low | 可選修改 |
| 7.20 | **PROCESS_LOG 事件類型不完整** — 缺 worker-spawned、scope-adjusted、challenge-raised 等。 | process-log-template.md | 已由 7.4 的精簡方案覆蓋（只記非常規事件）。 | — | 見 7.4 |
| 7.21 | **Date prefix 時區問題** — 未指定使用哪個時區。 | SKILL.md | 可加註 `(local timezone)`。 | Low | 可選修改 |
| 7.22 | **GUIDE 版本比較表不必要** — v1.3 vs v2.0 對新使用者無價值。 | GUIDE.zh-TW.md | 可移除或折疊。 | Low | 可選修改 |
| 7.23 | **METRICS format fragile parsing** — Worker shutdown 時回傳的 `METRICS:` 格式依賴精確字串。 | — | 架構限制，暫不修改。Agent 通常能遵守格式。 | N/A | 不修改 |
| 7.24 | **Worker crash/unresponsive 無處理** — context window 溢出或工具失敗時 pending 任務無人接手。 | SKILL.md | 可在 edge case handling 加入 reassign + spawn replacement 機制。 | Low | 可選修改 |

---

## 三、統計摘要

### 按優先級

| 優先級 | 數量 | 說明 |
|--------|------|------|
| P0 Critical | **6** | ddd-core(1), git-nanny(2), spec-to-md(1), explorer(2) |
| P1 Major | **42** | ddd-core(5), git-nanny(5), spec-to-md(3), md-to-code(8), reviewer(7), explorer(4), dev-team(7), 跨技能(3) |
| P2 Minor | **30** | ddd-core(3), git-nanny(2), spec-to-md(4), md-to-code(2), reviewer(5), explorer(4), dev-team(10) |
| P3 Cosmetic | **21** | ddd-core(3), git-nanny(3), spec-to-md(2), md-to-code(5), reviewer(2), explorer(4), dev-team(6), 跨技能(1) |
| **合計** | **~99** | |

### 按技能

| 技能 | 總 findings | 建議修改 | 不修改 / 可選 |
|------|------------|---------|--------------|
| ddd-core | 14 | 9 | 5 |
| git-nanny | 13 | 11 | 2 |
| spec-to-md | 10 | 8 | 2 |
| md-to-code | 16 | 10 | 6 |
| reviewer | 14 | 10 | 4 |
| explorer | 15 | 10 | 5 |
| dev-team | 24 | 18 | 6 |
| 跨技能 | 5 | 4 | 1 |
| **合計** | **~111** | **~80** | **~31** |

### 按建議行動

| 行動 | 數量 |
|------|------|
| **修改**（建議或必修） | ~80 |
| **可選修改** | ~12 |
| **不修改** | ~19 |

### 實施成本分佈

| 成本 | 數量 |
|------|------|
| Low（文字修改） | ~90% |
| Medium（需重新設計 context 傳遞或模板泛化） | ~10% |
| High | 0 |

---

## 四、建議實施順序

### 第一波：P0 必修（6 項）

1. **ddd-core** — Phase 3 加技術棧確認步驟 + sd-template 泛化
2. **git-nanny** — perf 版本號統一為 PATCH
3. **git-nanny** — force-with-lease 納入確認清單
4. **spec-to-md** — Spawn Prompt 改為檔案引用 + 摘要
5. **explorer** — 偵測邏輯擴充（+ microservices, library/SDK, cap 10）
6. **explorer** — 加 SAFETY RULES（敏感檔案 + 唯讀）

### 第二波：跨技能系統性修正

7. 所有技能的 superpowers 整合點統一為 Optional integration
8. ddd-core 載入路徑統一為相對路徑
9. md-to-code context size 管理指引

### 第三波：P1 各技能修正

10. 按上述各技能 P1 列表依序處理

### 第四波：P2/P3 改善

11. 按優先級依序處理
