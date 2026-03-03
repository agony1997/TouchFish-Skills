# 變更紀錄

此檔案將記錄本專案的所有重大變更。

格式基於 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.0.0/)，
並且本專案遵循 [語意化版本 (Semantic Versioning)](https://semver.org/lang/zh-TW/)。

## [未發布]

## [2.0.0] - 2026-03-03

### 破壞性變更
- **dev-team 拆分為獨立 repo**：[TouchFish-DevTeam](https://github.com/agony1997/TouchFish-DevTeam)。
  dev-team 是「編制框架」（完整開發系統），與其他 6 個「管線單元」在本質上不同，故獨立管理。
- marketplace 從 7 plugins 減為 **6 plugins**（移除 dev-team = breaking change）

## [1.4.0] - 2026-03-02

### 新增
- **`reviewer` 更新至 1.3.0**：新增規範萃取工作流（E1-E4），可從現有程式碼反向萃取隱含慣例產出 `.standards/` 草稿
    - E1 偵察技術棧 → E2 平行維度分析（每維度一個 sub-agent）→ E3 合併與信心評分 → E4 產出規範檔案

### 變更
- dev-team 重構至 v3.0.0（1-task-per-worker、分離測試、三方交叉驗證 QA、LLM-native 文件）
- 統一術語並加入 standards 檢查至 dev-team QA

## [1.3.0] - 2026-02-26

### 變更
- **品質審查修復**：依據專業審查報告對全部 7 個插件進行綜合品質修復

### 插件版本更新
- `git-nanny` 更新至 **1.2.0**（泛化 npm 專屬 release 流程、擴充中文關鍵字、新增 intent detection）
- `md-to-code` 更新至 **1.2.0**（加入 context 大小管理、加入 scope 偵測、新增 checkpoints）
- `explorer` 更新至 **1.2.0**（擴展至 5 種專案類型偵測、加入敏感檔案安全規則、cap sub-agents 上限）
- `reviewer` 更新至 **1.2.0**（加入平行審查、增量審查支援、嚴重程度分級）
- `ddd-core` 更新至 **1.1.1**（修復技術棧硬編碼、Event Sourcing 虛假宣傳、NFR 指引）
- `spec-to-md` 更新至 **1.1.1**（修復 context 炸彈 spawn prompts、修復 race condition）
- `dev-team` 更新至 2.2.0（batch 處理、誠實 metrics、早期契約警告、worker race condition 修復）

## [1.2.0] - 2026-02-26

### 變更
- **統一架構 (Unified Architecture)**：所有 7 個插件重構為統一架構
    - **英文 SKILL.md**（60-150 行）：AI 始終載入，terse directive style
    - **references/ + prompts/**：按需載入（Glob + Read），減少 context 佔用
    - **docs/GUIDE.zh-TW.md**：新增繁體中文人類使用指南
- 所有插件更新至 **1.1.0**（統一架構版本）
- dev-team 快速迭代（v1.2.0 → v1.3.0 → v2.0.0 → v2.1.0 → v2.2.0）

### 新增
- 6 個 SKILL.md 總行數從 ~2,895 行降至 ~752 行（-74%）
- 引入按需載入機制，共 13 個按需載入檔案
- 為每個插件新增獨立的 `GUIDE.zh-TW.md` 使用指南

## [1.1.0] - 2026-02-25

### 新增
- **`explorer` v1.0.0**：專案探索者，Opus Leader 指揮 sub-agents 並行探索，交叉比對產出 PROJECT_MAP.md
- **`dev-team` v1.0.0**：多角色開發團隊，pipeline 模式混合 agents（PM/Dev/QA）
- MIT 授權、English README、公開發布

## [1.0.0] - 2026-02-24

### 新增
- **touchfish-skills 產品誕生**：確立「只保留工作流型技能」設計哲學
- 從 33 個混合插件精煉為 5 個工作流插件：
    - `ddd-core` v1.0.0 — DDD 端到端交付
    - `git-nanny` v1.0.0 — Git 全方位專家
    - `reviewer` v1.0.0 — 專案規範審查員
    - `spec-to-md` v1.0.0 — 規格文件轉實作文件
    - `md-to-code` v1.0.0 — 實作文件轉程式碼
- 重新命名為 touchfish-skills，建立品牌識別

---

### Pre-release 歷史（0.x）

> 產品身分尚未確立的探索期，僅供參考。

| 版本 | 日期 | 事件 |
|------|------|------|
| 0.1.0 | 2026-01-28 | 初始 commit：33 個 Claude Code skill plugins |
| 0.2.0 | 2026-01-28 | 重組為 28 個插件（prefix naming + 合併） |
| 0.3.0 | 2026-02-02 | 加入 3 個 composite skills、移除 4 個與 Anthropic 重疊工具 |
