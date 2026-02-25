# Superpowers Integration Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** 在 5 個 TouchFish-Skills 的 SKILL.md 中加入 superpowers 整合點 blockquote，並在 README.md 加入常用開發情境對照表。

**Architecture:** 純文件變更。每個 SKILL.md 在關鍵決策點插入統一格式的 blockquote（`> **可選整合** — 若已安裝 superpowers 插件，可搭配...`）。README.md 在「搭配的外部插件」之後新增情境對照表。所有整合點皆為可選，未安裝 superpowers 的使用者不受影響。

**Tech Stack:** Markdown（Claude Code Skills Plugin）

**Design Doc:** `docs/plans/2026-02-24-superpowers-integration-design.md`

---

### Task 1: ddd-core — 加入 5 個整合點

**Files:**
- Modify: `plugins/ddd-core/skills/ddd-core/SKILL.md`

**Step 1: 在 Phase 1 之前加入 brainstorming 整合點**

在 `## Phase 1: Event Storming 領域探索` 標題之前（第 629 行前），插入：

```markdown
> **可選整合** — 若已安裝 superpowers 插件，可在啟動 DDD 流程前先搭配 `superpowers:brainstorming` 使用，協助探索需求意圖、釐清業務範圍與設計方向。

```

**Step 2: 在 Phase 4 TDD 任務拆解處加入 TDD 整合點**

在 Phase 4 的「拆解原則」區塊（第 1259 行 `#### 拆解原則` 之後、第 1264 行 `#### 拆解順序` 之前），插入：

```markdown
> **可選整合** — 若已安裝 superpowers 插件，可搭配 `superpowers:test-driven-development` 使用，以其 TDD 紅-綠-重構循環來執行每個任務。

```

**Step 3: 在 Phase 4 工作流程末尾加入 3 個整合點**

在 Phase 4 最末尾（第 1384 行 `- 若發現 SD 設計有遺漏，主動提出並建議補充` 之後），插入：

```markdown

> **可選整合** — 若已安裝 superpowers 插件，可搭配 `superpowers:writing-plans` 使用，以其計畫格式產出更結構化的實作計畫文件。

> **可選整合** — 若已安裝 superpowers 插件，進入實作階段時可搭配 `superpowers:using-git-worktrees` 使用，在獨立的 worktree 中隔離開發。

> **可選整合** — 若已安裝 superpowers 插件，實作完成後可搭配 `superpowers:verification-before-completion` 使用，確保所有產出的完整性。
```

**Step 4: Commit**

```bash
git add plugins/ddd-core/skills/ddd-core/SKILL.md
git commit -m "docs(ddd-core): add 5 optional superpowers integration points"
```

---

### Task 2: git-nanny — 加入 4 個整合點

**Files:**
- Modify: `plugins/git-nanny/skills/git-nanny/SKILL.md`

**Step 1: 在「分支開發」區塊加入 worktree 整合點**

在第三節分支策略的「Git Flow」功能開發流程之前（第 339 行 `**功能開發流程：**` 之前），插入：

```markdown
> **可選整合** — 若已安裝 superpowers 插件，開始分支開發時可搭配 `superpowers:using-git-worktrees` 使用，在獨立的 worktree 中隔離開發。

```

**Step 2: 在 PR 建立流程的「步驟五」之後加入 code review 整合點**

在第二節 PR 的「步驟五：驗證並回傳 PR 網址」之後（第 273 行 `gh pr view --web` 程式碼區塊結束後），插入：

```markdown

> **可選整合** — 若已安裝 superpowers 插件，PR 建立後可搭配 `superpowers:requesting-code-review` 使用，獲得結構化的程式碼審查。

> **可選整合** — 若已安裝 superpowers 插件，收到 review 回饋時可搭配 `superpowers:receiving-code-review` 使用，以審慎態度處理回饋，避免盲目同意或忽略。
```

**Step 3: 在「合併策略」區塊之後加入 finishing branch 整合點**

在第三節分支策略的「分支清理」區塊之後（第 439 行 `git fetch --prune` 程式碼區塊結束後），插入：

```markdown

> **可選整合** — 若已安裝 superpowers 插件，準備合併或完成分支時可搭配 `superpowers:finishing-a-development-branch` 使用，引導結案流程（合併、PR 或清理）。
```

**Step 4: Commit**

```bash
git add plugins/git-nanny/skills/git-nanny/SKILL.md
git commit -m "docs(git-nanny): add 4 optional superpowers integration points"
```

---

### Task 3: reviewer — 加入 2 個整合點

**Files:**
- Modify: `plugins/reviewer/skills/reviewer/SKILL.md`

**Step 1: 在「產出審查報告」之前加入 verification 整合點**

在 `### 4. 產出審查報告` 標題之前（第 49 行前），插入：

```markdown
> **可選整合** — 若已安裝 superpowers 插件，在產出報告前可搭配 `superpowers:verification-before-completion` 使用，確保審查的完整性。

```

**Step 2: 在審查報告格式之後加入 debugging 整合點**

在審查報告格式區塊結束後（第 74 行 ``` 結束後），插入：

```markdown

> **可選整合** — 若已安裝 superpowers 插件，當審查發現需修復的問題時，可搭配 `superpowers:systematic-debugging` 使用，以系統化方式定位和修復問題。
```

**Step 3: Commit**

```bash
git add plugins/reviewer/skills/reviewer/SKILL.md
git commit -m "docs(reviewer): add 2 optional superpowers integration points"
```

---

### Task 4: spec-to-md — 加入 1 個新整合點

**Files:**
- Modify: `plugins/spec-to-md/skills/spec-to-md/SKILL.md`

已有的 3 個 superpowers 引用（brainstorming、writing-plans、verification-before-completion）保持不變。

**Step 1: 在步驟 4 第三階段「Agent Teams 並行」標題後加入 dispatching 整合點**

在 `#### 第三階段（Agent Teams 並行）：` 標題之後、描述文字之前（第 88 行之後），插入：

```markdown
> **可選整合** — 若已安裝 superpowers 插件，可搭配 `superpowers:dispatching-parallel-agents` 使用，以其並行任務調度方法論管理 Agent Teams。

```

**Step 2: Commit**

```bash
git add plugins/spec-to-md/skills/spec-to-md/SKILL.md
git commit -m "docs(spec-to-md): add dispatching-parallel-agents integration point"
```

---

### Task 5: md-to-code — 加入 3 個新整合點

**Files:**
- Modify: `plugins/md-to-code/skills/md-to-code/SKILL.md`

已有的 2 個 superpowers 引用（writing-plans、verification-before-completion + requesting-code-review）保持不變。

**Step 1: 在步驟 3「Agent Teams 並行實作」標題後加入 TDD 和 debugging 整合點**

在 `### 3. Agent Teams 並行實作（後端 + 前端）` 標題之後（第 79 行之後），插入：

```markdown
> **可選整合** — 若已安裝 superpowers 插件，實作過程中可搭配 `superpowers:test-driven-development` 使用，以 TDD 紅-綠-重構循環確保程式碼品質。

> **可選整合** — 若已安裝 superpowers 插件，實作過程中遇到 bug 可搭配 `superpowers:systematic-debugging` 使用，以系統化方式定位問題。

```

**Step 2: 在步驟 5「關閉團隊 + 產出完成報告」末尾加入 finishing branch 整合點**

在 SKILL.md 末尾（最後一行 `- 所有 Task Agent / Teammates 指定 \`model: "opus"\`` 之後），插入：

```markdown

> **可選整合** — 若已安裝 superpowers 插件，完成實作後可搭配 `superpowers:finishing-a-development-branch` 使用，引導開發分支的結案流程。
```

**Step 3: Commit**

```bash
git add plugins/md-to-code/skills/md-to-code/SKILL.md
git commit -m "docs(md-to-code): add 3 optional superpowers integration points"
```

---

### Task 6: README.md — 加入常用開發情境對照表

**Files:**
- Modify: `README.md`

**Step 1: 在「搭配的外部插件」表格之後、「使用方式」之前插入新區塊**

在第 23 行（外部插件表格最後一行）之後、第 25 行 `## 使用方式` 之前，插入：

```markdown

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

```

**Step 2: Commit**

```bash
git add README.md
git commit -m "docs: add development workflow scenarios with superpowers integration"
```

---

### Task 7: 最終驗證

**Step 1: 確認所有檔案已修改**

```bash
git log --oneline -6
```

預期 6 筆新 commit：
1. `docs(ddd-core): add 5 optional superpowers integration points`
2. `docs(git-nanny): add 4 optional superpowers integration points`
3. `docs(reviewer): add 2 optional superpowers integration points`
4. `docs(spec-to-md): add dispatching-parallel-agents integration point`
5. `docs(md-to-code): add 3 optional superpowers integration points`
6. `docs: add development workflow scenarios with superpowers integration`

**Step 2: 確認工作目錄乾淨**

```bash
git status
```

預期：clean working tree

**Step 3: 搜尋確認 blockquote 格式一致**

搜尋所有 SKILL.md 中的整合點：

```bash
grep -r "可選整合" plugins/
```

預期：15 個結果（ddd-core 5 + git-nanny 4 + reviewer 2 + spec-to-md 1 + md-to-code 3）
