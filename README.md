# all-skills

Claude Code 技能插件 Marketplace，共 26 個插件，依前綴分為 5 大類。

## 插件清單

### core_ — 核心開發（7）

| 插件 | 說明 |
|------|------|
| `core_system-design` | 系統架構設計：技術選型、架構模式、NFR、整潔架構/六邊形架構 |
| `core_microservices` | 微服務架構模式，服務拆分、API Gateway、分散式事務 |
| `core_ddd-delivery` | DDD 端到端交付：理論 → Event Storming → SA → SD → 實作規劃 |
| `core_spring-boot` | Spring Boot 開發：REST API、自動配置、JPA/資料存取、效能優化 |
| `core_quarkus` | Quarkus 雲原生 Java 開發 |
| `core_testing-review` | 品質保證：TDD、測試策略、程式碼審查、重構 |
| `core_pg-standards` | PG 紀律遵規，規格書與程式碼風格 |

### db_ — 資料庫（5）

| 插件 | 說明 |
|------|------|
| `db_schema-design` | DDD 導向資料庫設計，聚合邊界→表結構、DDL、Flyway |
| `db_postgresql` | PostgreSQL 進階：JSONB、全文搜尋、分區表、窗口函數 |
| `db_mysql` | MySQL 索引優化、查詢優化、複製架構 |
| `db_mssql` | SQL Server T-SQL 與效能調校 |
| `db_redis` | Redis 快取策略、分散式鎖、Cluster |

### frontend_ — 前端（3）

| 插件 | 說明 |
|------|------|
| `frontend_vue` | Vue 2/3 Composition API、Vue Router、Pinia |
| `frontend_quasar` | Quasar Framework Vue 3 多平台應用 |
| `frontend_typescript` | TypeScript 泛型、進階型別、工具型別 |

### devops_ — DevOps（3）

| 插件 | 說明 |
|------|------|
| `devops_docker` | Docker 容器化、Dockerfile、Docker Compose |
| `devops_cicd` | CI/CD GitHub Actions、GitLab CI、Jenkins |
| `devops_git` | Git Commit、PR、分支策略、版本發布 |

### tools_ — 文件工具（8）

| 插件 | 說明 |
|------|------|
| `tools_api-docs` | API 文檔 OpenAPI 3.0/Swagger |
| `tools_business-report` | 商業報告：週報、月報、季報 |
| `tools_chart-generator` | 圖表生成 Mermaid/PlantUML |
| `tools_excel-converter` | Excel/CSV/JSON 格式互轉 |
| `tools_markdown-converter` | Markdown ↔ HTML/PDF/DOCX |
| `tools_pdf-processor` | PDF 合併、分割、提取、浮水印 |
| `tools_tech-presentation` | 技術簡報 PowerPoint 生成 |
| `tools_word-processor` | Word 文檔建立與編輯 |

## 使用方式

### 安裝

```bash
claude /plugin add ./path/to/all-skills
```

### 快速開始（複製範例設定檔）

本專案在 `examples/` 目錄提供可直接複製使用的設定檔：

| 檔案 | 用途 | 複製目標 |
|------|------|----------|
| `examples/global-settings.json` | 全域設定（User scope） | `~/.claude/settings.json` |
| `examples/project-settings.json` | 專案設定（Project scope） | `<project>/.claude/settings.json` |

**一鍵複製指令：**

```bash
# 複製全域設定（啟用 devops_git + 所有 tools_ 插件）
cp examples/global-settings.json ~/.claude/settings.json

# 複製專案設定（以 Spring Boot + PostgreSQL + Vue/Quasar 為例）
mkdir -p <project>/.claude
cp examples/project-settings.json <project>/.claude/settings.json
```

> 複製後請依自身需求調整 `enabledPlugins` 中各插件的 `true`/`false`。

### 建議配置

將 **devops_git + tools** 放在 User scope（全域可用），**技術棧**放在 Project scope（按專案需求啟用）。

插件在 `enabledPlugins` 中的 key 格式為 `插件名@marketplace名`，例如 `devops_git@all-skills`。

#### User scope — 全域工具

透過 `/plugin` 安裝 marketplace 後，在 User scope 啟用 `devops_git` 和所有 `tools_` 插件。

`~/.claude/settings.json`（對應 `examples/global-settings.json`）：

```jsonc
{
  "enabledPlugins": {
    "all-skills": true,                          // marketplace 本體
    "subtask": true,                             // 平行任務編排
    "devops_git@all-skills": true,               // Git 全域可用
    "tools_api-docs@all-skills": true,
    "tools_business-report@all-skills": true,
    "tools_chart-generator@all-skills": true,
    "tools_excel-converter@all-skills": true,
    "tools_markdown-converter@all-skills": true,
    "tools_pdf-processor@all-skills": true,
    "tools_tech-presentation@all-skills": true
  },
  "language": "繁體中文",
  "autoUpdatesChannel": "latest",
  "permissions": {
    "allow": [
      "Bash(*)", "Read", "Edit", "Write",
      "NotebookEdit", "WebFetch", "WebSearch"
    ],
    "ask": [
      "Bash(git push *)",
      "Bash(git push)"
    ]
  }
}
```

#### Project scope — 按專案啟用技術棧

在專案根目錄建立 `.claude/settings.json`，只啟用該專案需要的技術棧插件，並可將全域已啟用但專案不需要的 tools 設為 `false` 覆蓋。

`<project>/.claude/settings.json`（對應 `examples/project-settings.json`）：

**範例 A：Spring Boot + PostgreSQL + Vue/Quasar（個人開發）**

```jsonc
{
  "permissions": {
    "allow": [
      "Bash(*)", "Read", "Edit", "Write",
      "NotebookEdit", "WebFetch", "WebSearch"
    ],
    "deny": [],
    "ask": ["Bash(git push *)", "Bash(git push)"]
  },
  "enabledPlugins": {
    "all-skills": true,
    "core_ddd-delivery@all-skills": true,
    "core_system-design@all-skills": true,
    "core_spring-boot@all-skills": true,
    "core_testing-review@all-skills": true,
    "db_schema-design@all-skills": true,
    "db_postgresql@all-skills": true,
    "frontend_vue@all-skills": true,
    "frontend_quasar@all-skills": true,
    "frontend_typescript@all-skills": true,
    // 專案不需要的 tools 設為 false 覆蓋全域設定
    "tools_api-docs@all-skills": false,
    "tools_business-report@all-skills": false,
    "tools_chart-generator@all-skills": false,
    "tools_excel-converter@all-skills": false,
    "tools_markdown-converter@all-skills": false,
    "tools_pdf-processor@all-skills": false,
    "tools_tech-presentation@all-skills": false
  }
}
```

**範例 B：Quarkus + MSSQL + Quasar（公司開發）**

```jsonc
{
  "enabledPlugins": {
    "all-skills": true,
    "core_pg-standards@all-skills": true,
    "core_quarkus@all-skills": true,
    "core_testing-review@all-skills": true,
    "db_mssql@all-skills": true,
    "frontend_vue@all-skills": true,
    "frontend_quasar@all-skills": true,
    "frontend_typescript@all-skills": true
  }
}
```

### 覆蓋 User scope 設定

如果某個插件在 User scope 已啟用，但特定專案不需要，可以在 Project scope 中設為 `false` 覆蓋：

```jsonc
// <project>/.claude/settings.json
{
  "enabledPlugins": {
    "tools_business-report@all-skills": false    // 此專案停用商業報告
  }
}
```

不寫的插件會繼承 User scope 的值。

### 設定層級

```
User scope（~/.claude/settings.json）
  ├─ all-skills: true                        ← marketplace 本體
  ├─ subtask: true                           ← 平行任務編排
  ├─ devops_git + tools_ 全域可用，所有專案繼承
  └─ permissions / language / autoUpdatesChannel

Project scope（<project>/.claude/settings.json）
  ├─ core_ / db_ / frontend_ 按技術棧啟用
  ├─ 可用 false 覆蓋 User scope 中不需要的插件
  └─ permissions（可獨立設定）
```

### 目錄結構

```
all-skills/
├── README.md
├── plugins/                    # 26 個技能插件
│   ├── core_*/
│   ├── db_*/
│   ├── frontend_*/
│   ├── devops_*/
│   └── tools_*/
└── examples/                   # 可直接複製的設定檔範例
    ├── global-settings.json    # → ~/.claude/settings.json
    └── project-settings.json   # → <project>/.claude/settings.json
```
