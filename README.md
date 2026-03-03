# touchfish-skills

> Workflow-oriented skill plugins for Claude Code — 6 plugins covering DDD, Git, code review, spec conversion, implementation, and project exploration.

[![Version](https://img.shields.io/badge/version-2.0.0-green.svg)](CHANGELOG.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Plugins](https://img.shields.io/badge/plugins-6-blue.svg)](.claude-plugin/marketplace.json)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skills-blueviolet.svg)](https://docs.anthropic.com/en/docs/claude-code)

**繁體中文** | [English](README.en.md)

Claude Code 技能插件 Marketplace，共 6 個工作流型插件。

> **設計原則**：只保留有明確工作流的技能。純領域知識（Spring Boot、PostgreSQL、Vue.js 等）交給 Claude 本身的能力，流程方法論（TDD、brainstorming、debugging）交給 superpowers 等外部插件。

> **找 dev-team？** dev-team 已拆分為獨立 repo：[TouchFish-DevTeam](https://github.com/agony1997/TouchFish-DevTeam)。

### v1.1.0 統一架構（2026-02-26）

所有 6 個插件統一為：

- **英文 SKILL.md**（60–150 行）— AI 始終載入，terse directive style
- **references/ + prompts/**（按需載入）— 減少 context 佔用，Glob + Read 模式
- **docs/GUIDE.zh-TW.md** — 中文人類使用指南

| 指標 | v1.0.0 | v1.1.0 統一架構 | 目前 |
|------|--------|---------------|------|
| 6 個 SKILL.md 總行數 | ~2,895 | ~752（-74%） | ~914（-68%） |
| 按需載入檔案 | 1 | 13 | 19 |
| 中文人類指南 | 1 | 6 | 6 |

## 安裝

```bash
# 直接在 Claude Code 中安裝（支援整個 Repo 作為 Marketplace）
claude mcp add-plugin https://github.com/agony1997/TouchFish-Skills
```

> **Tip**：建議將 `git-nanny` 設為 User scope（全域可用），其餘插件依專案啟用。

## 插件清單

| 插件 | 版本 | 類型 | 說明 | 人類指南 |
|------|------|------|------|---------|
| `ddd-core` | 1.1.1 | 方法論 | DDD 端到端交付：DDD 理論基礎 → Event Storming → SA → SD → 實作規劃 | [指南](plugins/ddd-core/docs/GUIDE.zh-TW.md) |
| `git-nanny` | 1.2.0 | 操作流程 | Git 全方位專家：Commit 訊息、PR 建立與審查、分支策略、版本發布與 Changelog | [指南](plugins/git-nanny/docs/GUIDE.zh-TW.md) |
| `reviewer` | 1.3.0 | 審查流程 | 專案規範審查與萃取：合規審查 + 從程式碼反向萃取慣例產出 `.standards/` 草稿 | [指南](plugins/reviewer/docs/GUIDE.zh-TW.md) |
| `spec-to-md` | 1.1.1 | 轉換流程 | 規格文件轉 AI Coding 實作文件：產出技術規格、前後端實作指示 | [指南](plugins/spec-to-md/docs/GUIDE.zh-TW.md) |
| `md-to-code` | 1.2.0 | 實作流程 | 根據實作文件實作程式碼：並行 Agent Teams 開發後端與前端 | [指南](plugins/md-to-code/docs/GUIDE.zh-TW.md) |
| `explorer` | 1.2.0 | 探索工具 | 專案探索者：Opus Leader 指揮 sub-agents 並行探索，交叉比對產出專案地圖 | [指南](plugins/explorer/docs/GUIDE.zh-TW.md) |

> **大功能團隊協作？** 請使用 [TouchFish-DevTeam](https://github.com/agony1997/TouchFish-DevTeam)（dev-team v3.0.0）。

## 架構概覽

```
需求 ──→ ddd-core ──→ spec-to-md ──→ md-to-code ──→ git-nanny
          (DDD)        (規格→文件)    (文件→程式碼)   (提交/PR)

探索 ──→ explorer ──→ PROJECT_MAP.md
          (並行探索)

審查 ──→ reviewer ──→ 合規報告 / .standards/ 草稿
          (規範審查 + 慣例萃取)

大功能 ─→ TouchFish-DevTeam ──→ PM → API 契約 → 開發 + QA 流水線 → 交付
          （獨立 repo）          https://github.com/agony1997/TouchFish-DevTeam
```

## 搭配的外部插件

| 插件 | 來源 | 涵蓋 |
|------|------|------|
| superpowers | obra/superpowers-marketplace | brainstorming、TDD、debugging、code review、plan |
| document-skills | anthropic-agent-skills | docx、pptx、pdf、xlsx 文件處理 |
| claude-developer-platform | anthropic-agent-skills | Claude API 開發指南 |

## 常用開發情境

以下列出常見開發情境的推薦技能組合。**粗體**為 touchfish-skills 技能，其餘為 superpowers 技能（需另行安裝 superpowers 插件）。

| 情境 | 推薦技能組合 | 流程 |
|------|-------------|------|
| 接手新專案 | **`explorer`** | 並行探索專案結構 → 產出 PROJECT_MAP.md |
| 新功能（完整 DDD） | `brainstorming` → **`ddd-core`** → **`spec-to-md`** → **`md-to-code`** → **`git-nanny`** | 腦力激盪 → DDD 四階段 → 產出文件 → 並行實作 → 提交/PR |
| 新功能（大團隊） | [TouchFish-DevTeam](https://github.com/agony1997/TouchFish-DevTeam) `dev-team` | PM 需求分析 → API 契約 → 多角色流水線開發 → QA 審查 → 交付 |
| 新功能（輕量版） | `brainstorming` → `writing-plans` → **`md-to-code`** → **`git-nanny`** | 腦力激盪 → 寫計畫 → 實作 → 提交/PR |
| Bug 修復 | `systematic-debugging` → `test-driven-development` → **`git-nanny`** | 系統化定位 → TDD 修復 → 提交/PR |
| 程式碼審查 | **`reviewer`** + `requesting-code-review` | 專案規範審查 + 審查流程 |
| 重構 | `brainstorming` → `writing-plans` → `test-driven-development` → **`git-nanny`** | 討論範圍 → 擬計畫 → TDD 重構 → 提交/PR |

> **未安裝 superpowers？** 所有 touchfish-skills 技能皆可獨立使用，superpowers 整合點僅為可選增強。


## 目錄結構

```
touchfish-skills/
├── .claude-plugin/marketplace.json      # Marketplace 設定（v2.0.0）
├── README.md / README.en.md
├── CHANGELOG.md
├── LICENSE
├── plugins/
│   ├── ddd-core/
│   │   ├── .claude-plugin/plugin.json
│   │   ├── skills/ddd-core/
│   │   │   ├── SKILL.md                 # AI 指令（英文，始終載入）
│   │   │   └── references/              # 按需載入（理論、模板）
│   │   └── docs/GUIDE.zh-TW.md          # 中文人類指南
│   ├── git-nanny/                       # 同上結構
│   ├── reviewer/                        # 含 prompts/（萃取 sub-agent）
│   ├── spec-to-md/
│   │   ├── skills/spec-to-md/
│   │   │   ├── SKILL.md
│   │   │   ├── prompts/                 # Teammate spawn 模板（按需載入）
│   │   │   └── references/
│   │   └── docs/GUIDE.zh-TW.md
│   ├── md-to-code/                      # 同 spec-to-md 結構
│   └── explorer/                        # 含 prompts/ + references/
```

## Attribution

本專案的 `git-nanny` 技能引用了以下開放標準：

- [Conventional Commits](https://www.conventionalcommits.org/) (CC BY 3.0)
- [Semantic Versioning](https://semver.org/) (CC BY 3.0)
- [Keep a Changelog](https://keepachangelog.com/) (MIT)

## License

[MIT](LICENSE)
