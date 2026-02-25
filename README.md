# touchfish-skills

Claude Code 技能插件 Marketplace，共 5 個工作流型插件。

> **設計原則**：只保留有明確工作流的技能。純領域知識（Spring Boot、PostgreSQL、Vue.js 等）交給 Claude 本身的能力，流程方法論（TDD、brainstorming、debugging）交給 superpowers 等外部插件。

## 插件清單

| 插件 | 類型 | 說明 |
|------|------|------|
| `ddd-core` | 方法論 | DDD 端到端交付：Event Storming → SA → SD → 實作規劃 |
| `git-nanny` | 操作流程 | Git Commit、PR、分支策略、版本發布 + 團隊規範 |
| `reviewer` | 審查流程 | 專案規範審查員：根據專案載入對應規範，執行合規審查 |
| `spec-to-md` | 轉換流程 | 規格文件 → 結構化 AI Coding 實作文件 |
| `md-to-code` | 實作流程 | 實作文件 → 程式碼（並行 Agent Teams） |

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
| 新功能（完整 DDD） | `brainstorming` → **`ddd-core`** → **`spec-to-md`** → **`md-to-code`** → **`git-nanny`** | 腦力激盪 → DDD 四階段 → 產出文件 → 並行實作 → 提交/PR |
| 新功能（輕量版） | `brainstorming` → `writing-plans` → **`md-to-code`** → **`git-nanny`** | 腦力激盪 → 寫計畫 → 實作 → 提交/PR |
| Bug 修復 | `systematic-debugging` → `test-driven-development` → **`git-nanny`** | 系統化定位 → TDD 修復 → 提交/PR |
| 程式碼審查 | **`reviewer`** + `requesting-code-review` | 專案規範審查 + 審查流程 |
| 重構 | `brainstorming` → `writing-plans` → `test-driven-development` → **`git-nanny`** | 討論範圍 → 擬計畫 → TDD 重構 → 提交/PR |

> **未安裝 superpowers？** 所有 touchfish-skills 技能皆可獨立使用，superpowers 整合點僅為可選增強。

## 使用方式

### 安裝

```bash
claude /plugin add ./path/to/touchfish-skills
```

### 建議配置

將 **git-nanny** 放在 User scope（全域可用），其餘按專案需求啟用。

#### User scope — 全域工具

`~/.claude/settings.json`（對應 `examples/global-settings.json`）：

```jsonc
{
  "enabledPlugins": {
    "touchfish-skills": true,
    "git-nanny@touchfish-skills": true
  }
}
```

#### Project scope — 按專案啟用

`<project>/.claude/settings.json`（對應 `examples/project-settings.json`）：

```jsonc
{
  "enabledPlugins": {
    "touchfish-skills": true,
    "ddd-core@touchfish-skills": true,
    "reviewer@touchfish-skills": true,
    "spec-to-md@touchfish-skills": true,
    "md-to-code@touchfish-skills": true
  }
}
```

### 目錄結構

```
touchfish-skills/
├── README.md
├── plugins/
│   ├── ddd-core/                    # DDD 方法論
│   ├── git-nanny/                   # Git 控管流程
│   ├── reviewer/                    # 規範審查員（讀取專案內規範）
│   ├── spec-to-md/                  # 規格 → 實作文件
│   └── md-to-code/                  # 實作文件 → 程式碼
├── docs/plans/                      # 設計與計畫文件
└── examples/                        # 設定檔範例
    ├── global-settings.json
    └── project-settings.json
```
