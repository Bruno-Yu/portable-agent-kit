

## 變數宣告以下列關鍵字之一開頭：`Dim`、`Static`、`Private`、`Public`

---

## 🔷 各關鍵字說明

### 📌 `Dim`
- 預設行為：**非靜態（nonstatic）** 且 **私有（private）**
- 每次呼叫程序時，變數會**重新初始化**
- 可使用於 Module、Class 或程序（Sub/Function）範圍內

```vbscript
Sub Example()
    Dim x As Integer   ' 每次呼叫 Sub 時，x 重新初始化為 0
    x = x + 1
    Print x            ' 永遠印出 1
End Sub
```

---

### 📌 `Static`
- 變數的值在**程序呼叫之間會被保留**（不會重新初始化）
- ⚠️ **僅可使用於程序（Sub / Function / Property）範圍內**，不可用於 Module 或 Class 層級

```vbscript
Sub Example()
    Static counter As Integer   ' 值會在每次呼叫之間保留
    counter = counter + 1
    Print counter               ' 第1次呼叫印1，第2次印2，依此類推
End Sub
```

---

### 📌 `Public`
- 變數在**宣告它的模組或類別外部也可見（存取）**
- 只要該模組保持載入狀態，變數即持續有效
- ⚠️ **僅可使用於 Module 或 Class 層級**，不可使用於程序（Sub/Function）內部

```vbscript
' Module 層級
Public globalCount As Integer   ' 可被其他 Script 或模組存取

Sub Example()
    globalCount = 100
End Sub
```

---

### 📌 `Private`
- 變數**僅在宣告它的當前範圍（Module 或 Class）內可見**
- 外部無法直接存取
- ⚠️ **僅可使用於 Module 或 Class 層級**，不可使用於程序（Sub/Function）內部

```vbscript
' Module 層級
Private internalValue As String   ' 只有此 Module 內部可存取

Sub Example()
    internalValue = "Hello"
End Sub
```

---

## 📊 使用範圍總整理

| 關鍵字 | 可用於 Module/Class | 可用於 Sub/Function | 值保留於呼叫間 | 外部可見 |
|--------|:-------------------:|:-------------------:|:--------------:|:--------:|
| `Dim` | ✅ | ✅ | ❌ | ❌ |
| `Static` | ❌ | ✅ | ✅ | ❌ |
| `Public` | ✅ | ❌ | ✅* | ✅ |
| `Private` | ✅ | ❌ | ✅* | ❌ |

> *`Public` / `Private` 宣告於 Module 層級時，其生命週期與模組一致，值自然保留。

---

## 💡 重點提醒

> - **`Static`** → 只能在程序內用，解決「想在程序內記住上次的值」的需求
> - **`Public` / `Private`** → 只能在模組或類別層級用，控制「跨模組的可見度」
> - **`Dim`** → 最通用，但預設私有且不保留值，適合一般區域變數使用
