# explorer 技能使用指南

> 版本：1.1.0 | 最後更新：2026-02-26

## 這是什麼？

explorer 是一個專案探索技能，使用 Opus Leader 指揮多個 sub-agents 並行探索專案結構。
探索完成後會交叉比對結果，最終產出結構化的 **PROJECT_MAP.md** 專案地圖文件。

**適用情境：**
- 接手新專案，需要快速了解架構
- 實作前收集專案上下文
- 產出專案地圖供團隊使用
- Onboarding 新成員時提供專案概覽

## 使用流程

```
Phase 0: 偵測專案類型
  │  讀取根目錄 → 判定類型 → 確認分工
  ▼
Phase 1: 派出探索 sub-agents
  │  載入 prompt 模板 → 填入變數 → 並行派出
  ▼
Phase 2: 審視與交叉比對
  │  8 項檢查清單 → 規範確認 → 審視迴圈
  │  ↺ 有疑問 → 再派 sub-agent 或詢問使用者
  ▼
Phase 3: 彙整產出
     確認路徑 → 載入模板 → 寫入 PROJECT_MAP.md
```

### Phase 0: 偵測專案類型

Leader 會先讀取根目錄結構，辨識 build 檔案和設定檔，然後根據以下邏輯判定專案類型：

| 條件 | 類型 | sub-agent 分工 |
|------|------|----------------|
| 有 `packages/`、`modules/` 或多個 build 檔 + 共用 infra | 混合型 | infra-explorer + 各模組 explorer |
| 有 `packages/`、`modules/` 或多個 build 檔 | 多模組 / monorepo | 各模組 explorer |
| 有明確前後端分離 | 單體 | backend-explorer + frontend-explorer |
| 其他 | 單體（簡易） | 單一 explorer |

判定後會向你確認類型和分工計畫，你可以調整探索範圍或排除特定目錄。

### Phase 1: 派出探索 sub-agents

Leader 會載入 `prompts/explore-subagent.md` 中的 prompt 模板，為每個 sub-agent 填入探索範圍和專案根目錄，然後使用 Task tool 並行派出所有 sub-agents（Sonnet 模型）。

每個 sub-agent 會回報：
1. 目錄結構與檔案分類
2. 技術棧辨識
3. 關鍵檔案索引
4. 模組間依賴關係
5. 公用組件與可複用資源
6. 專案規範與慣例

### Phase 2: 審視與交叉比對

Leader（Opus）會逐項檢查所有報告：

**審視檢查清單：**
1. 各報告的技術棧描述是否一致
2. 前後端 API 端點路徑是否對齊
3. 共用型別/介面定義是否吻合
4. 設定檔之間是否有矛盾
5. 模組間依賴方向是否合理（無循環依賴）
6. 是否有報告遺漏重要目錄或檔案
7. 公用組件清單是否完整且一致
8. 專案規範是否已找到並統整

**規範確認（必要步驟）：**
- 若 sub-agents 未找到規範文件，Leader 會詢問你：
  - 專案是否有規範文件？位於何處？
  - 是否有未文件化的慣例？
  - 是否需要建立基本規範？
- 你的回答會影響 PROJECT_MAP.md 中規範章節的內容

**審視迴圈：**
- 有明確疑問 → 再派 sub-agent 深入探索
- 有不確定 → 詢問你確認
- 全部通過後才進入 Phase 3

### Phase 3: 彙整產出

1. 確認 PROJECT_MAP.md 存放位置（預設：專案根目錄）
2. 依照 `references/project-map-template.md` 模板產出
3. 呈現摘要：專案類型、技術棧、重點問題、檔案位置

## 專案類型偵測邏輯

```
if 根目錄有 packages/ 或 modules/ 或多個 build 檔:
    type = "多模組" 或 "monorepo"
    if 有共用 infra/（docker, k8s, ci）:
        type = "混合型"
        agents = [infra-explorer] + [module-N-explorer per module]
    else:
        agents = [module-N-explorer per module]
elif 有明確前後端分離（src/main + src/webapp, 或 server/ + client/）:
    type = "單體"
    agents = [backend-explorer, frontend-explorer]
else:
    type = "單體（簡易）"
    agents = [single-explorer]
```

## 使用範例

**範例 1：探索 Spring Boot + Vue 專案**
```
> 幫我探索這個專案的架構

Phase 0: 偵測到前後端分離（server/ + client/），判定為「單體」
  → 確認派出 backend-explorer + frontend-explorer

Phase 1: 並行派出 2 個 sub-agents
  → backend-explorer 探索 server/
  → frontend-explorer 探索 client/

Phase 2: 交叉比對
  → API 端點路徑對齊 ✓
  → 共用型別一致 ✓
  → 發現 .eslintrc 與 .prettierrc 有衝突 → 詢問使用者

Phase 3: 產出 PROJECT_MAP.md 於專案根目錄
```

**範例 2：探索 monorepo 專案**
```
> 探索這個 monorepo

Phase 0: 偵測到 packages/ 下有 5 個模組 + 共用 infra/
  → 判定為「混合型」
  → 確認派出 1 infra-explorer + 5 module-explorer

Phase 1: 並行派出 6 個 sub-agents

Phase 2: 交叉比對（多模組需要更仔細的依賴檢查）

Phase 3: 產出完整的 PROJECT_MAP.md（含模組清單、依賴關係）
```

## 檔案結構

```
plugins/explorer/
├── .claude-plugin/plugin.json              ← 插件設定（v1.1.0）
├── skills/explorer/
│   ├── SKILL.md                            ← AI 指令（英文，~100 行）
│   ├── prompts/
│   │   └── explore-subagent.md             ← sub-agent prompt 模板
│   └── references/
│       └── project-map-template.md         ← PROJECT_MAP.md 產出模板
└── docs/
    └── GUIDE.zh-TW.md                      ← 本文件（中文使用指南）
```

## 常見問題

**Q: 探索需要多久？**
A: 取決於專案大小。單體小專案約 1-2 分鐘，大型 monorepo 可能 3-5 分鐘。

**Q: 可以只探索部分目錄嗎？**
A: 可以。在 Phase 0 確認時，告知 Leader 你想要的探索範圍或排除範圍。

**Q: PROJECT_MAP.md 可以放在自訂位置嗎？**
A: 可以。Phase 3 會詢問存放位置，預設是專案根目錄，也可選 `docs/` 或自訂路徑。

**Q: 探索結果不準確怎麼辦？**
A: Phase 2 的審視迴圈會盡量修正不一致。如果最終結果仍有問題，你可以直接修改 PROJECT_MAP.md 或重新執行探索。

**Q: 支援哪些專案類型？**
A: 支援任何程式語言和框架的專案。偵測邏輯會根據 build 檔案和目錄結構自動判定類型。
