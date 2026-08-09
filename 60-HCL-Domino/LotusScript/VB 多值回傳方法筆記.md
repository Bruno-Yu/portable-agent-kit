
## 方法一：使用 `Type`（自訂結構）✅ 最常見

```vb
' 先定義 Type（放在 Module 最上方）
Type PersonInfo
    Name    As String
    Age     As Integer
    Salary  As Double
End Type

' Function 回傳 Type
Function GetPerson() As PersonInfo
    Dim result As PersonInfo
    result.Name   = "Alice"
    result.Age    = 30
    result.Salary = 50000.0
    GetPerson = result
End Function

' 呼叫
Sub Test()
    Dim p As PersonInfo
    p = GetPerson()
    MsgBox p.Name & ", " & p.Age & ", " & p.Salary
End Sub
```

---

## 方法二：使用 `Object`（Dictionary）

```vb
' 使用 Dictionary（需引用 Microsoft Scripting Runtime）
Function GetValues() As Object
    Dim dict As Object
    Set dict = CreateObject("Scripting.Dictionary")
    dict("Name")   = "Bob"
    dict("Age")    = 25
    dict("Salary") = 60000.0
    Set GetValues = dict
End Function

' 呼叫
Sub Test()
    Dim result As Object
    Set result = GetValues()
    MsgBox result("Name") & ", " & result("Age")
End Sub
```

---

## 方法三：使用 `ByRef` 參數回傳多值

```vb
' ByRef 參數可將修改後的值「帶回」呼叫端
Sub GetMultiValues(ByVal input As String, _
                   ByRef outName   As String, _
                   ByRef outAge    As Integer, _
                   ByRef outSalary As Double)
    outName   = "Charlie - " & input
    outAge    = 28
    outSalary = 70000.0
End Sub

' 呼叫
Sub Test()
    Dim n As String
    Dim a As Integer
    Dim s As Double

    GetMultiValues "Test", n, a, s

    MsgBox n & ", " & a & ", " & s
End Sub
```

---

## 方法四：回傳 `Array`（陣列）

```vb
Function GetArray() As Variant
    Dim arr(2) As Variant
    arr(0) = "David"
    arr(1) = 35
    arr(2) = 80000.0
    GetArray = arr
End Function

' 呼叫
Sub Test()
    Dim result As Variant
    result = GetArray()
    MsgBox result(0) & ", " & result(1) & ", " & result(2)
End Sub
```

---

## ⚙️ ByRef 與 ByVal 的差異說明

### 概念比較

| 關鍵字 | 傳遞方式 | 呼叫端變數是否受影響 | 類似概念 |
|--------|----------|----------------------|----------|
| `ByVal` | 傳入變數的**複本** | ❌ 不受影響 | `in` 參數 |
| `ByRef` | 傳入變數的**參考位址** | ✅ 會被修改 | `in/out` 或 `out` 參數 |

> ⚠️ VB 沒有純粹的 `out`（如 C# 的 `out`），但 `ByRef` 可達到相同效果。
> ⚠️ VB 若不明確標示，**預設為 `ByRef`**，建議明確寫出以避免混淆。

---

### ByVal 範例（呼叫端不受影響）

```vb
Sub ChangeValue(ByVal x As Integer)
    x = 999        ' 只改了複本，呼叫端的變數不變
End Sub

Sub Test()
    Dim n As Integer
    n = 10
    ChangeValue n
    MsgBox n       ' 結果仍是 10
End Sub
```

---

### ByRef 範例（呼叫端會被修改）

```vb
Sub ChangeValue(ByRef x As Integer)
    x = 999        ' 直接修改呼叫端的變數
End Sub

Sub Test()
    Dim n As Integer
    n = 10
    ChangeValue n
    MsgBox n       ' 結果變成 999
End Sub
```

---

### 強制使用 ByVal 傳遞（加括號）

> 即使函式宣告為 `ByRef`，呼叫時用**額外括號**包住參數，可強制以 `ByVal` 方式傳遞：

```vb
Sub ChangeValue(ByRef x As Integer)
    x = 999
End Sub

Sub Test()
    Dim n As Integer
    n = 10
    ChangeValue (n)   ' 加括號 → 強制 ByVal，呼叫端不受影響
    MsgBox n          ' 結果仍是 10
End Sub
```

---

## 🏆 使用時機建議

| 方法 | 適合情境 |
|------|----------|
| **`Type`** | 回傳結構固定、語意明確的多個欄位 |
| **`Object / Dictionary`** | 回傳欄位不固定或動態的情況 |
| **`ByRef` 參數** | 簡單快速，不想額外定義型別 |
| **`Array`** | 回傳同型別的多筆資料 |
