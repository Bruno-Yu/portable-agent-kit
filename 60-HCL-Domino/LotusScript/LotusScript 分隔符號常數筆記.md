

---

## 一、為什麼需要分隔符號常數？

在 LotusScript 程式中，常需要將多筆資料**組合成一個字串**儲存，
之後再依據分隔符號**拆解還原**，因此需要定義一組統一的分隔符號常數，
避免硬編碼（Hard-Code）造成維護困難。

> **核心概念：**
> - 程式內部 → 使用特殊符號組合，避免與資料內容衝突
> - UI 顯示層 → 使用易讀符號，方便使用者理解

---

## 二、常數定義

```vbscript
Const SF   = "^$"   ' Field Separator    — 欄位分隔
Const SP   = "^!"   ' Sub-Part Separator — 子項目分隔
Const ST   = "^"    ' Token Separator    — 細粒度分隔
Const SD   = "/"    ' Date/Dir Separator — 日期/路徑分隔

Const uiSF = " # "  ' UI 欄位分隔（對應 SF）
Const uiSP = " * "  ' UI 子項目分隔（對應 SP）
```

---

## 三、分隔層級關係

分隔符號依資料粒度由粗到細分為四層：

```
【粗】 SF  "^$"  ← 區隔不同欄位群組
       ↓
      SP  "^!"  ← 區隔同一欄位內的子項目
       ↓
      ST  "^"   ← 區隔最細的單一值（Token）
       ↓
【細】 SD  "/"   ← 專用於日期或路徑格式
```

對應 UI 顯示符號：

```
SF  "^$"  ←→  uiSF  " # "
SP  "^!"  ←→  uiSP  " * "
ST  "^"   ←→  無對應，直接格式化輸出
SD  "/"   ←→  無對應，本身即為易讀格式
```

---

## 四、各符號實例說明

### ▌SF = "^$"（欄位與欄位之間的分隔）

**使用情境：** 儲存員工的姓名、部門、分機號碼三個欄位

```vbscript
' 【程式儲存】
Dim sEmployee As String
sEmployee = "王小明" & SF & "資訊部" & SF & "1234"
' → "王小明^$資訊部^\$1234"

' 【拆解取用】
Dim arr() As String
arr = Split(sEmployee, SF)
' arr(0) = "王小明"
' arr(1) = "資訊部"
' arr(2) = "1234"

' 【UI 顯示】
Dim sShow As String
sShow = Join(Split(sEmployee, SF), uiSF)
MsgBox sShow
' → 畫面顯示：王小明 # 資訊部 # 1234
```

---

### ▌SP = "^!"（同欄位內，子項目之間的分隔）

**使用情境：** 員工有多個技能，技能分為「程式語言」和「資料庫」兩群組

```vbscript
' 【程式儲存】
' 群組1：程式語言（Java、Python）
' 群組2：資料庫（Oracle、MySQL）
Dim sSkill As String
sSkill = "Java" & SP & "Python" & SF & "Oracle" & SP & "MySQL"
' → "Java^!Python^$Oracle^!MySQL"

' 【拆解取用】
Dim groups() As String
groups = Split(sSkill, SF)
' groups(0) = "Java^!Python"  → 再用 SP 拆 = Java、Python
' groups(1) = "Oracle^!MySQL" → 再用 SP 拆 = Oracle、MySQL

' 【UI 顯示】
Dim sShow As String
sShow = Join(Split(sSkill, SP), uiSP)  ' 先換 SP → " * "
sShow = Join(Split(sShow,  SF), uiSF)  ' 再換 SF → " # "
MsgBox sShow
' → 畫面顯示：Java * Python # Oracle * MySQL
```

---

### ▌ST = "^"（最細粒度的 Token 分隔）

**使用情境：** 訂單內每筆商品，記錄商品名稱、數量、單價

```vbscript
' 【程式儲存】
Dim sOrder As String
sOrder = "鍵盤" & ST & "2" & ST & "500" & SF & "滑鼠" & ST & "1" & ST & "300"
' → "鍵盤^2^500^$滑鼠^1^300"

' 【拆解取用】
Dim items() As String
items = Split(sOrder, SF)       ' 先用 SF 拆出每筆商品

Dim detail() As String
detail = Split(items(0), ST)    ' 再用 ST 拆出細節
' detail(0) = "鍵盤"
' detail(1) = "2"
' detail(2) = "500"

' 【UI 顯示】
MsgBox "商品：" & detail(0) & "，數量：" & detail(1) & "，單價：" & detail(2)
' → 畫面顯示：商品：鍵盤，數量：2，單價：500
```

---

### ▌SD = "/"（日期或路徑的分隔）

**使用情境：** 儲存日期或檔案路徑

```vbscript
' 【程式儲存 — 日期】
Dim sDate As String
sDate = "2025" & SD & "07" & SD & "10"
' → "2025/07/10"

' 【程式儲存 — 路徑】
Dim sPath As String
sPath = "Docs" & SD & "Reports" & SD & "Q2.xlsx"
' → "Docs/Reports/Q2.xlsx"

' 【UI 顯示】
MsgBox "日期：" & sDate
' → 畫面顯示：日期：2025/07/10

MsgBox "路徑：" & sPath
' → 畫面顯示：路徑：Docs/Reports/Q2.xlsx
```

> SD 使用 `/` 本身已是易讀格式，無需額外轉換即可直接顯示。

---

## 五、綜合範例：程式內部 vs UI 呈現對照

```
【程式內部字串】
"鍵盤^2^500^$滑鼠^1^300"
  └─ ST ─┘  └SF┘ └─ ST ─┘

【UI 顯示字串】
"鍵盤 * 2 * 500 # 滑鼠 * 1 * 300"
  └─ uiSP ─┘  └uiSF┘ └─ uiSP ─┘
```

---

## 六、常數彙整表

| 常數  | 程式值 | 說明           | UI 對應 | UI 值 |
|-------|--------|----------------|---------|-------|
| SF    | ^$     | 欄位分隔       | uiSF    | ` # ` |
| SP    | ^!     | 子項目分隔     | uiSP    | ` * ` |
| ST    | ^      | Token 分隔     | 無      | 直接格式化 |
| SD    | /      | 日期/路徑分隔  | 無      | 直接顯示 |

---

## 七、設計原則總結

1. **統一管理** — 所有分隔符號集中定義為常數，修改時只需改一處
2. **避免衝突** — 程式內部使用 `^$`、`^!` 等特殊組合，資料內容幾乎不會包含此符號
3. **層級分明** — SF > SP > ST，由粗到細，結構清晰
4. **顯示友善** — UI 層使用 `#`、`*` 轉換顯示，提升使用者閱讀體驗
