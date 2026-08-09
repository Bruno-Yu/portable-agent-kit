**主要來源**

- [HCL Domino Designer 12.0.2 documentation](https://help.hcl-software.com/dom_designer/12.0.2/index.html)
- [Formula Language Rules](https://help.hcl-software.com/dom_designer/12.0.2/basic/H_5_FORMULAS_UNDERSTANDING_THE_NOTES_FORMULA_LANGUAGE.html)
- [List of @Functions](https://help.hcl-software.com/dom_designer/12.0.2/basic/H_FUNCTIONS_LISTED.html)
- [Using the Domino classes](https://help.hcl-software.com/dom_designer/12.0.2/basic/H_USING_THE_NOTES_CLASSES.html)
- [LotusScript Classes Tutorial](https://help.hcl-software.com/dom_designer/12.0.2/basic/H_TEACH_YOURSELF_LOTUSSCRIPT.html)

**第 0 層：先建立 Domino 心智模型**

Domino 不是一般「寫一支程式直接跑」的平台。它的程式通常活在 NSF database 裡，掛在 Form、View、Action、Agent、Button、Event 上執行。

你要先把 Domino 想成這樣：

```
NSF Database
  -> Form 定義資料長什麼樣子
  -> Document 是一筆實際資料
  -> View / Folder 是文件清單與索引
  -> Agent / Action / Button 是觸發程式的地方
  -> Formula / LotusScript 是主要寫邏輯的語言
```

最重要的觀念是：Domino 的程式常常不是從 `main()` 開始，而是從「使用者按了某個按鈕」、「打開某份文件」、「儲存文件」、「Agent 被排程執行」開始。

**第 1 層：Formula Language 先學什麼**

Formula 適合寫短邏輯、欄位計算、選擇條件、View selection、簡單按鈕行為。它不像完整程式語言那麼自由，但在 Domino 裡很常見。

基本語法：

```
FIELD Status := "Open";
@If(Status = "Open"; "處理中"; "已結束")
```

Formula 常見用途：

```
View selection:
SELECT Form = "Task" & Status != "Done"

欄位預設值:
@Name([CN]; @UserName)

按鈕公式:
FIELD Status := "Done";
@Command([FileSave])
```

你必學的 Formula 類型：

|類型|常用函式|使用情境|
|---|---|---|
|條件判斷|`@If`|狀態判斷、欄位顯示|
|使用者資訊|`@UserName`, `@Name`|取得目前登入者|
|文字處理|`@Left`, `@Right`, `@Middle`, `@Contains`|拆字串、搜尋文字|
|日期時間|`@Today`, `@Now`, `@Adjust`|到期日、建立日期|
|欄位操作|`FIELD x := value`|修改文件欄位|
|指令|Command(...)|儲存、關閉、開文件、跑 Agent|

簡單例子：任務過期判斷。

```
@If(DueDate < @Today & Status != "Done"; "Overdue"; "Normal")
```

使用情境：View 裡顯示一個計算欄位，讓使用者看到哪些任務已過期。

**第 2 層：LotusScript 基本語法**

LotusScript 比 Formula 更像 VB。當你需要迴圈、處理多份文件、建立文件、寄信、呼叫 Domino 物件模型時，就用 LotusScript。

基本結構：

```
Sub Click(Source As Button)
    MsgBox "Hello Domino"
End Sub
```

變數：

```
Dim name As String
Dim count As Integer
Dim total As Long

name = "Alice"
count = 3
```

條件：

```
If count > 0 Then
    MsgBox "有資料"
Else
    MsgBox "沒有資料"
End If
```

迴圈：

```
Dim i As Integer

For i = 1 To 5
    Print i
Next
```

錯誤處理基礎：

```
On Error GoTo errHandler

' main logic here
Exit Sub

errHandler:
    MsgBox "Error " & Err & ": " & Error$
```

使用情境：Formula 適合一行判斷，LotusScript 適合「做一批事」。例如：掃過某個 View，把所有逾期文件標成 Overdue。

**第 3 層：Domino 後端物件鏈**

你最該先背起來的是這條鏈：

```
NotesSession
  -> NotesDatabase
    -> NotesView
      -> NotesDocument
        -> NotesItem / fields
```

常用意思：

|物件|代表什麼|
|---|---|
|`NotesSession`|目前執行環境|
|`NotesDatabase`|一個 NSF database|
|`NotesView`|一個 View 或 Folder|
|`NotesDocument`|一份文件|
|`NotesItem`|文件裡的一個欄位資料|

典型範例：取得目前資料庫標題。

```
Dim session As New NotesSession
Dim db As NotesDatabase

Set db = session.CurrentDatabase
MsgBox db.Title
```

典型範例：建立一份新文件。

```
Dim session As New NotesSession
Dim db As NotesDatabase
Dim doc As NotesDocument

Set db = session.CurrentDatabase
Set doc = db.CreateDocument

Call doc.ReplaceItemValue("Form", "Task")
Call doc.ReplaceItemValue("Subject", "Learn LotusScript")
Call doc.ReplaceItemValue("Status", "Open")

Call doc.Save(True, False)
```

使用情境：按一個 Action，自動建立一筆 Task 文件。

**第 4 層：View 與 Document 讀取**

從 View 拿第一筆文件：

```
Dim session As New NotesSession
Dim db As NotesDatabase
Dim view As NotesView
Dim doc As NotesDocument

Set db = session.CurrentDatabase
Set view = db.GetView("Tasks")
Set doc = view.GetFirstDocument

If Not doc Is Nothing Then
    MsgBox doc.GetItemValue("Subject")(0)
End If
```

跑完整個 View：

```
Dim nextDoc As NotesDocument

Set doc = view.GetFirstDocument

While Not doc Is Nothing
    Set nextDoc = view.GetNextDocument(doc)

    Print doc.GetItemValue("Subject")(0)

    Set doc = nextDoc
Wend
```

使用情境：批次檢查所有 Task，把逾期任務更新狀態。

**第 5 層：前端 UI 物件鏈**

另一條鏈是 UI 物件：

```
NotesUIWorkspace
  -> NotesUIDatabase
  -> NotesUIView
  -> NotesUIDocument
```

後端與前端差異：

|類型|用途|
|---|---|
|Backend classes|真正操作 database、view、document，可用於 Agent|
|Frontend UI classes|操作使用者目前畫面，只能在 Notes client/workstation 情境使用|

取得目前打開的文件：

```
Dim ws As New NotesUIWorkspace
Dim uidoc As NotesUIDocument

Set uidoc = ws.CurrentDocument

If Not uidoc Is Nothing Then
    MsgBox uidoc.FieldGetText("Subject")
End If
```

使用情境：使用者正在編輯文件，按按鈕後讀取畫面上的欄位值。

注意：`NotesUIDocument` 是畫面上的文件；`NotesDocument` 是後端資料文件。這兩個很容易混，但非常重要。

**第 6 層：什麼時候用 Formula，什麼時候用 LotusScript**

|需求|建議|
|---|---|
|欄位預設值、簡單顯示判斷|Formula|
|View selection|Formula|
|按鈕只做儲存、設定欄位|Formula|
|需要迴圈處理多份文件|LotusScript|
|建立/修改/寄送文件|LotusScript|
|操作目前 UI 文件|LotusScript + UI classes|
|排程或背景批次作業|LotusScript Agent|

最實用的學習順序我會排成：

1. Formula：`@If`、`FIELD`、`@Command`、`@UserName`、日期與文字函式
2. LotusScript：變數、Sub、Function、If、For、While、錯誤處理
3. Backend：`Session -> Database -> View -> Document`
4. Frontend：`Workspace -> UIDocument`
5. Agent：用 LotusScript 批次處理文件
6. 官方 LotusScript Classes Tutorial 三課：看懂後再做練習題

你現在最該掌握的一句話是：

```
Formula 用來做「短公式」；LotusScript 用來做「流程」；
Backend classes 操作資料；UI classes 操作使用者目前畫面。
```

下一步我建議我幫你把這份教材擴成「Note 12 學習章節版」：每章包含語法、簡單範例、使用情境、常見錯誤、再接 3 到 5 題小練習，但先不附答案。
