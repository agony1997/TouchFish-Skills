# 產出文件模板結構

## prompt.md — AI 實作大綱

```markdown
# <功能編號> <功能名稱>

## 功能概述

<一段話描述功能目的、執行時機、假設前提>

## 實作範圍

- 後端：<N> 個 Handler/Controller/Processor（<列出名稱>）
- 前端：Types + State Management + <N> 個元件（<列出名稱>）

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
| `<path/to/HandlerOrController>` | 說明 | 新增/修改 |

## 步驟 1：<HandlerName> (adapt naming to project convention)

建立 `<path/to/Handler>`，繼承/實作 `<project base class or interface>`.

### 設定/元資料 (adapt to project pattern)
回傳：
- <endpoint identifier>
- <description>
- <required fields / validation config>

### 核心處理方法

加上專案慣用的 annotation/decorator（權限、交易、日誌等，依專案規範）。

### 業務邏輯

1. 驗證必填欄位
2. 提取並轉換參數
3. 查詢/操作資料
4. 組合回應

（每步列出具體欄位名稱、呼叫的方法、使用的 Entity/Model）

### 驗證邏輯
（描述業務規則驗證的 private method/function）

---

## 步驟 2：<下一個 Handler>
...

## 步驟 N：路由/端點配置

在專案路由配置位置新增或修改端點設定。
（具體配置內容）

## Entity/Model 新增方法（如需要）

### <EntityName>
- `methodName(params)`: 邏輯描述
```

## 03_前端實作.md — 前端逐步實作指示

```markdown
# <功能編號> <功能名稱> - 前端實作

## 檔案清單

| 路徑 | 說明 | 操作 |
|------|------|------|
| `<path/to/types>` | 類型定義 | 新增 |

## 步驟 1：類型定義

建立 `<path per project convention>`.

### Interface / Type
（完整類型定義，語言依專案技術棧）

### 常數
（完整常數定義）

### 工具函數
（函數簽名 + 邏輯描述）

---

## 步驟 2：State Management (Store / State / Context)

建立 `<path per project convention>`.

### State
（欄位表格：名稱、類型、說明、預設值）

### Derived State / Getters / Selectors
（清單：名稱、回傳類型、計算邏輯）

### Actions / Methods

#### fetchXxx()
1. 設定 loading state
2. 呼叫 API
3. 更新 state
4. 處理錯誤

#### saveXxx()
1. 執行驗證
2. 收集變更項目
3. 呼叫 API
4. 重新載入資料

---

## 步驟 3：主頁面

建立 `<path per project convention>`.

### 頁面結構
（組件樹，用縮排表示巢狀關係）

### 查詢區 / 輸入區
- Header：<標題 + 操作按鈕配置>
- Body：<欄位配置與佈局>

### 結果區 / 展示區
- Header：<標題>
- Body：<列表/表格元件>

### 按鈕/互動狀態邏輯
| 操作 | 啟用條件 | 行為 |
|------|----------|------|

### 初始化流程
1. 載入參考資料（下拉選單等）
2. 設定預設值
3. 自動查詢（如適用）

---

## 步驟 4：子元件

### <ComponentName>

**Props / Input**:
| 名稱 | 類型 | 說明 |

**Events / Output**:
| 名稱 | 參數 | 說明 |

**實作要點**:
- <要點描述>

---

## 步驟 5：路由配置

修改路由配置檔，新增：
（路由配置內容）

## 使用的共用組件

| 組件 | 用途 | 參考 |
|------|------|------|
```
