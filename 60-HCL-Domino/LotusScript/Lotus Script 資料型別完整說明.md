

## 📊 基本資料型別（Primitive Types）

| 型別 | 關鍵字 | 範圍／說明 | 範例 |
|------|--------|-----------|------|
| 整數 | `Integer` | -32,768 ~ 32,767 | `Dim x As Integer` |
| 長整數 | `Long` | -2,147,483,648 ~ 2,147,483,647 | `Dim x As Long` |
| 單精度浮點 | `Single` | 約 7 位有效數字 | `Dim x As Single` |
| 雙精度浮點 | `Double` | 約 15 位有效數字 | `Dim x As Double` |
| 貨幣 | `Currency` | 固定小數點 4 位 | `Dim x As Currency` |
| 字串 | `String` | 文字字串 | `Dim s As String` |
| 布林 | `Boolean` | `True` / `False` | `Dim b As Boolean` |
| 日期 | `Date` | 日期時間值 | `Dim d As Date` |
| 位元組 | `Byte` | 0 ~ 255 | `Dim b As Byte` |

---

## 🔮 特殊型別

### `Variant`
- **可存放任何型別的值**，型別在執行期動態決定
- Lotus Script 若**不宣告型別，預設就是 Variant**

```vbscript
Dim v As Variant

v = 123          ' 現在是 Integer
v = "Hello"      ' 現在變成 String
v = True         ' 現在變成 Boolean

' 可用 TypeName() 查看當前型別
Print TypeName(v)   ' 輸出: Boolean
```

---

## 🏛️ Object 型別

### 概念說明

在 Lotus Script 中，Object 型別分為兩大類：

```
Object
├── 1. 自訂 Class 實例        （用 Class 關鍵字自訂）
└── 2. Notes 內建物件         （NotesSession、NotesDocument 等）
```

---

### 1️⃣ 通用 Object 型別宣告（晚期繫結 Late Binding）

```vbscript
' 宣告為泛型 Object（類似 JS 的 let obj）
Dim obj As Object

' 執行期才決定實際型別
Set obj = New MyClass
' 或
Set obj = session.CurrentDatabase
```

> ⚠️ 使用 `As Object` 是**晚期繫結（Late Binding）**，
> 編譯期不會檢查型別，執行期才確認，效能較差但彈性較高。

---

### 2️⃣ 明確型別宣告（早期繫結 Early Binding）

```vbscript
' 明確指定型別（推薦做法）
Dim session As New NotesSession
Dim db As NotesDatabase
Dim doc As NotesDocument
Dim obj As MyClass

Set db = session.CurrentDatabase
Set doc = db.CreateDocument()
Set obj = New MyClass
```

> ✅ 明確宣告型別是**早期繫結（Early Binding）**，
> 編譯期就能做型別檢查，效能較好，也有自動完成提示。

---

### 3️⃣ Notes 常用內建物件一覽

| 物件型別 | 說明 |
|---------|------|
| `NotesSession` | 當前 Notes 工作階段的入口點 |
| `NotesDatabase` | 代表一個 Notes 資料庫（.nsf） |
| `NotesDocument` | 代表一份文件 |
| `NotesView` | 代表資料庫中的一個檢視 |
| `NotesItem` | 代表文件中的一個欄位 |
| `NotesUIWorkspace` | 代表 UI 工作區（前端操作） |
| `NotesUIDocument` | 代表目前開啟的 UI 文件 |
| `NotesDocumentCollection` | 文件集合 |
| `NotesMIMEEntity` | MIME 格式內容處理 |

```vbscript
' 典型的 Notes 物件使用流程
Dim session As New NotesSession
Dim db As NotesDatabase
Dim view As NotesView
Dim doc As NotesDocument

Set db = session.CurrentDatabase
Set view = db.GetView("MyView")
Set doc = view.GetFirstDocument()

Do While Not doc Is Nothing
    Print doc.GetItemValue("Subject")(0)
    Set doc = view.GetNextDocument(doc)
Loop
```

---

## 🔍 物件判斷相關語法

### `Is Nothing` — 判斷物件是否為空

```vbscript
If doc Is Nothing Then
    Print "找不到文件"
Else
    Print "找到文件了"
End If
```

### `TypeName()` — 取得型別名稱

```vbscript
Dim session As New NotesSession
Print TypeName(session)   ' 輸出: NotesSession

Dim x As Integer
Print TypeName(x)         ' 輸出: Integer
```

### `IsObject()` — 判斷是否為物件

```vbscript
Dim obj As New MyClass
Print IsObject(obj)   ' 輸出: True

Dim x As Integer
Print IsObject(x)     ' 輸出: False
```

---

## 📌 物件賦值與釋放

```vbscript
' 建立物件
Set obj = New MyClass

' 物件間互相指派（兩個變數指向同一個物件）
Set obj2 = obj

' 釋放物件參考（設為 Nothing）
Set obj = Nothing
Set obj2 = Nothing
```

---

## 🔄 `Set` 使用規則整理

| 情況 | 語法 | 是否需要 Set |
|------|------|:----------:|
| 一般值（Integer、String...）賦值 | `x = 100` | ❌ |
| 建立新物件實例 | `Set obj = New MyClass` | ✅ |
| 宣告時直接初始化 | `Dim obj As New MyClass` | ❌（語法糖） |
| 物件參考互相指派 | `Set obj2 = obj` | ✅ |
| 釋放物件 | `Set obj = Nothing` | ✅ |

---

## 💡 總結重點

| 分類 | 說明 |
|------|------|
| 一般型別（Integer、String...） | 直接賦值，不用 `Set` |
| 物件型別（Object、Class、Notes 物件） | 必須用 `Set` 賦值 |
| `Variant` | 萬用型別，可裝任何東西，但要小心型別錯誤 |
| `As Object`（泛型） | 晚期繫結，彈性高但效能略差 |
| 明確型別宣告 | 早期繫結，效能好，建議優先使用 |
