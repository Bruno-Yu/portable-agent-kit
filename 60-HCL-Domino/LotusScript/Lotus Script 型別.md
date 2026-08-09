## Numeric Data Types

|Integer Types| | |
|---|---|---|
|Byte|8 bit (0 - 255) 2 的 8 次方|Unsigned (無負數)|
|Integer|16 bit (-32768 - 32767)|Signed|
|Long|32 bit (-2.1B - 2.1B)|Signed|

|Floating-Point Types| | |
|---|---|---|
|Single|32 bit||
|Double|64 bit||
|Currency|64 bit|fixed-point|
|Decimal|96 bit|hing precision|

|**型別**|**大小**|**儲存方式**|**約可表示有效位數**|**適合用途**|
|---|---|---|---|---|
|Single|32-bit|IEEE 浮點|約 7 位|遊戲、圖形、一般計算|
|Double|64-bit|IEEE 浮點|約 15~16 位 (2的次方)|科學計算、工程|
|Currency|64-bit|固定小數|19 位（固定 4 位小數）|金額|
|Decimal|96-bit|十進位整數 + 縮放|約 28~29 位|財務、高精度計算|

|**VB6**|**C#**| |
|---|---|---|
|Single|`float`|需要節省記憶體，大量圖形或遊戲運算|
|Double|`double`|一般工程、數值分析、預設的浮點型別。|
|Currency|沒有完全對應（通常使用 `decimal`）|VB6 舊系統的金額型別，固定四位小數。|
|Decimal|`decimal`|財務、稅務、價格、精確小數計算的首選。|
||||

```vbnet
Dim a As Single
a = 3.14
a = 1.23E8
```

Suffixes specify the data type of a literal value, allowing more accurate computations:

|Suffix|Data Type|Example|
|---|---|---|
|`&`|Long|12345&|
|`!`|Single|3.14!|
|`#`|Double|3.1415926535#|
|`@`|Currency|19.99@|

Also, enclose **string literals** in quotes and **date literals** within `#` symbols.

這一段其實是在講 **VB6 的 Literal（常值）型別宣告**。

如果你有 C# 背景，可以把它理解成 **C# 的數字後綴 (`L`、`F`、`M`) 的前身**。

例如 C#：

```csharp
123L      // long
3.14f     // float
3.14d     // double
19.99m    // decimal
```

|**VB6**|**C#**|
|---|---|
|`&`|`L`|
|`!`|`f`|
|`#`|`d`|
|`@`|`m`（概念最接近）|

**為什麼要特別指定？**

```vbnet
Dim price As Currency

price = 19.99

' 如果沒有用 @ 這 currency literal
' 流程： 19.99 -> Double -> Currency

' 如果有用 @ 成 19.99@
' 流程： Currency Literal -> Currency Variable 不用轉
```

Date

```vbnet
'Date literal
#2026/07/04#

Dim d As Date
d = #7/4/2026#
```

在 VB6 的年代（1998 左右），編譯器的型別推導能力遠不如現在，因此讓開發者用簡單的符號直接指定常值型別，可以：

1. **避免編譯器猜錯型別。**
2. **減少隱式型別轉換**（例如 Double → Single）。
3. **讓程式更有效率**（當年的硬體資源有限）。

VB 有 Sub, Function

- Sub 沒有回傳值 (VB6 沒有 void 的概念最接近的就是 Sub 沒有回傳值)
- Function 有回傳值

```vbnet
'Private 存取修飾詞 (Access Modifier) 表示只有這個 Form 可以呼叫，其他的 Form 不能呼叫

Private Sub Form_Load()

	Dim memberName As String
	Dim TelNumber As String
	Dim LastDay As Date
	Dim ExpTime As Date

	memberName = "Turban, John"

	TelNumber = "1800-900-888-777"

	LastDay = #12/31/2000#

	ExpTime = #12:00 AM#


	MsgBox "Hello"

End Sub
```

Function 需要回傳值

```vbnet
// VB6 沒有 Return 同等於把函式名稱當成變數
Private Function Add(a As Integer, b As Integer) As Integer
	Add = a + b
End Function
```

回傳多值

1. 回傳 UDT (User Defined Type)

```vbnet
Type Employee
	Name As string
	Age As Integer
End Type

Function GetEmployee() As Emplolyee
```

1. 回傳 Class

```vbnet
Function GetEmployee() As Customer
```

1. ByRef (可以修改多個參數)

```vbnet
Sub GetData(ByRef Name As String, ByRef Age As Integer)
```

## B6 的空值系統與 Variant 動態型別

VB6 的空值概念比較特殊，因為它同時存在**四種**看似相似但語意完全不同的「空」狀態，這也是从 VB 遷移到 C#/.NET 時最容易踩雷的地方。

### 一、四種空值比較

|值|適用型別|意義|判斷方式|
|---|---|---|---|
|`Nothing`|Object 參考型別|物件參考未指向任何實體|`Is Nothing`|
|`Null`|Variant（尤其是 DB 欄位）|**資料庫語意的「未知/缺值」**，非 0、非空字串|`IsNull()`|
|`Empty`|Variant|變數已宣告但**從未被賦值**|`IsEmpty()`|
|`""` (空字串)|String|明確賦值為空字串，是「已知的空」|`= ""`|

#### 關鍵概念：Null 是 SQL 語意的延伸

vb

```vbnet
Dim x As Variant
x = Null

If x = Null Then    ' 永遠是 False！因為任何運算涉及 Null 結果都是 Null
    MsgBox "不會執行到這裡"
End If

If IsNull(x) Then    ' 正確作法
    MsgBox "x 是 Null"
End
```

這跟 SQL 的 `NULL` 邏輯完全一致 —— `NULL = NULL` 在 SQL 裡也不是 `TRUE`，而是 `UNKNOWN`。這也是為什麼 VB6 操作 Recordset 時，欄位是 Null 常常搞死人：

```vbnet
Dim rs As Recordset
' ...
If IsNull(rs("EndDate")) Then
    lblEndDate.Caption = "尚未結束"
Else
    lblEndDate.Caption = rs("EndDate")
End If

If IsNull(rs("")) Then

Else

End I
```

如果直接 `rs("EndDate") & ""` 硬轉字串，雖然能避開錯誤，但邏輯上偷懶（後面會講原因）。

### 二、Variant：VB6 的動態型別容器

`Variant` 是 VB6 唯一的「萬用箱」型別，內部其實是一個帶 tag 的 union，執行期才決定實際存的是什麼。

```vbnet
Dim v As Variant

v = 100          ' VarType(v) = vbInteger (2)
v = "hello"      ' VarType(v) = vbString (8)
v = Null         ' VarType(v) = vbNull (1)
v = Empty        ' VarType(v) = vbEmpty (0)
v = Array(1,2,3) ' VarType(v) = vbArray + vbVariant
```

#### VarType 判斷表

|常數|值|情境|
|---|---|---|
|`vbEmpty`|0|尚未賦值|
|`vbNull`|1|明確賦值 Null|
|`vbInteger` / `vbLong` / `vbDouble`|2/3/5|數值|
|`vbString`|8|字串|
|`vbObject`|9|物件參考|

#### 隱式轉型的地雷

Variant 最大的問題是**運算子重載的自動轉型**，這也是遷移到 .NET 強型別後最常出現行為差異的地方：

vb

```vbnet
Dim v As Variant
v = "5"
Debug.Print v + 1        ' 結果 6（自動轉數值）
Debug.Print v & 1        ' 結果 "51"（強制字串）

Dim v2 As Variant
v2 = Null
Debug.Print v2 + 1       ' 結果仍是 Null（不會報錯，直接往下傳染）
Debug.Print IsNull(v2 & "test")  ' False！& 運算子對 Null 會轉成 ""
```

**這條規則很重要**：`&` 串接運算子會把 Null 當空字串處理，但 `+` 算術運算子遇到 Null 會讓整個運算結果變成 Null（Null propagation，類似 SQL）。這也是很多 VB6 遺留程式碼裡看到 `rs("Field") & ""` 這種寫法的原因——目的是強制把 Null 轉成安全的空字串，避免後續字串處理爆炸。

## 6.2 Essential Operators

Visual Basic uses specific operators for calculations and string operations. Understanding these is crucial for effective programming:

|Operator|Description|Example|Result|
|---|---|---|---|
|`^`|Exponentiation|`3 ^ 4`|81|
|`Mod`|Modulus (Remainder)|`17 Mod 5`|2|
|`\`|Integer Division|`19 \ 4`|4|
|`&`|String Concatenation|`"Visual" & "Basic"`|"VisualBasic"|

```vbnet
Dim itemPrice As Currency

Dim quantity As Integer

Dim totalValue As Currency

Private Sub CalculateTotal()

    On Error Resume Next

    itemPrice = CCur(txtPrice.Text)

    quantity = CInt(txtQty.Text)

    totalValue = itemPrice * quantity



    lblTotal.Caption = Format(totalValue, "Currency")

    If Err.Number <> 0 Then

        MsgBox "Invalid numeric input", vbExclamation

    End If

End Sub

```

## 7.1 Conditional Operators

Conditional operators allow your VB6 programs to compare values and determine program flow. These operators form the foundation of decision-making structures in programming:

|Operator|Meaning|Example|
|---|---|---|
|`=`|Equal to|`If x = y Then`|
|`>`|Greater than|`If x > y Then`|
|`<`|Less than|`If x < y Then`|
|`>=`|Greater than or equal to|`If x >= y Then`|
|`<=`|Less than or equal to|`If x <= y Then`|
|`<>`|Not equal to|`If x <> y Then`|

### Important Note

When comparing strings, uppercase letters are considered "less than" lowercase letters, and numbers are "less than" letters.

## 7.2 Logical Operators

Logical operators allow you to combine multiple conditions to create complex decision structures:

|Operator|Description|Example|
|---|---|---|
|`And`|Both conditions must be true|`If x > 5 And x < 10 Then`|
|`Or`|At least one condition must be true|`If x < 0 Or x > 100 Then`|
|`Xor`|Exactly one condition must be true|`If x > 0 Xor y > 0 Then`|
|`Not`|Reverses the logical value|`If Not x = y Then`|

```vbnet
If condition Then
	...
ElseIf anotherCondition Then
	...
Else
	...
End If
```

```vbnet
Private Sub cmdCalComm_Click()

    Dim salevol, comm As Currency

    salevol = Val(TxtSaleVol.Text)

    If salevol >= 5000 And salevol < 10000 Then

        comm = salevol * 0.05

    ElseIf salevol >= 10000 And salevol < 15000 Then

        comm = salevol * 0.1

    ElseIf salevol >= 15000 And salevol < 20000 Then

        comm = salevol * 0.15

    ElseIf salevol >= 20000 Then

        comm = salevol * 0.2

    Else

        comm = 0

    End If

    LblComm.Caption = Format(comm, "$#,##0.00")

End Sub
```

你如果熟悉 C#，可以把它們理解成：

- `Val()` ≈ `int.Parse()` / `double.Parse()`（但比較寬鬆）
- `Format()` ≈ `ToString("格式"`

|**輸入**|`Val()`|`double.Parse()`|
|---|---|---|
|`"123"`|123|123|
|`"123ABC"`|123|例外 (`FormatException`)|
|`"ABC123"`|0|例外|

**`Format()`**

作用：將數值轉成指定格式的字串。

```vbnet
Format(1234567.8, "$#,##0.00") '$1,234,567.80
```

|**格式**|**意義**|**範例**|
|---|---|---|
|`0`|必須顯示數字|`005`|
|`#`|有數字才顯示|`5`|
|`,`|千分位|`1,234`|
|`.00`|固定兩位小數|`12.30`|

LotusScript 常見的內建函式：

|**函式**|**VB6**|**LotusScript**|**功能**|
|---|---|---|---|
|`Val()`|✅|✅|字串轉數字|
|`Format()`|✅|✅|格式化|
|`Left()`|✅|✅|左邊字串|
|`Right()`|✅|✅|右邊字串|
|`Mid()`|✅|✅|中間字串|
|`Len()`|✅|✅|長度|
|`Trim()`|✅|✅|去空白|
|`UCase()`|✅|✅|大寫|
|`LCase()`|✅|✅|小寫|
|`Instr()`|✅|✅|找字串|

LotusScript 和 VB6 都受到早期 BASIC 語言影響，因此很多函式名稱相同。

因為兩者都屬於 **BASIC 家族**，所以如果你會 VB6，閱讀 LotusScript 通常不會太困難。

```vbnet
Format()

Format(0.356, "0%") '36%

Format(#7/4/2026#, "yyyy/mm/dd") '2026/07/04
```

|`Left()`|✅|✅|左邊字串|
|---|---|---|---|
|`Right()`|✅|✅|右邊字串|

差別在於：

- `Left()`：回傳 **Variant**
- `Left$()`：直接回傳 **String**

在 VB6 時代，`Left$()` 少了一次 `Variant` 包裝，因此**效能比較好**。

```vbnet
Left(string, length) 取左邊 6 個

Left("ABCDEFG", 3)

Right(FileName, 4)

Left$

Right$

```

**`Mid(string, start, length)`**

**VB6 的字串索引是從 1 開始，不是 0。**

```vbnet
Mid("Visual Basic", 3, 6) '"sual B"
```

`InStr([start], string1, string2)`

**B6 的字串索引是從 1 開始，不是 0。**

```vbnet
InStr(1, "Visual Basic", "Basic") '8
```

```vbnet
Str(123.45) → "123.45"

Val("123.45") → 123.45

UCase("Visual Basic") → "VISUAL BASIC"

LCase("Visual Basic") → "visual basic"
```

- Pre-test While Loop

```vbnet
Do While condition
	..
Loop
```

- Post-test While Loop

```vbnet
Do
...

Loop While condition

```

- Pre-test Until Loop

```vbnet
Do Until condition
...
Loop
```

- Post-test Until Loop

```vbnet
Do
...
Loop Until condition
```

## Exiting the Loop

```vbnet
Dim sum, n As Integer

Private Sub Form_Activate()

	List1.AddItem "n" & vbTab & "sum"

	Do

		n = n + 1
		sum = sum + n
		List1.AddItem n & vbTab & Sum

		If n = 100 Then
			Exit Do
		End If
	Loop
End Sub
```

## The For… Next Loop

```vbnet
For counter = startNumber To endNumber [Step increment]

    ' One or more VB statements

Next [counter]
```

```vbnet
For counter = 1000 To 5 Step -5

    counter = counter - 10

    If counter < 50 Then

        Exit For

    Else

        Print "Keep Counting"

    End If

Next
```