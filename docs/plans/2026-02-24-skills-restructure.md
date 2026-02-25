# Skills Restructure Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** 將 all-skills 從 25 個插件精簡到 5 個，只保留有真正工作流的技能，並遷入 spec-to-md / md-to-code。

**Architecture:** 刪除 22 個純知識型插件，保留 core_ddd-delivery 和 devops_git，新建 standards-reviewer，從 `~/.claude/skills/` 遷入 spec-to-md 和 md-to-code。更新 README 和 example settings。

**Tech Stack:** Claude Code Skills Plugin（SKILL.md + plugin.json + references/）

---

### Task 1: Delete 22 Pure-Knowledge Plugins

**Files:**
- Delete: `plugins/core_spring-boot/` (entire directory)
- Delete: `plugins/core_quarkus/` (entire directory)
- Delete: `plugins/core_microservices/` (entire directory)
- Delete: `plugins/core_system-design/` (entire directory)
- Delete: `plugins/core_pg-standards/` (entire directory)
- Delete: `plugins/core_testing-review/` (entire directory)
- Delete: `plugins/db_postgresql/` (entire directory)
- Delete: `plugins/db_mysql/` (entire directory)
- Delete: `plugins/db_mssql/` (entire directory)
- Delete: `plugins/db_redis/` (entire directory)
- Delete: `plugins/db_schema-design/` (entire directory)
- Delete: `plugins/frontend_vue/` (entire directory)
- Delete: `plugins/frontend_quasar/` (entire directory)
- Delete: `plugins/frontend_typescript/` (entire directory)
- Delete: `plugins/devops_docker/` (entire directory)
- Delete: `plugins/devops_cicd/` (entire directory)
- Delete: `plugins/tools_api-docs/` (entire directory)
- Delete: `plugins/tools_chart-generator/` (entire directory)
- Delete: `plugins/tools_markdown-converter/` (entire directory)
- Delete: `plugins/tools_business-report/` (entire directory)
- Delete: `plugins/composite_architect/` (entire directory)
- Delete: `plugins/composite_spring-stack/` (entire directory)
- Delete: `plugins/composite_frontend/` (entire directory)

**Step 1: Delete all 22 plugin directories**

```bash
cd C:/Users/a0304/IdeaProjects/all-skills
rm -rf plugins/core_spring-boot plugins/core_quarkus plugins/core_microservices plugins/core_system-design plugins/core_pg-standards plugins/core_testing-review
rm -rf plugins/db_postgresql plugins/db_mysql plugins/db_mssql plugins/db_redis plugins/db_schema-design
rm -rf plugins/frontend_vue plugins/frontend_quasar plugins/frontend_typescript
rm -rf plugins/devops_docker plugins/devops_cicd
rm -rf plugins/tools_api-docs plugins/tools_chart-generator plugins/tools_markdown-converter plugins/tools_business-report
rm -rf plugins/composite_architect plugins/composite_spring-stack plugins/composite_frontend
```

**Step 2: Verify only 2 plugins remain**

```bash
ls plugins/
```

Expected: Only `core_ddd-delivery/` and `devops_git/` remain.

**Step 3: Commit**

```bash
git add -A && git commit -m "refactor: remove 22 pure-knowledge plugins, keep only workflow-based skills"
```

---

### Task 2: Migrate spec-to-md into all-skills

**Files:**
- Source: `C:/Users/a0304/.claude/skills/spec-to-md/` (read-only reference)
- Create: `plugins/spec-to-md/.claude-plugin/plugin.json`
- Create: `plugins/spec-to-md/skills/spec-to-md/SKILL.md`
- Create: `plugins/spec-to-md/skills/spec-to-md/references/template-structure.md`

**Step 1: Create plugin directory structure**

```bash
mkdir -p plugins/spec-to-md/.claude-plugin
mkdir -p plugins/spec-to-md/skills/spec-to-md/references
```

**Step 2: Create plugin.json**

Write `plugins/spec-to-md/.claude-plugin/plugin.json`:

```json
{
  "version": "1.0.0",
  "name": "spec-to-md",
  "description": "Spec-to-MD: Convert specification files to AI coding implementation documents",
  "author": {
    "name": "Custom Skills"
  },
  "skills": [
    "./skills/spec-to-md"
  ]
}
```

**Step 3: Copy SKILL.md content**

Copy the full content from `C:/Users/a0304/.claude/skills/spec-to-md/SKILL.md` to `plugins/spec-to-md/skills/spec-to-md/SKILL.md`. Keep content exactly as-is — no modifications.

**Step 4: Copy references/template-structure.md**

Copy the full content from `C:/Users/a0304/.claude/skills/spec-to-md/references/template-structure.md` to `plugins/spec-to-md/skills/spec-to-md/references/template-structure.md`. Keep content exactly as-is.

**Step 5: Verify structure**

```bash
find plugins/spec-to-md -type f
```

Expected:
```
plugins/spec-to-md/.claude-plugin/plugin.json
plugins/spec-to-md/skills/spec-to-md/SKILL.md
plugins/spec-to-md/skills/spec-to-md/references/template-structure.md
```

**Step 6: Commit**

```bash
git add plugins/spec-to-md/ && git commit -m "feat: migrate spec-to-md skill into all-skills"
```

---

### Task 3: Migrate md-to-code into all-skills

**Files:**
- Source: `C:/Users/a0304/.claude/skills/md-to-code/` (read-only reference)
- Create: `plugins/md-to-code/.claude-plugin/plugin.json`
- Create: `plugins/md-to-code/skills/md-to-code/SKILL.md`

**Step 1: Create plugin directory structure**

```bash
mkdir -p plugins/md-to-code/.claude-plugin
mkdir -p plugins/md-to-code/skills/md-to-code
```

**Step 2: Create plugin.json**

Write `plugins/md-to-code/.claude-plugin/plugin.json`:

```json
{
  "version": "1.0.0",
  "name": "md-to-code",
  "description": "MD-to-Code: Implement code step-by-step from structured AI coding documents",
  "author": {
    "name": "Custom Skills"
  },
  "skills": [
    "./skills/md-to-code"
  ]
}
```

**Step 3: Copy SKILL.md content**

Copy the full content from `C:/Users/a0304/.claude/skills/md-to-code/SKILL.md` to `plugins/md-to-code/skills/md-to-code/SKILL.md`. Keep content exactly as-is — no modifications.

**Step 4: Verify structure**

```bash
find plugins/md-to-code -type f
```

Expected:
```
plugins/md-to-code/.claude-plugin/plugin.json
plugins/md-to-code/skills/md-to-code/SKILL.md
```

**Step 5: Commit**

```bash
git add plugins/md-to-code/ && git commit -m "feat: migrate md-to-code skill into all-skills"
```

---

### Task 4: Create standards-reviewer Plugin

**Files:**
- Create: `plugins/standards-reviewer/.claude-plugin/plugin.json`
- Create: `plugins/standards-reviewer/skills/standards-reviewer/SKILL.md`
- Create: `plugins/standards-reviewer/skills/standards-reviewer/references/soetek-common.md`
- Create: `plugins/standards-reviewer/skills/standards-reviewer/references/epa-standards.md`
- Create: `plugins/standards-reviewer/skills/standards-reviewer/references/serp-standards.md`

**Step 1: Create plugin directory structure**

```bash
mkdir -p plugins/standards-reviewer/.claude-plugin
mkdir -p plugins/standards-reviewer/skills/standards-reviewer/references
```

**Step 2: Create plugin.json**

Write `plugins/standards-reviewer/.claude-plugin/plugin.json`:

```json
{
  "version": "1.0.0",
  "name": "standards-reviewer",
  "description": "Project standards reviewer: load project-specific coding standards and review code compliance",
  "author": {
    "name": "Custom Skills"
  },
  "skills": [
    "./skills/standards-reviewer"
  ]
}
```

**Step 3: Create SKILL.md**

Write `plugins/standards-reviewer/skills/standards-reviewer/SKILL.md`:

```markdown
---
name: standards-reviewer
description: >
  專案規範審查員：根據專案載入對應的開發規範，執行程式碼合規審查。
  支援多公司多專案規範的延遲載入。在實作完成後或 CI 前觸發，逐項核對程式碼是否符合規範。
  使用時機：實作完成後要求審查、CI 前檢查、程式碼合規確認、規範檢查。
  關鍵字：review, 審查, 規範, standards, compliance, 合規, 檢查, CI, pre-commit,
  code review, 程式碼審查, 規格檢查。
---

# 專案規範審查員 (Standards Reviewer)

## 流程

### 1. 偵測專案並載入規範

觸發後，根據以下方式確定當前專案：

1. **自動偵測**：檢查當前工作目錄的專案標識（pom.xml、package.json、專案名稱等）
2. **無法判斷時**：使用 AskUserQuestion 詢問使用者

根據專案選擇對應規範（Progressive Disclosure — 只載入需要的）：

| 條件 | 載入的規範 |
|------|-----------|
| 所有 SOETEK 專案 | `references/soetek-common.md`（公司共通規範） |
| EPA 專案 | + `references/epa-standards.md`（EPA 專案規範） |
| SERP 專案 | + `references/serp-standards.md`（SERP 專案規範） |

### 2. 確認審查範圍

向使用者確認：
- 要審查哪些檔案？（指定檔案 / 最近修改的檔案 / 整個功能模組）
- 審查深度？（快速掃描 / 完整審查）

### 3. 執行審查 Checklist

逐項核對程式碼：

#### 3.1 命名規範
- [ ] 類別、方法、變數命名是否符合規範
- [ ] 檔案路徑和目錄結構是否正確
- [ ] 常數和列舉命名

#### 3.2 架構規範
- [ ] 是否遵循指定的架構模式
- [ ] 層級職責是否正確（Controller/Service/Repository）
- [ ] 相依方向是否合規

#### 3.3 程式碼風格
- [ ] 是否使用指定的工具類別和共用元件
- [ ] 錯誤處理方式是否符合規範
- [ ] API 格式（請求/回應）是否符合標準

#### 3.4 資料庫規範
- [ ] Entity 對應是否正確
- [ ] SQL / Migration 命名和格式
- [ ] 索引和約束

#### 3.5 前端規範（如適用）
- [ ] 元件結構和命名
- [ ] 狀態管理模式
- [ ] 型別定義完整性

> **注意**：以上為通用 checklist，具體審查項目以載入的規範文件為準。

### 4. 產出審查報告

格式：

```
## 審查報告

**專案**: <專案名稱>
**載入規範**: <規範清單>
**審查範圍**: <檔案清單>
**審查時間**: <日期>

### 不合規項目

| # | 檔案 | 行號 | 規範項目 | 問題描述 | 建議修正 |
|---|------|------|----------|----------|----------|
| 1 | ... | ... | ... | ... | ... |

### 合規項目摘要

- [x] <通過的項目>

### 總結

- 審查項目數: N
- 合規: N
- 不合規: N
- 合規率: N%
```

## 新增專案規範

要支援新專案，在 `references/` 下新增對應的規範檔案，並在步驟 1 的條件表中加入判斷邏輯。

規範檔案建議結構：

```markdown
# <專案名稱> 開發規範

## 命名規範
...

## 架構規範
...

## 程式碼風格
...

## 資料庫規範
...

## 前端規範
...
```
```

**Step 4: Create placeholder reference files**

Write `plugins/standards-reviewer/skills/standards-reviewer/references/soetek-common.md`:

```markdown
# SOETEK 公司共通開發規範

> TODO: 填入公司共通的開發規範內容

## 命名規範

（待填入）

## 架構規範

（待填入）

## 程式碼風格

（待填入）

## 資料庫規範

（待填入）
```

Write `plugins/standards-reviewer/skills/standards-reviewer/references/epa-standards.md`:

```markdown
# EPA 專案開發規範

> TODO: 填入 EPA 專案特有的開發規範內容
> 本文件為 EPA 專案的補充規範，通用規範請參考 soetek-common.md

## EPA 專案特有規範

（待填入）
```

Write `plugins/standards-reviewer/skills/standards-reviewer/references/serp-standards.md`:

```markdown
# SERP 專案開發規範

> TODO: 填入 SERP 專案特有的開發規範內容
> 本文件為 SERP 專案的補充規範，通用規範請參考 soetek-common.md

## SERP 專案特有規範

（待填入）
```

**Step 5: Verify structure**

```bash
find plugins/standards-reviewer -type f
```

Expected:
```
plugins/standards-reviewer/.claude-plugin/plugin.json
plugins/standards-reviewer/skills/standards-reviewer/SKILL.md
plugins/standards-reviewer/skills/standards-reviewer/references/soetek-common.md
plugins/standards-reviewer/skills/standards-reviewer/references/epa-standards.md
plugins/standards-reviewer/skills/standards-reviewer/references/serp-standards.md
```

**Step 6: Commit**

```bash
git add plugins/standards-reviewer/ && git commit -m "feat: add standards-reviewer skill with project-specific rule loading"
```

---

### Task 5: Update README.md

**Files:**
- Modify: `README.md`

**Step 1: Rewrite README.md**

Replace entire content of `README.md` with:

```markdown
# all-skills

Claude Code 技能插件 Marketplace，共 5 個工作流型插件。

> **設計原則**：只保留有明確工作流的技能。純領域知識（Spring Boot、PostgreSQL、Vue.js 等）交給 Claude 本身的能力，流程方法論（TDD、brainstorming、debugging）交給 superpowers 等外部插件。

## 插件清單

| 插件 | 類型 | 說明 |
|------|------|------|
| `core_ddd-delivery` | 方法論 | DDD 端到端交付：Event Storming → SA → SD → 實作規劃 |
| `devops_git` | 操作流程 | Git Commit、PR、分支策略、版本發布 + 團隊規範 |
| `standards-reviewer` | 審查流程 | 專案規範審查員：根據專案載入對應規範，執行合規審查 |
| `spec-to-md` | 轉換流程 | 規格文件 → 結構化 AI Coding 實作文件 |
| `md-to-code` | 實作流程 | 實作文件 → 程式碼（並行 Agent Teams） |

## 搭配的外部插件

| 插件 | 來源 | 涵蓋 |
|------|------|------|
| superpowers | obra/superpowers-marketplace | brainstorming、TDD、debugging、code review、plan |
| document-skills | anthropic-agent-skills | docx、pptx、pdf、xlsx 文件處理 |
| claude-developer-platform | anthropic-agent-skills | Claude API 開發指南 |

## 使用方式

### 安裝

```bash
claude /plugin add ./path/to/all-skills
```

### 建議配置

將 **devops_git** 放在 User scope（全域可用），其餘按專案需求啟用。

#### User scope — 全域工具

`~/.claude/settings.json`（對應 `examples/global-settings.json`）：

```jsonc
{
  "enabledPlugins": {
    "all-skills": true,
    "devops_git@all-skills": true
  }
}
```

#### Project scope — 按專案啟用

`<project>/.claude/settings.json`（對應 `examples/project-settings.json`）：

```jsonc
{
  "enabledPlugins": {
    "all-skills": true,
    "core_ddd-delivery@all-skills": true,
    "standards-reviewer@all-skills": true,
    "spec-to-md@all-skills": true,
    "md-to-code@all-skills": true
  }
}
```

### 目錄結構

```
all-skills/
├── README.md
├── plugins/
│   ├── core_ddd-delivery/        # DDD 方法論
│   ├── devops_git/               # Git 控管流程
│   ├── standards-reviewer/       # 規範審查員（支援多專案規範）
│   ├── spec-to-md/               # 規格 → 實作文件
│   └── md-to-code/               # 實作文件 → 程式碼
├── docs/plans/                   # 設計與計畫文件
└── examples/                     # 設定檔範例
    ├── global-settings.json
    └── project-settings.json
```
```

**Step 2: Commit**

```bash
git add README.md && git commit -m "docs: rewrite README for 5-plugin restructured marketplace"
```

---

### Task 6: Update Example Settings Files

**Files:**
- Modify: `examples/global-settings.json`
- Modify: `examples/project-settings.json`

**Step 1: Update global-settings.json**

Replace entire content of `examples/global-settings.json`:

```json
{
  "enabledPlugins": {
    "all-skills": true,
    "devops_git@all-skills": true
  },
  "language": "繁體中文",
  "autoUpdatesChannel": "latest",
  "permissions": {
    "allow": [
      "Bash(*)",
      "Read",
      "Edit",
      "Write",
      "NotebookEdit",
      "WebFetch",
      "WebSearch"
    ],
    "ask": [
      "Bash(git push *)",
      "Bash(git push)"
    ]
  }
}
```

**Step 2: Update project-settings.json**

Replace entire content of `examples/project-settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(*)",
      "Read",
      "Edit",
      "Write",
      "NotebookEdit",
      "WebFetch",
      "WebSearch"
    ],
    "deny": [],
    "ask": [
      "Bash(git push *)",
      "Bash(git push)"
    ]
  },
  "enabledPlugins": {
    "all-skills": true,
    "core_ddd-delivery@all-skills": true,
    "standards-reviewer@all-skills": true,
    "spec-to-md@all-skills": true,
    "md-to-code@all-skills": true
  }
}
```

**Step 3: Commit**

```bash
git add examples/ && git commit -m "docs: update example settings for 5-plugin structure"
```

---

### Task 7: Final Verification

**Step 1: Verify plugin count**

```bash
ls plugins/
```

Expected: `core_ddd-delivery/  devops_git/  md-to-code/  spec-to-md/  standards-reviewer/`

**Step 2: Verify each plugin has correct structure**

```bash
for d in plugins/*/; do echo "=== $d ==="; find "$d" -type f; done
```

Expected: Each plugin has `.claude-plugin/plugin.json` and `skills/<name>/SKILL.md`.

**Step 3: Verify git log**

```bash
git log --oneline -10
```

Expected: 6 new commits in order:
1. refactor: remove 22 pure-knowledge plugins
2. feat: migrate spec-to-md skill
3. feat: migrate md-to-code skill
4. feat: add standards-reviewer skill
5. docs: rewrite README
6. docs: update example settings

**Step 4: Verify no untracked files**

```bash
git status
```

Expected: Clean working tree (except `.claude/` if untracked).
