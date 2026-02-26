# TouchFish-Skills 技能修復報告

> 修復日期：2026-02-26
> Commit：`554181b`
> 依據：`docs/skill-review-report.md`
> 範圍：全部 7 個 Skill（ddd-core, git-nanny, spec-to-md, md-to-code, reviewer, explorer, dev-team）
> 統計：34 個檔案修改，+579 / -429 行

---

## 一、跨技能修復

| # | 狀態 | 項目 | 修復內容 |
|---|------|------|---------|
| S1 | ✅ | Context 炸彈 | spec-to-md / md-to-code spawn prompts 改為共享 context 檔案 + teammate 自行 Read |
| S2 | ✅ | 技術棧硬編碼 | ddd-core sd-template、dev-team api-contract-template 改為語言/框架無關佔位符 |
| S3 | ✅ | superpowers 依賴不一致 | spec-to-md / md-to-code 所有整合點統一改為 `> Optional integration` + fallback |
| S4 | ✅ | On-demand 載入路徑不統一 | ddd-core / git-nanny / explorer / dev-team 的 Glob pattern 全改為相對路徑 |
| S5 | ✅ | SKILL.md 缺版本號 | 全部 7 個 SKILL.md 頂部加入 `<!-- version: 1.2.0 -->` |

---

## 二、各技能修復明細

### 1. ddd-core

**修改檔案**：SKILL.md、sd-template.md、impl-plan-template.md、ddd-theory.md

| # | 狀態 | 項目 | 修復內容 |
|---|------|------|---------|
| 1.1 | ✅ | Phase 3 技術棧硬綁 | Phase 3 新增「Confirm technology stack with user」步驟；sd-template 的 `Spring Boot 3.x + JPA + PostgreSQL` 改為 `[User's confirmed tech stack]`，package layout 改為通用結構加範例標記 |
| 1.2 | ✅ | Phase 0 觸發條件 | 改為 3 個具體觸發條件（用戶問概念 / 請求學習 / Phase 中途問術語），聲明為 Non-Blocking |
| 1.3 | ✅ | 確認步驟過度摩擦 | Phase 2-4 入口改為條件式確認：「If continuing from previous Phase, use its output directly」 |
| 1.4 | ✅ | Event Sourcing 虛假宣傳 | frontmatter description 和 keywords 移除 `Event Sourcing` |
| 1.5 | ✅ | Phase 間銜接缺驗證 | Phase 2/3/4 入口加 Entry check，驗證前一階段產出完整性 |
| 1.6 | ✅ | SA Template 缺 NFR | Phase 2 新增步驟 6「Identify Non-Functional Requirements (NFR)」 |
| 1.7 | ✅ | 載入路徑不一致（S4） | Phase 0 Glob pattern 改為相對路徑 `references/ddd-theory.md` |
| 1.8 | ✅ | sd-template 步驟不對齊 | Phase 3 補充步驟 9（error handling）和步驟 10（ADR）|
| 1.9 | ✅ | impl-plan 依賴圖格式 | impl-plan-template.md 依賴圖改為 Mermaid graph 格式，加 20+ 任務分組提示 |
| 1.10 | ✅ | ddd-theory.md 冗餘 | 移除 Shared Kernel 中重複的 Money class 完整範例，改為引用 Value Object 章節 |
| 1.11 | ⏭ | 產出模板語言混用 | 設計意圖（繁中產出），不修改 |
| 1.12 | ✅ | Factory 模式半殘 | Quick Reference 的 Factory 加註 `(use static factory methods on Aggregate Root)` |
| 1.13 | ⏭ | 缺增量式 DDD | 未來 feature request，不在當前範圍 |
| 1.14 | ⏭ | 缺版本化追蹤 | 未來 feature request，不在當前範圍 |

---

### 2. git-nanny

**修改檔案**：SKILL.md、release-changelog.md、conventional-commits.md、pr-template.md、branch-strategies.md

| # | 狀態 | 項目 | 修復內容 |
|---|------|------|---------|
| 2.1 | ✅ | perf 版本號矛盾 | release-changelog.md 的 perf 改為 PATCH；conventional-commits.md Auto-Versioning 擴充為完整 11 種 commit type 對應表 |
| 2.2 | ✅ | force-with-lease 灰色地帶 | SKILL.md Safety Rules 將 `--force-with-lease` 納入需確認清單；pr-template.md 加確認提醒 |
| 2.3 | ✅ | Intent Detection 中文關鍵字 | 新增「合併請求」「代碼審查」「版本號」「標籤」「發版」等 10+ 中文關鍵字 + 多 Section 串接指引 |
| 2.4 | ✅ | Section 3 缺輸出格式 | 新增具體分析步驟（`git tag -l`、CI config、`git branch -a`）+ 推薦結果格式模板 |
| 2.5 | ✅ | release-changelog 硬編碼 npm | 改為多生態系範例（npm、Maven、Gradle、Cargo、Go）；Release Automation 改為工具比較表 |
| 2.6 | 🔀 | Section 3 缺分析步驟 | 已併入 2.4 一同解決 |
| 2.7 | ✅ | Hotfix 流程假設 Git Flow | 加 Note 標明為 Git Flow 慣例 + 其他策略替代說明 |
| 2.8 | ✅ | PR Review 與 reviewer 重疊 | pr-template.md Code Review Checklist 精簡為基本 PR 安全檢查，指向 reviewer 技能 |
| 2.9 | ✅ | Quick Reference 含禁止命令 | `git commit --amend` 和 `git reset --soft HEAD~1` 旁加 `# Confirm first` |
| 2.10 | ✅ | pr-template push 未加確認 | Step 4 push 指令前加確認提醒 |
| 2.11 | ✅ | Co-Authored-By 硬編碼版本 | 所有 `Claude Opus 4.6` 改為 `Claude` |
| 2.12 | ✅ | Safety Rules 與內建行為重複 | 精簡為引用 Claude Code 內建規則 + git-nanny 特有補充 |
| 2.13 | ✅ | branch-strategies push 確認 | Release Flow 和 Hotfix Flow 的 push 命令旁加確認注釋 |

---

### 3. spec-to-md

**修改檔案**：SKILL.md、template-structure.md、backend-spec.md、frontend-spec.md、GUIDE.zh-TW.md

| # | 狀態 | 項目 | 修復內容 |
|---|------|------|---------|
| 3.1 | ✅ | Spawn Prompt Context 過載（S1） | 改為「寫入共享 context 檔案 + teammate 自行 Read」；backend-spec.md / frontend-spec.md 加 STARTUP 區段執行 Read |
| 3.2 | ✅ | 競態條件 | 改為 TL 協調模式：teammates 向 TL 報告，TL 提取 API 清單協調交叉驗證；prompt 加「Do NOT SendMessage directly」 |
| 3.3 | ✅ | template-structure.md 客製化（S2） | 移除 `ApiRouteProcessor`、`LogType.BIZ_XXX`、`@ActivateRequestContext`、Pinia 等框架特定 API，改為通用描述 |
| 3.4 | ✅ | superpowers 強制依賴（S3） | Step 3/4a/5 改為 Optional integration + fallback |
| 3.5 | ✅ | Consistency Check 抽象 | 加入 7 項具體 checklist（API endpoint、parameter、response format、field names、error codes、permissions、component names）|
| 3.6 | ✅ | GUIDE 不一致 | 修改 FAQ 說明，建議全套產出以確保一致性 |
| 3.7 | ✅ | Spawn prompt 缺模板格式指示 | backend-spec.md 和 frontend-spec.md 加 STARTUP step 引用 template-structure.md |
| 3.8 | ✅ | Shutdown rejection 無處理 | 加「If teammate rejects shutdown, ask to wrap up first, then retry」|
| 3.9 | ✅ | TeamCreate team_name 格式 | 加 `(lowercase, no spaces, use hyphens)` 說明 |
| 3.10 | ✅ | 並行 Subagent 失敗處理 | Step 2 加 fallback：「AskUserQuestion to verify paths before retrying; if retry fails, perform in main flow」|

---

### 4. md-to-code

**修改檔案**：SKILL.md、completion-report-template.md、backend-dev.md、frontend-dev.md、GUIDE.zh-TW.md

| # | 狀態 | 項目 | 修復內容 |
|---|------|------|---------|
| 4.1 | ✅ | Context window 爆炸（S1） | Principles 加 context size 管理：>200 行文件改為摘要+路徑；prompts 加 STARTUP 指示自行 Read |
| 4.2 | ✅ | code-reviewer 未定義 | 改為「use `reviewer` skill if available (`/reviewer`); otherwise self-review」|
| 4.3 | ✅ | prompt.md 位置不明 | 加搜尋邏輯：用戶提供路徑 → Glob `**/prompt.md` → 多結果時 AskUserQuestion |
| 4.4 | ✅ | {variables} 無映射表 | Step 3b 加完整 5 變數映射表（各變數對應 Agent A/B/C 的哪個產出）|
| 4.5 | ✅ | 無分支邏輯 | 加 scope detection：只有 02（無 03）→ backend only；只有 03（無 02）→ frontend only |
| 4.6 | ✅ | 錯誤處理缺失 | Principles 加 error handling（subagent retry、TL takeover、build fail、unresponsive reassign）|
| 4.7 | ✅ | Subagent 分工重疊 | 加 conflict resolution：naming/style → Agent B 優先；impl steps → Agent C；components → merge |
| 4.8 | ✅ | 與 spec-to-md 緊耦合 | Principles 加預期 input format 聲明 + 適應機制（讀 prompt.md navigation section 取得實際路徑）|
| 4.9 | ✅ | superpowers 依賴不清（S3） | Step 2 和 Step 4 改為 Optional integration |
| 4.10 | ✅ | 完成報告模板語言不一致 | completion-report-template.md 改為英文 |
| 4.11 | ✅ | Agent B 用 Opus 成本 | Step 1 和 Principles 加註「Agent B may use sonnet for cost efficiency」|
| 4.12 | ⏭ | 半步編號風格 | 語意清晰（gate ≠ step），保留 |
| 4.13 | ⏭ | TeamCreate API 存疑 | 確認為真實 API，不修改 |
| 4.15 | ⏭ | 無版本控制指引 | 屬 git-nanny 職責，不重複 |
| 4.16 | ⏭ | 測試策略缺失 | 由 spec docs 決定，符合 YAGNI |

---

### 5. reviewer

**修改檔案**：SKILL.md、review-report-template.md

| # | 狀態 | 項目 | 修復內容 |
|---|------|------|---------|
| 5.1 | ✅ | Progressive disclosure 矛盾 | 移除矛盾措辭，改為「Load ALL standards files — they are the review source of truth」|
| 5.2 | ✅ | 沒有並行能力 | Step 3 加入：>5 檔案時 group by module/layer，launch parallel Task agents (sonnet) |
| 5.3 | ✅ | 沒有增量審查 | Step 2 加 scope resolution：「recently modified」→ `git diff --name-only` (staged/HEAD~1) |
| 5.4 | ✅ | 審查結果無持久化 | Step 4 加存檔選項：詢問是否存至 `.standards/reviews/YYYY-MM-DD-<scope>.md` |
| 5.5 | ✅ | 缺嚴重度分級 | review-report-template.md 表格加 Severity 欄（Critical/Major/Minor），Summary 加分級統計 |
| 5.6 | ✅ | 無自動修正橋接 | Step 4 加 fix workflow：Minor/Major 直接修，Critical 先確認，修後 re-review |
| 5.7 | ✅ | 大型專案效能 | Step 2 加 scale guard：>20 檔案建議縮小範圍或用 parallel subagents |
| 5.8 | ✅ | Review dimensions 二義性 | 改為「Use standards' own section structure if present; fall back to generic dimensions if not」|
| 5.9 | ✅ | review-report-template 一檔兩用 | 引用時明確指明區段（§ "Project Setup Guide" 和 § "Review Report"）|
| 5.10 | ✅ | Standards 衝突處理 | Notes 加入：「If standards files conflict, flag in report and ask user to clarify」|
| 5.11 | ✅ | Step 1 搜尋方式不明確 | 明確指定用 Glob 搜尋 |
| 5.12 | ✅ | SKILL.md 語氣不一致 | frontmatter description 語氣統一 |
| 5.13 | ⏭ | 功能單薄 | 設計意圖如此，透過 5.2-5.7 已提升價值 |
| 5.14 | ⏭ | 是否應獨立技能 | 維持獨立，有使用場景 |

---

### 6. explorer

**修改檔案**：SKILL.md、explore-subagent.md、project-map-template.md、GUIDE.zh-TW.md

| # | 狀態 | 項目 | 修復內容 |
|---|------|------|---------|
| 6.1 | ✅ | 偵測邏輯過於簡化 | 從 3 分支擴充為 5 個 primary signals（microservices、monorepo、library/SDK、full-stack、simple monolith）+ secondary signals + cap 10 |
| 6.2 | ✅ | 敏感檔案洩漏 | SKILL.md 加 Safety Rules 區段；explore-subagent.md 加 SAFETY RULES（.env 等只報告存在）+ binary/generated/data 排除清單 |
| 6.3 | ✅ | 審視迴圈無終止條件 | 加 max 3 iterations + 三分類（Factual discrepancy / Contextual uncertainty / True inconsistency），達上限後未解決項轉為 Known Issue |
| 6.4 | ✅ | Sub-agent 數量爆炸 | cap 10 + 分組策略（按 naming prefix 或目錄結構分組）|
| 6.5 | ✅ | Sub-agent 失敗處理 | 加 retry once → Leader 手動 fallback → partial success 繼續 |
| 6.6 | ✅ | 報告長度未控制 | explore-subagent.md 加 500 行上限 + 每 section top 10 items |
| 6.7 | ✅ | 探索深度不清 | 加 EXPLORATION RULES：4 層深度、500KB 限制、source files 只讀結構 |
| 6.8 | ✅ | 缺二進位排除 | 加完整排除清單（binary、generated、data dumps、media、dependencies）|
| 6.9 | ✅ | 唯讀安全指示缺失 | SKILL.md 和 explore-subagent.md 雙重 READ-ONLY 聲明 |
| 6.10 | ✅ | PROJECT_MAP 偏向 Java+Vue | `Controller -> Service -> Repository` 改為 `<describe actual layering pattern>`，範例通用化 |
| 6.11 | 🔀 | 「Clear doubt」分界線不明 | 已併入 6.3 三分類方案解決 |
| 6.12 | ⏭ | Sub-agent 用 Sonnet | 合理設計，Sonnet 適合 read+report 型任務 |
| 6.13 | ⏭ | 無增量更新機制 | 定位為一次性偵察，增量為未來 feature |
| 6.14 | ⏭ | Phase 從 0 開始 | 有語義（偵測/準備），保留 |
| 6.15 | ✅ | GUIDE 時間估計過於樂觀 | 改為「取決於專案大小和複雜度」|

---

### 7. dev-team

**修改檔案**：SKILL.md、delivery-report-template.md、process-log-template.md、trace-template.md、api-contract-template.md、qa-review-template.md、worker.md、challenger.md、GUIDE.zh-TW.md

| # | 狀態 | 項目 | 修復內容 |
|---|------|------|---------|
| 7.1 | ✅ | TL 單點瓶頸 | SKILL.md 加 batch processing 指引：多 worker 同時完成時，TL 可批次更新 TRACE 後並行 spawn QA sub-agents |
| 7.2 | ✅ | Agent Metrics 空洞 | delivery-report-template.md 的 tokens/cost 欄改為 `n/a`，Cost Breakdown 重命名為「QA Cost Breakdown (exact data only)」|
| 7.3 | ✅ | QA 審查深度不足 | qa-review-template.md 加 acceptance_criteria 和 relevant_context 變數 |
| 7.4 | ✅ | PROCESS_LOG 與 TRACE 重疊 | process-log-template.md 精簡為只記非常規事件（decisions、issues、contract amendments、team changes）|
| 7.5 | ✅ | Multi-spec 共用檔案衝突 | 加共用檔案識別和處理指引：共用 >3 個檔案時建議 Sequential 模式 |
| 7.6 | ✅ | Phase 5 驗證時機太晚 | Phase 4 加早期預警：QA 偵測到 contract mismatch 時立即廣播給受影響 workers |
| 7.7 | ✅ | Worker race condition | worker.md 加 claim 驗證步驟：TaskUpdate → TaskGet → 確認 owner 是自己 |
| 7.8 | ✅ | Challenger 措辭不當 | challenger.md 改為「When activated by TL, use TaskList to understand current progress before reviewing」|
| 7.9 | ✅ | Worker 數量計算邊界 | 加 edge cases：`<= 3pts → 1 worker may suffice`、`single L → prefer splitting` |
| 7.10 | ✅ | STOP RULE 邊界模糊 | worker.md 加例外說明：acknowledgment 中夾帶指示時視為指示處理 |
| 7.11 | ✅ | TRACE 狀態機不完整 | trace-template.md 加 Fix Task 關聯欄位 + 完整 status flow 說明 |
| 7.12 | ✅ | api-contract 硬編碼 TypeScript（S2）| 改為語言無關描述：「Use the project's primary language」|
| 7.13 | ✅ | Phase 2 對非 API 專案 | 加條件：「If requirements contain no API interactions: skip Phase 2, note N/A in TRACE」|
| 7.14 | ✅ | 複雜度評分標準缺失 | 加 S/M/L 判斷標準（S: 單檔 / M: 2-3 檔跨層 / L: 4+ 檔或核心邏輯）|
| 7.15 | ✅ | Worker context 傳遞 | worker.md 加「Read these files at the start of your work session」|
| 7.16 | ✅ | Phase 0 explorer 偵測 | 加具體偵測邏輯：Glob `**/explorer/**/SKILL.md` |
| 7.17 | ✅ | Metrics 定價硬編碼 | 在定價旁加 `# as of 2026-02` 標記 |
| 7.18 | ✅ | Challenger METRICS 格式 | challenger.md METRICS 格式擴充：`reviews={count} \| challenges={count} \| concerns={count}` |
| 7.19 | ✅ | File Scope 無技術強制 | worker.md 加 VIOLATION WARNING |
| 7.20 | 🔀 | PROCESS_LOG 事件類型不完整 | 已由 7.4 精簡方案覆蓋 |
| 7.21 | ✅ | Date prefix 時區 | 加 `(local timezone)` 說明 |
| 7.22 | ✅ | GUIDE 版本比較表 | 移除 v1.3→v2.0 對新使用者無價值的歷史比較表 |
| 7.23 | ⏭ | METRICS format fragile | 架構限制，Agent 通常遵守，不修改 |
| 7.24 | ✅ | Worker crash 無處理 | SKILL.md edge case 加：2 次無回應 → assume crash → TL reassign + spawn replacement |

---

## 三、統計摘要

| 指標 | 數量 |
|------|------|
| ✅ 已完成 | 82 項 |
| 🔀 合併處理 | 3 項（2.6, 6.11, 7.20）|
| ⏭ 略過（設計意圖） | 14 項 |
| **總計** | **99 項** |
| 修改檔案 | 34 個 |
| 新增行數 | +579 |
| 刪除行數 | -429 |
| Commit | `554181b` |
