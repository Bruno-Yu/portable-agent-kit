

## 一、引入標頭檔：`%Include`

### 什麼是 `%` 符號？
`%` 在 LotusScript 中是**編譯器指令（Compiler Directive）**的前綴符號。
這類指令不是一般程式碼，而是在**編譯階段**就被處理，程式尚未執行時就生效。

```vbscript
%Include "lserr.lss"
```

- 編譯器會將 `lserr.lss` 的**原始碼內容直接複製貼上**到這個位置
- 類似 C 語言的 `#include`
- `lserr.lss` 裡定義了所有預定義錯誤常數（如 `ErrFileNotFound`、`ErrOpenFailed` 等）
- 若未引入，這些常數無法被識別，編譯時會報錯

### `%Include` vs `using` / `import`

| | `%Include`（LotusScript） | `using`（C#）/ `import`（Java/Python） |
|---|---|---|
| **運作方式** | 直接把檔案內容「複製貼上」進來（文字替換） | 引用已編譯好的外部模組/命名空間 |
| **引用對象** | `.lss` 純文字原始碼檔案 | 已編譯的 library / package |
| **類比** | 更像 C 的 `#include` | 才是真正的 library 引用 |
| **重複引用** | 內容會被重複貼入，需注意 | 通常有防重複機制 |

> ⚠️ `%Include` 只能引用 `.lss` 原始碼檔案，若要引用外部 Java library 或 COM 元件，
> 需使用其他機制（如 `Uselsx`、`Use`）。

### 常見可引用的 `.lss` 檔案

```vbscript
%Include "lserr.lss"      ' LotusScript 內建錯誤常數定義檔
%Include "lsconst.lss"    ' LotusScript 內建一般常數定義檔
%Include "myUtils.lss"    ' 自己撰寫的共用函數檔案
```

---

## 二、編譯期指令 `%If` vs 執行期 `If`

`%` 開頭的指令都是**編譯階段**處理，與一般執行期的指令有本質差異：

| | `%If` / `%Else` / `%End If` | `If` / `Else` / `End If` |
|---|---|---|
| **執行時間** | 編譯階段（程式還沒跑） | 執行階段（程式實際運行時） |
| **用途** | 決定哪些程式碼要被**編譯進去** | 決定哪些程式碼要被**執行** |
| **條件內容** | 只能用編譯期已知的常數/環境變數 | 可以用任何變數、運算結果 |
| **類比語言** | C 的 `#ifdef` / `#if` | 一般程式語言的 `if` |

```vbscript
' %If — 編譯階段決定，只有符合條件的程式碼才會被編譯進去
%If DEBUGMODE Then
    Msgbox "除錯模式：變數值 = " & x
%Else
    ' 正式環境，上面那行根本不會存在於編譯結果中
%End If

' If — 執行階段決定，兩個分支都已編譯進去，只是選擇執行哪條路
If isDebug = True Then
    Msgbox "除錯模式"
End If
```

> 💡 **簡單記法**：
> - **有 `%`** → 給**編譯器**看的，影響「哪些程式碼存在」
> - **無 `%`** → 給**CPU**看的，影響「程式碼執行哪條路」

---

## 三、什麼是 Error Constants？

Error Constants 是 LotusScript 提供的**預定義錯誤常數**，
讓開發者用**有意義的名稱**來捕捉特定錯誤，而不必記住錯誤編號。

### 常見預定義錯誤常數

| 常數名稱 | 說明 |
|---|---|
| `ErrOpenFailed` | 檔案無法開啟（找不到、被鎖定或已損毀） |
| `ErrFileNotFound` | 找不到指定的檔案 |
| `ErrObjectVariableNotSet` | 物件變數從未被實例化，直接使用屬性或方法時觸發 |

### `ErrObjectVariableNotSet` 說明

就是 Object 變數**宣告了但從未被賦值或 `New`**，就直接存取其屬性或方法：

```vbscript
' ❌ 錯誤：只宣告，未實例化
Dim db As NotesDatabase
Call db.Open("", "test.nsf")   ' 💥 觸發 ErrObjectVariableNotSet

' ✅ 正確：宣告並實例化
Dim db As New NotesDatabase("", "test.nsf")
' 或
Set db = New NotesDatabase("", "test.nsf")
```

---

## 四、Error Handling 語法

### 基本語法

```vbscript
On Error errorConstant Goto label
```

### `Resume` 系列指令

`Resume` 系列指令用於錯誤處理結束後控制程式流程：

| 指令 | 行為 |
|---|---|
| `Resume` | 回到**發生錯誤的同一行**重新執行 |
| `Resume Next` | 回到**發生錯誤的下一行**繼續執行 |
| `Resume label` | 跳到**指定標籤**繼續執行 |

---

## 五、錯誤資訊內建變數：`Err`、`Erl`、`Error$`

這三個是 LotusScript 中用於**取得錯誤詳細資訊**的內建函數/變數，
通常在 `On Error` 的錯誤處理區塊內使用。

| 變數 | 類型 | 用途 |
|---|---|---|
| `Err` | Integer | 返回當前錯誤的**錯誤編號** |
| `Erl` | Long | 返回發生錯誤的**原始碼行號** |
| `Error$` | String | 返回描述錯誤的**文字訊息** |

```vbscript
ErrorHandler:
    MsgBox "錯誤編號：" & Err      ' 例如：13 代表型別不符
    MsgBox "發生行號：" & Erl      ' 例如：第 25 行
    MsgBox "錯誤描述：" & Error$   ' 例如："Type mismatch"
    Resume Next
```

### ⚠️ 重要注意事項

| 項目 | 說明 |
|---|---|
| **清除時機** | 一旦執行 `Resume` 陳述式，三個變數的資訊會**立即被清除** |
| **恢復後狀態** | `Resume` 後，`Err`、`Erl`、`Error$` 均會**回傳空值或 0** |
| **下次錯誤** | 直到**下一個錯誤發生**，這些變數才會再次有值 |

> 💡 **建議**：在執行 `Resume` 前，先將這三個變數的值儲存到自訂變數中，以便後續記錄或分析。

---

## 六、實際使用範例

### 範例一：使用預定義常數

```vbscript
' (Options)
%Include "lserr.lss"

' (Click)
Sub Click(source As Button)
    On Error ErrFileNotFound Goto noFileFound
    On Error ErrOpenFailed Goto openFailed

    Call db.Open("", "discuss.nsf")

    Exit Sub   ' ← 正常流程結束，隔開下方錯誤處理區塊

noFileFound:
    Messagebox "Could not find database file."
    Resume Next

openFailed:
    Messagebox "Could not open database file."
    Resume Next

End Sub
```

### 範例二：使用自定義常數

```vbscript
CONST OutOfRange = 99037

Sub ValidateNumber(num As Integer)
    On Error OutOfRange Goto handleError

    If num < 0 Or num > 100 Then
        Error OutOfRange, "Number out of range"
    End If

    Msgbox "數值正常：" & num

    Exit Sub   ' ← 正常流程結束，隔開下方錯誤處理區塊

handleError:
    Messagebox "發生錯誤：數值超出範圍，錯誤碼：" & Err
    Resume Next

End Sub
```

> 💡 流程說明：
> ```
> num 在範圍內 → If 不成立 → Msgbox 正常 → Exit Sub（結束，不執行 handleError）
>
> num 超出範圍 → If 成立
>              → Error OutOfRange 觸發
>              → 跳至 handleError:
>              → Messagebox 顯示錯誤訊息
>              → Resume Next 回到 Error 那行的下一行繼續
> ```

### 範例三：綜合使用 `Err`、`Erl`、`Error$`

```vbscript
Sub TestErrorHandling()
    On Error Goto ErrHandle

    Dim x As Integer
    x = 1 / 0   ' 故意製造除以零的錯誤

    Exit Sub

ErrHandle:
    ' 先儲存錯誤資訊，避免 Resume 後被清除
    Dim errCode As Integer
    Dim errLine As Long
    Dim errMsg As String

    errCode = Err
    errLine = Erl
    errMsg = Error$

    Dim msg As String
    msg = "錯誤編號：" & errCode & Chr(10)
    msg = msg & "發生行號：" & errLine & Chr(10)
    msg = msg & "錯誤描述：" & errMsg
    MsgBox msg

    Resume Next   ' 執行後，Err / Erl / Error$ 將被清除
End Sub
```

---

## 七、總結對照表

| 概念 | 說明 |
|---|---|
| `%Include` | 編譯期將外部 `.lss` 檔案內容貼入，類似 C 的 `#include` |
| `%If` / `%Else` | 編譯期條件判斷，決定哪些程式碼被編譯進去 |
| `If` / `Else` | 執行期條件判斷，決定程式執行哪條路 |
| 預定義 Error Constants | 引入 `lserr.lss` 後可用，以有意義名稱捕捉系統錯誤 |
| 自定義 Error Constants | 用 `CONST` 自訂編號（建議 99000+），搭配 `Error` 手動觸發 |
| `Err` | 取得當前錯誤編號 |
| `Erl` | 取得錯誤發生的行號 |
| `Error$` | 取得錯誤的文字描述 |
| `Resume Next` | 錯誤處理完畢後，從錯誤行的下一行繼續執行 |
| `Exit Sub` | 隔開正常流程與錯誤處理區塊，防止誤執行 |
