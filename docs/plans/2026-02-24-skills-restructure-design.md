# Skills Restructure Design

## Background

all-skills 專案原有 25 個插件，分為 core、db、frontend、devops、tools、composite 六大類。
經分析發現大多數為「角色扮演型」純領域知識技能，Claude 本身已具備這些知識，
加上外部插件（superpowers、document-skills）已涵蓋流程方法論，自維護價值低。

### 核心問題
1. 25 個技能太多，context 佔用大
2. 純知識型技能效果不如預期（Claude 本身就懂）
3. 觸發不精準（太廣或太窄）
4. 缺乏可執行的工作流

## Decision: 方案 B — 只留工作流型

從 25 個精簡到 5 個，只保留有真正工作流的技能。

## 最終技能清單

### 自維護（all-skills 內，5 個）

| 技能 | 類型 | 說明 |
|------|------|------|
| `core_ddd-delivery` | 方法論 | DDD 4 階段交付（Event Storming → SA → SD → Implementation） |
| `devops_git` | 操作流程 | Git 控管流程 + Conventional Commits + 團隊規範 |
| `standards-reviewer` | 審查流程 | 新建。根據專案載入對應規範並執行審查 checklist |
| `spec-to-md` | 轉換流程 | 遷入。規格文件 → 結構化 AI coding 實作文件 |
| `md-to-code` | 實作流程 | 遷入。實作文件 → 程式碼 |

### 外部插件（不維護）

| 插件 | 來源 | 涵蓋 |
|------|------|------|
| superpowers | obra/superpowers-marketplace | brainstorming、TDD、debugging、code review、plan |
| document-skills | anthropic-agent-skills | docx、pptx、pdf、xlsx 文件處理 |
| claude-developer-platform | anthropic-agent-skills | Claude API 開發指南 |

### 刪除（22 個）

core_spring-boot, core_quarkus, core_microservices, core_system-design,
core_pg-standards, core_testing-review, db_postgresql, db_mysql, db_mssql,
db_redis, db_schema-design, frontend_vue, frontend_quasar, frontend_typescript,
devops_docker, devops_cicd, tools_api-docs, tools_chart-generator,
tools_markdown-converter, tools_business-report, composite_architect,
composite_spring-stack, composite_frontend

## standards-reviewer 設計

### 架構
```
standards-reviewer/
├── SKILL.md                    # 核心審查工作流
└── references/
    ├── soetek-common.md        # SOETEK 公司共通規範
    ├── epa-standards.md        # EPA 專案規範
    └── serp-standards.md       # SERP 專案規範
```

### 工作流
1. 偵測專案 → 根據目錄 / 設定檔 / 使用者指定確定專案
2. 載入規範 → 必定載入 soetek-common.md + 對應專案規範
3. 執行審查 checklist → 逐項核對程式碼
4. 產出審查報告 → 不合規項目 + 建議修正

### Progressive Disclosure
- SKILL.md 只含工作流（少量 token）
- references/ 延遲載入（只讀取當前專案對應的規範）

## 實作步驟概要

1. 刪除 22 個插件目錄
2. 新建 standards-reviewer 插件
3. 遷入 SpecToMd / MdToCode 到 all-skills
4. 更新 README.md 和 example settings
5. Git commit
