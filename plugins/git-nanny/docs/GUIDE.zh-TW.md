# git-nanny 技能使用指南

> 版本：1.1.0 | 最後更新：2026-02-26

## 這是什麼？

git-nanny 是一個 Git 全方位專家技能，涵蓋四大領域：

1. **Commit 訊息** — Conventional Commits 規範、語意化提交、HEREDOC 格式
2. **Pull Request** — PR 建立、描述生成、程式碼審查、合併衝突處理
3. **分支策略** — Git Flow / Trunk-Based / GitHub Flow 比較與建議
4. **版本發布** — Semantic Versioning、Changelog、GitHub Release、自動化

---

## 核心安全規則

git-nanny 內建嚴格的安全規則，**永遠生效，無法繞過**：

### 未經你明確要求，絕不會執行：
- `git push` / `git push --force` — 推送到遠端
- `git reset --hard` / `git checkout .` / `git clean -f` — 破壞性重置
- `git rebase --skip` / `git branch -D` — 跳過/強制刪除
- `--no-verify` / `--no-gpg-sign` — 跳過 hooks 或簽名
- 合併或關閉 PR

### 一定會遵守的：
- 提交前分析所有變更
- 產生 commit 訊息後先給你確認
- 多行訊息使用 HEREDOC 格式
- 提交後用 `git status` 和 `git log -1` 驗證
- 檔案逐一加入暫存區（不用 `git add .`）
- 建立 PR 時分析完整 commit 歷史
- PR 建立後回傳網址

### 絕對不會：
- 提交後自動推送
- 未經要求修改（amend）commit
- 跳過 pre-commit hooks
- 提交敏感檔案（.env、credentials、keys）

---

## 使用流程

根據你的需求，git-nanny 會自動判斷應載入哪個參考文件：

```
你的請求
    │
    ├── 「commit」「提交」
    │   → 載入 conventional-commits.md
    │   → 分析變更 → 分類類型 → 產生訊息 → 你確認 → 執行 → 驗證
    │
    ├── 「PR」「pull request」「review」
    │   → 載入 pr-template.md
    │   → 分析所有 commit → 產生標題 → 產生描述 → gh pr create → 回傳網址
    │
    ├── 「branch」「分支」「strategy」
    │   → 載入 branch-strategies.md
    │   → 評估專案 → 推薦策略 → 提供命名規範和合併建議
    │
    └── 「release」「tag」「changelog」
        → 載入 release-changelog.md
        → 判斷版本升級 → 更新 changelog → commit + tag → GitHub Release
```

---

## 第一節：Commit 訊息

### 基本流程

1. AI 執行 `git status`、`git diff` 分析你的變更
2. 根據 Conventional Commits 規範判斷類型（feat、fix、refactor 等）
3. 產生 commit 訊息並呈現給你
4. **你確認後**才會執行 commit（使用 HEREDOC 格式）
5. 執行後自動驗證

### 類型速查

| 類型 | 使用時機 |
|------|----------|
| `feat` | 新功能 |
| `fix` | 錯誤修復 |
| `refactor` | 重構（行為不變） |
| `perf` | 效能優化 |
| `test` | 測試相關 |
| `docs` | 文件變更 |
| `build` | 建置/依賴 |
| `ci` | CI/CD |
| `style` | 格式化 |
| `chore` | 其他 |

### 使用範例

```
你：幫我 commit 這些變更
AI：（分析後）建議使用以下 commit 訊息：
    feat(auth): add OAuth2 login support
    要執行嗎？
你：好
AI：（執行 commit，驗證成功，不會自動 push）
```

---

## 第二節：Pull Request

### 基本流程

1. AI 分析當前分支的**所有** commit（不只最新一筆）
2. 產生標題（<70 字元，祈使語氣）
3. 產生包含摘要、變更內容、測試計畫、影響範圍的描述
4. 使用 `gh pr create` 建立 PR
5. 回傳 PR 網址

### 使用範例

```
你：幫我建立 PR
AI：（分析 3 個 commits）
    標題：Add employee leave request and approval workflow
    描述：（結構化模板）
    要建立嗎？
你：好，target branch 用 develop
AI：（執行 gh pr create --base develop）
    PR 已建立：https://github.com/...
```

### 搭配 superpowers 插件

如果你有安裝 superpowers 插件：
- PR 建立後 → 使用 `superpowers:requesting-code-review` 進行結構化審查
- 收到 review 回饋 → 使用 `superpowers:receiving-code-review` 審慎處理

---

## 第三節：分支策略

### 三種策略比較

| 面向 | Git Flow | Trunk-Based | GitHub Flow |
|------|----------|-------------|-------------|
| 適用 | 大型團隊、排程發布 | 快節奏、成熟 CI/CD | SaaS、持續部署 |
| 複雜度 | 高 | 低 | 低 |
| 發布 | 排程 | 持續 | 持續 |

### 使用範例

```
你：我們團隊 5 人，兩週一次發布，應該用什麼分支策略？
AI：（載入參考文件，分析專案情境）
    建議使用 Git Flow，原因如下...
    分支命名規範：feature/、fix/、release/...
```

### 搭配 superpowers 插件

- 開始分支開發 → `superpowers:using-git-worktrees` 隔離開發
- 完成分支 → `superpowers:finishing-a-development-branch` 引導結案

---

## 第四節：版本發布

### 基本流程

1. 從 commit 歷史判斷版本升級幅度
   - `BREAKING CHANGE` → MAJOR（1.x.x → 2.0.0）
   - `feat` → MINOR（1.2.x → 1.3.0）
   - `fix` / `docs` / `refactor` → PATCH（1.2.3 → 1.2.4）
2. 更新 CHANGELOG.md（Keep a Changelog 格式）
3. Commit 版本號和 changelog 變更
4. 建立帶註解的 git tag
5. 建立 GitHub Release

### 使用範例

```
你：準備發布新版本
AI：（分析自上次 tag 以來的 commits）
    建議版本：1.3.0（有 2 個 feat commits）
    changelog 草稿：...
    確認嗎？
你：好
AI：（更新版本、changelog、commit、tag、建立 GitHub Release）
```

---

## 檔案結構

```
plugins/git-nanny/
├── .claude-plugin/
│   └── plugin.json                         ← 插件設定（v1.1.0）
├── skills/git-nanny/
│   ├── SKILL.md                            ← AI 指令（英文，~130 行）
│   └── references/                         ← 按需載入的參考文件
│       ├── conventional-commits.md         ← Conventional Commits 規範
│       ├── pr-template.md                  ← PR 模板與審查清單
│       ├── branch-strategies.md            ← 三種分支策略
│       └── release-changelog.md            ← SemVer + Changelog + Release
└── docs/
    └── GUIDE.zh-TW.md                     ← 本文件（中文使用指南）
```

### 按需載入機制

SKILL.md 只有 ~130 行，包含安全規則和簡要工作流程。
詳細的規範、模板、範例放在 `references/` 目錄下，AI 會根據你的需求自動載入對應文件。
這樣做的好處：
- 減少 context window 佔用
- 只載入需要的內容
- 安全規則永遠生效（內嵌在 SKILL.md 中）

---

## 常見問題

### Q: commit 後會自動 push 嗎？
**不會。** 這是安全規則之一。你必須明確要求 push。

### Q: 可以用中文寫 commit 訊息嗎？
Conventional Commits 規範建議使用英文。但 scope 和 body 可以視團隊規範使用中文。

### Q: AI 會自動選擇 amend 嗎？
**不會。** 除非你明確要求 amend，否則一律建立新的 commit。

### Q: 支援哪些分支策略？
三種：Git Flow（大型團隊）、Trunk-Based（快節奏）、GitHub Flow（SaaS）。
AI 會根據你的專案情境推薦最適合的。

### Q: 可以搭配其他插件使用嗎？
可以。SKILL.md 中標註了 superpowers 插件的整合點：
- `requesting-code-review` — PR 建立後的結構化審查
- `receiving-code-review` — 處理 review 回饋
- `using-git-worktrees` — 隔離分支開發
- `finishing-a-development-branch` — 分支結案流程
- `tdd-workflow` — 測試驅動開發搭配 commit

### Q: 如何處理 pre-commit hook 失敗？
AI 會修正問題後建立**新的** commit，絕不會使用 `--no-verify` 或 `--amend` 繞過。

### Q: 版本號怎麼決定？
根據 Semantic Versioning 規範，從 commit 歷史自動判斷：
- 有 `BREAKING CHANGE` → 大版本升級
- 有 `feat` → 中版本升級
- 只有 `fix`/`docs`/`refactor` → 修補版本升級
