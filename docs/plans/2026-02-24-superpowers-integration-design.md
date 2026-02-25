# Superpowers 整合設計

## 背景

TouchFish-Skills 有 5 個自訂工作流技能，superpowers 有 13 個流程方法論技能。
目前 `spec-to-md` 和 `md-to-code` 已部分引用 superpowers，其餘 3 個技能未引用。
使用者希望建立一致的整合方式，讓兩套技能在開發流程中自然串接。

## 設計決策

- **方案**：方案 C — 步驟內嵌 blockquote 選項 + README 情境對照表
- **深度**：提供使用者選擇（非強制委派，也非純補充）
- **SOP 範圍**：3-5 個常用開發情境的技能組合表
- **產出位置**：README.md + 各 SKILL.md
- **相容性**：未安裝 superpowers 的使用者不受影響，技能流程仍完整可用

## 統一格式規範

### Blockquote 格式

```markdown
> **可選整合** — 若已安裝 superpowers 插件，可搭配 `superpowers:xxx` 使用。[一句話說明好處]。
```

### 放置原則

- 只在**關鍵決策點**放置，不是每個步驟都加
- 放在該步驟描述之後、下一步驟之前
- 如果有功能重疊（如 TDD），提示使用者可自由選擇
- 不影響原有流程的完整性 — 移除所有 blockquote 後流程仍完整

## 各技能整合點

### 1. ddd-core

| 位置 | 整合提示 | superpowers 技能 |
|------|---------|-----------------|
| 流程啟動前（Phase 1 之前） | 探索需求意圖和設計方向 | `brainstorming` |
| Phase 4 完成後 | 可用 superpowers 格式產出實作計畫 | `writing-plans` |
| Phase 4 TDD 任務拆解 | 可改用 superpowers 的 TDD 流程 | `test-driven-development` |
| 進入實作階段 | 可用 worktree 隔離開發 | `using-git-worktrees` |
| 實作完成後 | 驗證完成度 | `verification-before-completion` |

### 2. git-nanny

| 位置 | 整合提示 | superpowers 技能 |
|------|---------|-----------------|
| PR 建立後 | 可請求程式碼審查 | `requesting-code-review` |
| 收到 review 回饋後 | 以審慎態度處理回饋 | `receiving-code-review` |
| 分支開發開始時 | 可用 worktree 隔離 | `using-git-worktrees` |
| 準備合併/完成分支 | 完成開發分支結案流程 | `finishing-a-development-branch` |

### 3. reviewer

| 位置 | 整合提示 | superpowers 技能 |
|------|---------|-----------------|
| 審查完成後發現問題需修復 | 以系統化方式除錯 | `systematic-debugging` |
| 審查報告產出前 | 驗證審查完整性 | `verification-before-completion` |

### 4. spec-to-md（已有部分引用，補強）

| 位置 | 整合提示 | superpowers 技能 | 狀態 |
|------|---------|-----------------|------|
| 步驟 3 確認需求 | 探索需求和設計意圖 | `brainstorming` | 已有 |
| 步驟 4 第一階段 prompt.md | 規劃方法論 | `writing-plans` | 已有 |
| 步驟 5 最終確認 | 驗證完成度 | `verification-before-completion` | 已有 |
| 步驟 4 第三階段 Agent Teams | 並行任務調度 | `dispatching-parallel-agents` | 新增 |

### 5. md-to-code（已有部分引用，補強）

| 位置 | 整合提示 | superpowers 技能 | 狀態 |
|------|---------|-----------------|------|
| 步驟 2 產生實作計畫 | 規劃方法論 | `writing-plans` | 已有 |
| 步驟 4 實作後驗證 | 驗證 + 審查 | `verification-before-completion`, `requesting-code-review` | 已有 |
| 步驟 3 實作階段 | 可啟用 TDD 流程 | `test-driven-development` | 新增 |
| 步驟 3 遇到 bug | 系統化除錯 | `systematic-debugging` | 新增 |
| 步驟 5 完成後 | 完成開發分支結案 | `finishing-a-development-branch` | 新增 |

## README 常用情境對照表

| 情境 | 推薦技能組合 | 流程 |
|------|-------------|------|
| 新功能（完整 DDD） | `brainstorming` → `ddd-core` → `spec-to-md` → `md-to-code` → `git-nanny` | 腦力激盪 → DDD 四階段 → 產出文件 → 並行實作 → 提交/PR |
| 新功能（輕量版） | `brainstorming` → `writing-plans` → `md-to-code` → `git-nanny` | 腦力激盪 → 寫計畫 → 實作 → 提交/PR |
| Bug 修復 | `systematic-debugging` → `test-driven-development` → `git-nanny` | 系統化定位 → TDD 修復 → 提交/PR |
| 程式碼審查 | `reviewer` + `requesting-code-review` | 專案規範審查 + 審查流程 |
| 重構 | `brainstorming` → `writing-plans` → `test-driven-development` → `git-nanny` | 討論範圍 → 擬計畫 → TDD 重構 → 提交/PR |

## 修改範圍

### 需修改的檔案

1. `README.md` — 加入「常用開發情境」區塊
2. `plugins/ddd-core/skills/ddd-core/SKILL.md` — 加入 5 個整合點 blockquote
3. `plugins/git-nanny/skills/git-nanny/SKILL.md` — 加入 4 個整合點 blockquote
4. `plugins/reviewer/skills/reviewer/SKILL.md` — 加入 2 個整合點 blockquote
5. `plugins/spec-to-md/skills/spec-to-md/SKILL.md` — 加入 1 個新整合點（已有 3 個保持不變）
6. `plugins/md-to-code/skills/md-to-code/SKILL.md` — 加入 3 個新整合點（已有 2 個保持不變）

### 不修改的部分

- 各技能的核心工作流程文本
- 現有的 superpowers 引用（spec-to-md、md-to-code 中已有的）
- plugin.json 和 marketplace.json
- 目錄結構
