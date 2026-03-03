# 變更紀錄

此檔案將記錄本專案的所有重大變更。

格式基於 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.0.0/)，
並且本專案遵循 [語意化版本 (Semantic Versioning)](https://semver.org/lang/zh-TW/)。

## [未發布]

## [2026-03-03] - 拆分 dev-team

### 變更
- **dev-team 拆分為獨立 repo**：[TouchFish-DevTeam](https://github.com/agony1997/TouchFish-DevTeam)。
  dev-team 是「編制框架」（完整開發系統），與其他 6 個「管線單元」在本質上不同，故獨立管理。
- marketplace 版本升級至 6.0.0（7 plugins → 6 plugins）

## [2026-03-02] - dev-team v3.0.0

### 破壞性變更
- **dev-team 架構重構至 v3.0.0**：
    - 移除 Challenger 角色，品質保證改由 per-task 三方交叉驗證 QA + Phase 4 全域審查
    - Worker 從任務池自取改為 **1-task-per-worker**（TL 指派，完成即 shutdown）
    - 新增 **分離測試**：test-agent（Opus sub-agent）先寫測試 → Worker 寫 code → QA 審查
    - 文件架構改為 **LLM-native 格式**（PLAN + CONTRACT）+ Markdown（DELIVERY）+ 分散式 logs/
    - Phase 從 6 個精簡為 **5 個**（P0 偵察→P1 規劃→P2 Contract→P3 開發→P4 全域審查→P5 交付）
    - 移除 TRACE / PROCESS_LOG / ISSUES 文件，改用 TaskList + 分散式 append-only log

### 新增
- **新 prompts**：`test-agent.md`、`qa-task.md`、`qa-global.md`、`delivery-sub.md`
- **新 references**：`plan-template.md`、`log-templates.md`（替換舊模板）
- **探測增強**：Phase 0 偵測 6 個整合目標（explorer、PROJECT_MAP、TDD、reviewer、.standards、OpenSpec）
- **DELIVERY 升級**：從簡要報告升級為 8 區塊開發回歸文件

### 移除
- `prompts/challenger.md`
- `references/trace-template.md`、`process-log-template.md`、`issues-template.md`、`qa-review-template.md`

## [2026-02-26] - 品質修復 v1.2.0 / dev-team v2.2.0

### 修復
- **品質審查修復** (commit `554181b`)：依據 `docs/skill-review-report.md` 對全部 7 個插件進行綜合品質修復。
- **版本號補齊**：SKILL.md 中已加入版本標記，補齊對應的 plugin.json 版本。
- **插件版本更新**：
    - `git-nanny` 更新至 1.2.0（修復 perf 版本號矛盾、擴充中文關鍵字、泛化 npm 專屬 release 流程）
    - `spec-to-md` 更新至 1.2.0（修復 context 炸彈 spawn prompts、修復 race condition）
    - `md-to-code` 更新至 1.2.0（加入 context 大小管理、修復 code-reviewer 引用、加入 scope 偵測）
    - `explorer` 更新至 1.2.0（擴展至 5 種專案類型偵測、加入敏感檔案安全規則、cap sub-agents 上限）
    - `dev-team` 更新至 2.2.0（batch 處理、誠實 metrics、早期契約警告、worker race condition 修復）
    - `ddd-core` 維持 1.1.0（修復技術棧硬編碼、Event Sourcing 虛假宣傳、NFR 指引）
    - `reviewer` 維持 1.1.0（加入平行審查、增量審查支援、嚴重程度分級）

## [2026-02-26] - 統一架構 v1.1.0

### 變更
- **統一架構 (Unified Architecture)**：所有 7 個插件已重構為統一架構：
    - **英文 SKILL.md**：AI 始終載入，採用簡潔指令風格 (terse directive style)。
    - **references/ + prompts/**：按需載入 (Glob + Read 模式)，減少 Context 佔用。
    - **docs/GUIDE.zh-TW.md**：新增繁體中文人類使用指南。
- **插件版本更新**：
    - `ddd-core` 更新至 1.1.0
    - `git-nanny` 更新至 1.1.0
    - `reviewer` 更新至 1.1.0
    - `spec-to-md` 更新至 1.1.0
    - `md-to-code` 更新至 1.1.0
    - `explorer` 更新至 1.1.0
    - `dev-team` 更新至 2.2.0

### 新增
- **效能優化**：
    - 7 個 SKILL.md 總行數從約 3,055 行減少至約 912 行 (-70%)。
    - 引入按需載入機制，現在共有 18 個按需載入檔案。
    - 為每個插件新增獨立的 `GUIDE.zh-TW.md` 使用指南。

## [1.0.0] - 初始發布

### 新增
- TouchFish-Skills 插件組初始發布。
- 包含插件：`ddd-core`, `git-nanny`, `reviewer`, `spec-to-md`, `md-to-code`, `explorer`, `dev-team`。
