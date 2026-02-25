# 產出文件模板結構

## prompt.md — AI 實作大綱

```markdown
# <功能編號> <功能名稱>

## 功能概述

<一段話描述功能目的、執行時機、假設前提>

## 實作範圍

- 後端：<N> 個 Processor（<列出名稱>）
- 前端：Types + Store + <N> 個 Vue 元件（<列出名稱>）

## 文件導航

依以下順序閱讀和實作：

1. **01_技術規格.md** — API 端點、資料表、Entity、業務規則的完整定義。實作前先通讀。
2. **02_後端實作.md** — 後端 Processor 逐步實作指示。依序建立每個 Processor。
3. **03_前端實作.md** — 前端 Types / Store / Vue 元件逐步實作指示。後端完成後再開始。

## 實作順序

1. 後端：Entity 方法（若需新增）→ Processor 1 → Processor 2 → ... → Route 配置
2. 前端：Types → Store → 主頁面 → 子元件 → 路由配置

## 專案規範參考

- 後端開發規範：<路徑>
- 前端開發規範：<路徑>
- 命名慣例：<路徑>
- API 規格標準：<路徑>
- 共用元件文件：<路徑>
```

## 01_技術規格.md — 完整技術規格

```markdown
# <功能編號> <功能名稱> - 技術規格

## API 端點清單

| 端點 | 方法 | 說明 | 權限 |
|------|------|------|------|
| `/api/xxx` | POST | 說明 | 權限碼 |

## API 詳細規格

### <端點名稱>

**端點**: `POST /api/xxx`
**權限**: `XXX:XXX_QUERY`

**請求參數**:
| 參數 | 類型 | 必填 | 說明 |
|------|------|------|------|

**成功回應**:
（JSON 範例）

**錯誤回應**:
（JSON 範例 + 錯誤碼表）

## 資料表結構

### <表名>
| 欄位 | 類型 | 說明 |
|------|------|------|

### 資料表關聯
（Mermaid erDiagram）

## Entity 欄位對應

### <EntityName>
- 表: `TABLE_NAME`
- 主鍵: <欄位>
- 欄位對應表

## 業務規則

1. <規則描述>
2. <規則描述>

## SQL 參考

（相關 SQL 語句）
```

## 02_後端實作.md — 後端逐步實作指示

```markdown
# <功能編號> <功能名稱> - 後端實作

## 檔案清單

| 檔案路徑 | 說明 | 操作 |
|----------|------|------|
| `backend/.../XxxProcessor.java` | 說明 | 新增 |

## 步驟 1：<ProcessorName>

建立 `backend/.../XxxProcessor.java`，繼承 `ApiRouteProcessor`。

### getTemplateParams()
回傳：
- routeId: "xxx"
- apiDescription: "說明"
- requiredFields: "field1,field2"

### getProcessorType()
回傳：`LogType.BIZ_XXX`

### process()
加上以下註解：
- `@ActivateRequestContext`
- `@RequirePermission("XXX:XXX_QUERY")` （如有權限要求）
- `@AuditLog(operation = ..., entity = "...", description = "...")`
- `@Transactional` （如有寫入操作）

### processBusinessLogic()

1. 驗證必填欄位：`validateRequiredFields(payload, "field1,field2")`
2. 提取參數：
   - `String x = JsonUtil.getStringFromMap(payload, "x")`
   - `Integer y = JsonUtil.getIntegerFromMap(payload, "y", "Y欄位")`
3. 查詢資料：`Entity.findByXxx(x, y)`
4. 組合回應：`buildStandardResponse(traceId, data, "xxx")`

### 驗證邏輯
（描述業務規則驗證的 private method）

---

## 步驟 2：<下一個 Processor>
...

## 步驟 N：Route 配置

在 `backend/src/main/resources/routes/` 新增或修改 XML：
（Route XML 內容）

## Entity 新增方法（如需要）

### <EntityName>
- `methodName(params)`: 邏輯描述
```

## 03_前端實作.md — 前端逐步實作指示

```markdown
# <功能編號> <功能名稱> - 前端實作

## 檔案清單

| 路徑 | 說明 | 操作 |
|------|------|------|
| `frontend/src/types/...` | 類型定義 | 新增 |

## 步驟 1：類型定義

建立 `frontend/src/types/<module>/<id>Types.ts`。

### Interface
（完整 TypeScript interface 定義）

### 常數
（完整常數定義）

### 工具函數
（函數簽名 + 邏輯描述）

---

## 步驟 2：Pinia Store

建立 `frontend/src/stores/<module>/<id>/use<Id>Store.ts`。

### State
（欄位表格：名稱、類型、說明、預設值）

### Getters
（Getter 清單：名稱、回傳類型、計算邏輯）

### Actions

#### fetchXxx()
1. 設定 loading = true
2. 呼叫 `api.post('/api/xxx', { ... })`
3. 更新 state
4. 處理錯誤

#### saveXxx()
1. 執行 validateBeforeSave()
2. 收集 dirty items
3. 呼叫 API
4. 重新載入資料

---

## 步驟 3：主頁面

建立 `frontend/src/pages/<module>/<id>/<Id>Query.vue`。

### 頁面結構
（組件樹，用縮排表示巢狀關係）

### 查詢區
- Header：<圖示 + 標題 + 按鈕配置>
- Body：<欄位配置，用 col-N 表示寬度>

### 結果區
- Header：<圖示 + 標題>
- Body：<表格元件>

### 按鈕狀態邏輯
| 按鈕 | 啟用條件 | 點擊行為 |
|------|----------|----------|

### onMounted 初始化
1. 載入下拉選單
2. 設定預設值
3. 自動查詢

---

## 步驟 4：子元件

### <ComponentName>.vue

**Props**:
| Prop | 類型 | 說明 |

**Events**:
| Event | 參數 | 說明 |

**實作要點**:
- <要點描述>

---

## 步驟 5：路由配置

修改 `frontend/src/router/routes.ts`，新增：
（路由配置物件）

## 使用的共用組件

| 組件 | 用途 | 參考 |
|------|------|------|
```
