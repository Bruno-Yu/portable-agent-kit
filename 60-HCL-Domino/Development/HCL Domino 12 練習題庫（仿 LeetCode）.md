

> 依你的入手順序設計：①LotusScript + Formula 語法 → ②Notes 物件模型存取鏈（重點）→ ③Designer 建 .nsf 做 CRUD → ④Java agent / XPages 再補。
>
> 用法建議：先讀題目區、自己寫，卡住看 Hint，最後對附錄的參考解。**每題標了能不能 headless 驗證**，方便你接上 ODP + Dockerized Domino 的流程；純語法題用一個 throwaway agent 跑 `Print` 就能驗。

---

## Phase 0 — 怎麼在沒有 Designer 的情況下驗證答案

你的 ODP/VSCode/Docker Domino 流程跑通前，最快的「練習用」執行管道有三種，由輕到重：

1. **VM Designer 裡的 Agent（最快上手）**：建一個 LotusScript Agent，type 設 `Run once (@Commands may not be used)` 或 manual，把練習碼丟進 `Initialize`，`Print` 輸出會進 server console / Agent Log。語法題、物件鏈題都靠這個。
2. **Headless ODP 編譯（你的目標流程）**：NSF ODP Tooling 在 Dockerized Linux Domino 上做 compile，驗證「能不能編過」+ agent 執行結果寫進一個 log 文件。語法 / API 正確性題適合放這條，因為編譯期就能抓型別與類別名錯誤。
3. **Notes Client 跑表單**：Phase 3 的 front-end（NotesUIDocument）題目一定要真的開表單，這條無法 headless，乖乖在 VM 裡點。

> 標記說明：`[H]` = 可 headless 驗證（編譯 + agent print）；`[UI]` = 需要 Notes client 開表單；`[F]` = Formula，放在表單/欄位/view 上才有意義。

---

## Phase 1 — LotusScript 語法 + Formula `@函式`（預計 1–2 天）

目標不是背完，是把「LeetCode 會用到的基本動作」在 LS / Formula 裡找到對應寫法：字串切割、陣列、迴圈、條件、查表。

### 1A. LotusScript 語法drills

---

**LS-1 `[H]` Easy｜測試：Variant 與型別** 任務：宣告一個未指定型別的變數，依序指派整數、字串、陣列，每次用 `TypeName()` 印出當下型別。 驗收：輸出依序出現 `INTEGER`、`STRING`、再來是陣列型別。 Hint：`Dim x` 不接 `As` 就是 Variant。

---

**LS-2 `[H]` Easy｜測試：字串函式** 任務：給定 `s = "North,ExampleCorp,IT,QA"`，用**兩種方法**各自取出第 3 個欄位 `"IT"`：(a) 用 `StrToken`，(b) 用 `Split` 後索引。 驗收：兩種都印出 `IT`。 Hint：`StrToken(s, ",", 3)`；`Split(s, ",")(2)`（注意 Split 是 0-based）。

---

**LS-3 `[H]` Easy｜測試：動態陣列與 ReDim Preserve** 任務：寫一個 sub，傳入一個整數 n，建立一個 `0 To n-1` 的字串陣列，內容為 `"item0".."item(n-1)"`；接著把它擴充成兩倍長度但**保留原值**，新增的填 `"new"`。 驗收：印出全部，前半是 item，後半是 new。 Hint：`ReDim Preserve` 只能改最後一個維度的上界。

---

**LS-4 `[H]` Medium｜測試：LS List（鍵值集合，很 Domino 特色）** 任務：用 `List` 統計一段字 `"a b a c b a"`（空白分隔）裡每個字出現次數。 驗收：印出 `a=3 b=2 c=1`（順序不拘）。 Hint：

```
Dim cnt List As Integer
ForAll w In Split(theString, " ")
  If IsElement(cnt(w)) Then cnt(w) = cnt(w) + 1 Else cnt(w) = 1
End ForAll
ForAll n In cnt
  Print ListTag(n) & "=" & n
End ForAll
```

---

**LS-5 `[H]` Medium｜測試：ForAll 的「參考」特性** 任務：給一個整數陣列 `arr = (1,2,3,4)`，用 `ForAll` 把每個元素就地平方（不要另開陣列）。 驗收：`arr` 變成 `(1,4,9,16)`。 Hint：`ForAll e In arr : e = e * e : End ForAll` —— ForAll 的迴圈變數是元素的 reference，直接改會寫回原陣列，這跟 For-index 不同，很常被考。

---

**LS-6 `[H]` Medium｜測試：Select Case 與字串比對** 任務：寫一個 function `Grade(score As Integer) As String`，90+→A、80–89→B、70–79→C、其餘→F，用 `Select Case` 的範圍語法。 驗收：`Grade(85)="B"`、`Grade(70)="C"`、`Grade(40)="F"`。 Hint：`Case 90 To 100`、`Case Is >= 80`。

---

**LS-7 `[H]` Hard｜測試：錯誤處理 On Error** 任務：寫一個會故意除以零的 sub，用 `On Error Goto` 攔截，印出錯誤碼、錯誤訊息、發生行號，然後 `Resume Next` 繼續執行後面的 `Print "done"`。 驗收：先印錯誤資訊（Err=11 之類），再印 `done`。 Hint：`Err`、`Error$`、`Erl`；handler 末尾 `Resume Next`；正常路徑結尾要 `Exit Sub` 才不會掉進 handler。

---

**LS-8 `[H]` Hard｜測試：日期時間運算** 任務：用 `NotesDateTime` 算出「今天往後第 45 個工作日的日期」（只跳過六日，不考慮國定假日）。 驗收：印出一個日期字串。 Hint：`Dim dt As New NotesDateTime(Now)`，迴圈裡 `dt.AdjustDay(1)`，用 `Weekday(CDat(dt.LSLocalTime))` 判斷 1(日)/7(六) 略過，計數到 45。

---

### 1B. Formula `@函式` drills

> 這些放在「欄位的 Default Value / Input Validation」、「View column」、或 Action button 上才有意義。先用一個 throwaway form 練。

---

**F-1 `[F]` Easy｜測試：@條件與字串** 任務：一個 computed 欄位，值為 `FirstName` + 空格 + `LastName`，但任一為空就只顯示有值的那個、不要多餘空白。 Hint：`@Trim(FirstName + " " + LastName)` —— `@Trim` 會清掉頭尾與多餘的中間空白。

---

**F-2 `[F]` Easy｜測試：@Word / @Explode** 任務：欄位 `FullPath = "a/b/c/d"`，用 formula 取出最後一段 `d`。 Hint：`@Word(FullPath; "/"; @Elements(@Explode(FullPath; "/")))`，或更簡單 `@Subset(@Explode(FullPath; "/"); -1)`。

---

**F-3 `[F]` Medium｜測試：Input Validation** 任務：欄位 `Amount` 的 Input Validation：必須為正數，否則跳錯誤訊息「Amount must be > 0」並擋下存檔。 Hint：`@If(Amount > 0; @Success; @Failure("Amount must be > 0"))`。

---

**F-4 `[F]` Medium｜測試：@DbColumn 取下拉選單** 任務：一個 dialog list 欄位，選項來自同一個 DB 裡名為 `(LkProjects)` view 的第 1 欄。 Hint：`@DbColumn("Notes":"NoCache"; ""; "(LkProjects)"; 1)`；`""` 代表當前 DB。括號開頭的 view name 表示是隱藏的 lookup view（慣例）。

---

**F-5 `[F]` Medium｜測試：@DbLookup 連動帶值** 任務：使用者選了 `ProjectCode`，自動帶出 `ProjectOwner`。對照表在 `(LkProjects)`，第 1 欄是 code、第 3 欄是 owner。 Hint：`@DbLookup("Notes":"NoCache"; ""; "(LkProjects)"; ProjectCode; 3)`；正式碼要包 `@IsError` 處理查無資料。

---

**F-6 `[F]` Hard｜測試：Formula 迴圈 @For** 任務：欄位 `Numbers` 是多值文字（如 `"3":"5":"8"`），用 `@For` 算總和回傳。 Hint：

```
nums := @TextToNumber(Numbers);
sum := 0;
@For(i := 1; i <= @Elements(nums); i := i + 1;
   sum := sum + nums[i]);
sum
```

進階反思：其實 `@Sum(@TextToNumber(Numbers))` 一行就好——體會 Formula 的「集合運算優先於迴圈」哲學，這跟 LotusScript 的思路不同。

---

## Phase 2 — Notes 物件模型存取鏈（**你的重點**）

先把這條鏈刻進腦子，後面所有題目都是它的變形：

```
NotesSession                         ← 程式進入點（一定有）
  └─ NotesDatabase                   ← 一個 .nsf
       ├─ NotesView                  ← 有索引、有排序的「視圖」
       │    ├─ NotesViewEntry        ← view 裡的一列（含 ColumnValues，不用開文件就拿得到值！）
       │    └─ NotesViewNavigator    ← 高效逐列走訪
       ├─ NotesDocumentCollection    ← 一批文件（Search / FTSearch / DQL 的結果）
       └─ NotesDocument              ← 一筆文件（= 一份 record）
            └─ NotesItem             ← 文件裡的一個欄位（含值、型別、旗標）

front-end（只在 Notes client 有效）：
NotesUIWorkspace
  └─ NotesUIDocument                 ← 使用者「正開著」的那份文件
       └─ .Document → NotesDocument  ← 從前端跳回後端的橋
```

**心智模型三個關鍵**（面試也常問）：

- **back-end vs front-end**：agent / 排程 / web 大多走 back-end（無 UI）；表單事件（QuerySave、Click）才有 front-end。`uidoc.Document` 是兩邊的橋。
- **「取值」三條路、效能差很多**：① `view entry 的 ColumnValues`（不開文件，最快）② `doc.GetItemValue`（要把文件載入記憶體）③ FTSearch / DQL（走索引）。逐筆 `GetItemValue` 掃全庫是新手最大效能坑。
- **GetItemValue 永遠回傳陣列**：單值欄位也要 `doc.GetItemValue("X")(0)`。

---

**OM-1 `[H]` Easy｜走完整條鏈** 任務：在 agent 裡，從 `NotesSession` 拿到 `CurrentDatabase`，印出它的 `Title`、`FilePath`、以及文件總數。 驗收：印出三個值。 Hint：`session.CurrentDatabase`、`db.Title`、`db.FilePath`、`db.AllDocuments.Count`。

---

**OM-2 `[H]` Easy｜GetView + 逐筆走訪** 任務：開一個既有 view（如 `"MainView"`），用 `GetFirstDocument` / `GetNextDocument` 走訪，印出每筆的 `Subject` 欄位。 驗收：逐行印出 subject。 Hint：

```
Set doc = view.GetFirstDocument
Do Until doc Is Nothing
  Print doc.GetItemValue("Subject")(0)
  Set doc = view.GetNextDocument(doc)
Loop
```

---

**OM-3 `[H]` Medium｜ColumnValues vs GetItemValue（效能對照）** 任務：同一個 view，用兩種方式各印出第 1 欄的值：(a) 走 `NotesViewEntry.ColumnValues`，(b) 走 `entry.Document.GetItemValue`。在註解寫出哪個比較快、為什麼。 驗收：兩種輸出相同；註解講對「ColumnValues 不需把整份文件載入記憶體」這個重點。 Hint：用 `view.CreateViewNav` 取 `NotesViewNavigator`，`nav.GetFirst` / `GetNext` 拿 `NotesViewEntry`；`entry.ColumnValues(0)`。

---

**OM-4 `[H]` Medium｜GetItemValue 回傳陣列的陷阱** 任務：給一個有「多值欄位」`Categories`（如 3 個值）和「單值欄位」`Status` 的文件，正確印出：Categories 全部（逗號連接）、Status 單一字串。 驗收：Categories 印出 `A,B,C`；Status 印出單值，不能印出 `Variant` 或陣列表示。 Hint：`Join(doc.GetItemValue("Categories"), ",")`；`doc.GetItemValue("Status")(0)`。

---

**OM-5 `[H]` Medium｜建立並儲存文件（CRUD 的 C）** 任務：在 agent 裡 `CreateDocument`，設 `Form="Task"`、`Subject="練習"`、`Created=Now`，存檔。 驗收：跑完後 view 裡多一筆。 Hint：`Set doc = db.CreateDocument`、`doc.ReplaceItemValue "Form", "Task"`、`doc.Save(True, False)`。 反思：`doc.Form = "Task"` 這種「點欄位名」寫法也行，但 `ReplaceItemValue` 比較安全（明確、可放變數名）。

---

**OM-6 `[H]` Medium｜ReplaceItemValue 與 ComputeWithForm** 任務：撈出所有 `Status="Open"` 的文件，把 `Priority` 統一改成 `"High"` 並存檔；存檔前呼叫 `ComputeWithForm` 讓表單上的 computed 欄位重算。 驗收：受影響文件的 Priority 都變 High。 Hint：用 `db.Search("Form=""Task"" & Status=""Open""", Nothing, 0)` 拿 collection；`doc.ComputeWithForm(False, False)`。

---

**OM-7 `[H]` Medium｜三種查詢方式對照（核心觀念題）** 任務：對「找出 `Form="Task"` 且 `Status="Open"`」這個需求，分別用以下三種寫出來，並在註解比較適用場景：

- (a) `db.Search(formula, Nothing, 0)`
- (b) `db.FTSearch(query, 0)`（需要 DB 有全文索引）
- (c) `db.CreateDominoQuery` + DQL `Form = 'Task' and Status = 'Open'` 驗收：三段都能拿到 collection；註解講對：Search 對每筆套 formula（無索引、量大時慢）；FTSearch 走全文索引（快但要先建索引、且是「搜尋」語意）；DQL 是 v10+ 現代做法、能用 view/索引、語法乾淨。 Hint（DQL）：

```
Dim dq As NotesDominoQuery
Set dq = db.CreateDominoQuery()
Dim col As NotesDocumentCollection
Set col = dq.Execute("Form = 'Task' and Status = 'Open'")
```

---

**OM-8 `[H]` Hard｜DocumentCollection.StampAll 批次更新** 任務：把 OM-6 改寫成**不逐筆開文件**的版本：用 collection 的 `StampAll` 一次把 `Priority` 蓋成 `"High"`。在註解說明它跟逐筆 `GetNextDocument + Save` 的差異。 驗收：結果相同，但碼裡沒有逐筆迴圈。 Hint：`col.StampAll "Priority", "High"`。註解重點：StampAll 在 server 端直接寫，不把每份文件載進記憶體，大量更新時快很多；代價是不會觸發 ComputeWithForm、也不重算 computed 欄位。

---

**OM-9 `[H]` Hard｜主從文件（Response 階層）** 任務：建一筆 `Form="Task"` 的主文件，再建兩筆 `Form="Comment"` 的回應文件掛在它底下；接著從主文件用 `Responses` 走訪、印出每筆 comment 的內容。 驗收：印出 2 筆 comment。 Hint：回應文件 `Call resp.MakeResponse(parentDoc)` 再 `resp.Save`；主文件 `Set rc = parentDoc.Responses`（NotesDocumentCollection），走訪它。

---

**OM-10 `[UI]` Hard｜front-end ↔ back-end 的橋** 任務：在表單的一個 Action button（LotusScript）裡：取得使用者正開著的文件，讀前端某欄位「未存檔的當前輸入值」，把它寫進另一個欄位並 refresh，全程不存檔。 驗收：按鈕按下後，目標欄位即時出現來源欄位的值。 Hint：

```
Dim ws As New NotesUIWorkspace
Dim uidoc As NotesUIDocument
Set uidoc = ws.CurrentDocument
Dim v As String
v = uidoc.FieldGetText("Source")      ' 讀「畫面上」的值（含未存檔）
Call uidoc.FieldSetText("Target", v)
Call uidoc.Refresh
```

反思：為什麼用 `FieldGetText` 而不是 `uidoc.Document.GetItemValue`？因為後者讀的是**已存進後端**的值，使用者剛打的字還沒進去。這個區別是 front/back-end 心智模型的試金石。

---

## Phase 3 — Designer 建一個小 .nsf 做 CRUD（對應官方 tutorial）

這是一個分階段的 mini-project，不是單題。主題：**Task Tracker**。完成它你就走完一輪完整的 Notes 應用開發。建議照官方 LotusScript Classes Tutorial 的節奏，但用自己的題目。

> 這階段大多是 `[UI]`，要在 VM 的 Designer + Notes client 裡做。

**里程碑 M1 — 資料模型（Form）**

- 建 form `Task`，欄位：`Subject`(Text, 必填)、`Status`(Dialog list：Open/In Progress/Done，預設 Open)、`Priority`(Radio：Low/Med/High)、`DueDate`(Date/Time)、`Owner`(Names 或 Text)。
- 加 Input Validation：Subject 不可空（`@If(Subject=""; @Failure("Subject required"); @Success)`）。
- 驗收：能 compose 一筆並存檔。

**里程碑 M2 — 視圖（View）**

- 建 view `(All Tasks)`：欄位顯示 Subject / Status / Priority / DueDate；依 DueDate 升冪排序；DueDate 已過期且未 Done 的列，用 view column 的 formula 加一個 `!` 標記。
- 建 categorized view `(By Status)`：依 Status 分類。
- 驗收：兩個 view 都正確顯示、排序、分類。

**里程碑 M3 — CRUD agents（back-end，可接你的 ODP 流程）**

- **Create**：agent「Quick Add 5 Tasks」批次建 5 筆測試資料。
- **Read/Report**：agent 走訪 `(All Tasks)`，把逾期未完成的清單 `Print` 出來（或寫進一份 report 文件）。
- **Update**：agent「Close Done-able」把 DueDate 已過、Status=In Progress 的批次改成 Done（練 `StampAll` vs 逐筆兩種寫法）。
- **Delete**：agent「Purge Old Done」把 Status=Done 且 DueDate 超過 90 天的 `doc.Remove(True)`。
- 驗收：四個 agent 都能在 server / client 跑出預期結果。

**里程碑 M4 — front-end 互動（Notes client）**

- 在 `Task` form 加 Action button「Mark Done」：把 Status 設成 Done、寫入 `CompletedDate=@Now`、存檔關閉（front-end LotusScript：`uidoc.Document`、`uidoc.Save`、`uidoc.Close`）。
- 加 QuerySave event：Status=Done 但 DueDate 為空時擋下並提示。
- 驗收：按鈕與驗證都如預期。

**里程碑 M5（接你的真實流程）— 匯出成 ODP 進 git**

- 用 NSF ODP Tooling 把這個 .nsf 匯出成 On-Disk Project，commit 進 git，再從 ODP headless 編譯回 .nsf，確認能 round-trip。
- 驗收：ODP → 編譯 → 跑 M3 的 agent，結果與 Designer 內一致。這步驗證的是**你真正要用的 CI/CD 工作流**，不是 Domino 本身。

---

## Phase 4 — Java agent vs XPages（等專案揭曉再深入）

先建立「什麼時候用哪個」的判斷，題目偏向定位與小實作。

**判斷速查：**

- **LotusScript agent**：Notes client 邏輯、排程後端作業、表單事件。最通用的起點。
- **Java agent**：需要外部 library（JDBC、HTTP client、JSON、加密）、或重運算、或要跟 Java 生態整合時。API 跟 LS 的 Notes 類別幾乎一一對應（`Session`、`Database`、`View`、`Document`），學了 LS 物件鏈幾乎無痛轉。
- **XPages**：要做「瀏覽器存取」的現代 web UI，元件化、走 server-side JavaScript / Java backing bean。是另一個世界，學習曲線最陡。

---

**JX-1 `[H]` Easy｜Java agent 對應物件鏈** 任務：把 OM-2（走 view 印 Subject）用 **Java agent** 重寫。 驗收：輸出與 LS 版相同。 Hint：`Session session = getSession(); Database db = session.getCurrentDatabase(); View view = db.getView("MainView"); Document doc = view.getFirstDocument();` —— 注意 Java 版要自己 `recycle()` 物件、且 `getItemValue` 回傳 `Vector`。

---

**JX-2 `[H]` Medium｜Java 的 recycle 觀念** 任務：把 JX-1 加上正確的 `recycle()`：在 `while` 迴圈裡，拿到 `nextDoc` 後 recycle 掉 `doc`。在註解說明為什麼 LS 不用、Java 要用。 Hint：Java 端的 Notes 物件是 C API handle 的 wrapper，GC 不會自動釋放底層 handle，長迴圈不 recycle 會記憶體爆掉；LS 有自己的 backend 物件管理所以不需手動。

---

**JX-3 `[F/UI]` Medium｜XPages 最小資料綁定** 任務：建一個 XPage，用 `xp:viewPanel` 綁到 `(All Tasks)` view，顯示 Subject / Status 兩欄。 驗收：瀏覽器開得出列表。 Hint：`xp:dominoView` data source 指 `viewName="(All Tasks)"`；`xp:viewPanel` 的 `value="#{view1}"`。

---

**JX-4 `[UI]` Hard｜XPages CRUD 一頁** 任務：做一個 XPage 能新增 / 編輯單筆 Task：`xp:dominoDocument` data source + `xp:inputText` 綁 `Subject`、`xp:comboBox` 綁 `Status`，一個 Save 按鈕用 simple action `Save Document`。 驗收：能在瀏覽器建立並存一筆，回 Notes client 看得到。 Hint：data source 的 `formName="Task"`、`action="editDocument"`；Save 按鈕 `xp:eventHandler` + `xp:saveDocument`。

---

# 附錄：參考解

> 先自己寫，真的卡住再看。只列關鍵碼，省略 `Sub Initialize` 等樣板。

### LS-1

```
Dim x
x = 5            : Print TypeName(x)   ' INTEGER
x = "hello"      : Print TypeName(x)   ' STRING
x = Array(1,2,3) : Print TypeName(x)   ' VARIANT( )  ← 陣列
```

### LS-2

```
Dim s As String : s = "North,ExampleCorp,IT,QA"
Print StrToken(s, ",", 3)   ' CIM
Print Split(s, ",")(2)      ' CIM（0-based）
```

### LS-3

```
Sub Build(n As Integer)
  Dim arr() As String
  ReDim arr(0 To n-1)
  Dim i As Integer
  For i = 0 To n-1 : arr(i) = "item" & i : Next
  ReDim Preserve arr(0 To 2*n-1)
  For i = n To 2*n-1 : arr(i) = "new" : Next
  ForAll e In arr : Print e : End ForAll
End Sub
```

### LS-4

```
Dim cnt List As Integer
ForAll w In Split("a b a c b a", " ")
  If IsElement(cnt(w)) Then cnt(w) = cnt(w)+1 Else cnt(w) = 1
End ForAll
ForAll n In cnt : Print ListTag(n) & "=" & n : End ForAll
```

### LS-5

```
Dim arr(0 To 3) As Integer
arr(0)=1 : arr(1)=2 : arr(2)=3 : arr(3)=4
ForAll e In arr : e = e*e : End ForAll   ' 就地寫回
ForAll e In arr : Print e : End ForAll   ' 1 4 9 16
```

### LS-6

```
Function Grade(score As Integer) As String
  Select Case score
  Case 90 To 100 : Grade = "A"
  Case 80 To 89  : Grade = "B"
  Case 70 To 79  : Grade = "C"
  Case Else      : Grade = "F"
  End Select
End Function
```

### LS-7

```
Sub Demo
  On Error Goto h
  Dim a As Integer, b As Integer
  b = 0 : a = 5 / b
  Exit Sub
h:
  Print "Err=" & Err & " " & Error$ & " @line " & Erl
  Resume Next   ' 跳過出錯行繼續
End Sub
' 末尾 Print "done" 仍會執行
```

### LS-8

```
Dim dt As New NotesDateTime(Now)
Dim added As Integer : added = 0
Do While added < 45
  Call dt.AdjustDay(1)
  Dim wd As Integer : wd = Weekday(CDat(dt.LSLocalTime))  ' 1=Sun,7=Sat
  If wd <> 1 And wd <> 7 Then added = added + 1
Loop
Print dt.DateOnly
```

### F-1 ～ F-6

- F-1：`@Trim(FirstName + " " + LastName)`
- F-2：`@Subset(@Explode(FullPath; "/"); -1)`
- F-3：`@If(Amount > 0; @Success; @Failure("Amount must be > 0"))`
- F-4：`@DbColumn("Notes":"NoCache"; ""; "(LkProjects)"; 1)`
- F-5：

```
v := @DbLookup("Notes":"NoCache"; ""; "(LkProjects)"; ProjectCode; 3);
@If(@IsError(v); ""; v)
```

- F-6：`@Sum(@TextToNumber(Numbers))`（或附帶的 @For 版本）

### OM-1

```
Dim s As New NotesSession
Dim db As NotesDatabase : Set db = s.CurrentDatabase
Print db.Title & " | " & db.FilePath & " | " & db.AllDocuments.Count
```

### OM-2

```
Dim view As NotesView : Set view = db.GetView("MainView")
Dim doc As NotesDocument : Set doc = view.GetFirstDocument
Do Until doc Is Nothing
  Print doc.GetItemValue("Subject")(0)
  Set doc = view.GetNextDocument(doc)
Loop
```

### OM-3

```
' (a) 不開文件，快
Dim nav As NotesViewNavigator : Set nav = view.CreateViewNav
Dim e As NotesViewEntry : Set e = nav.GetFirst
Do Until e Is Nothing
  Print e.ColumnValues(0)
  Set e = nav.GetNext(e)
Loop
' (b) 開文件，慢——每筆都把整份載入記憶體
Set e = nav.GetFirst
Do Until e Is Nothing
  Print e.Document.GetItemValue("Subject")(0)
  Set e = nav.GetNext(e)
Loop
```

### OM-4

```
Print Join(doc.GetItemValue("Categories"), ",")   ' A,B,C
Print doc.GetItemValue("Status")(0)                ' 單值
```

### OM-5

```
Dim doc As NotesDocument : Set doc = db.CreateDocument
Call doc.ReplaceItemValue("Form", "Task")
Call doc.ReplaceItemValue("Subject", "練習")
Call doc.ReplaceItemValue("Created", Now)
Call doc.Save(True, False)
```

### OM-6

```
Dim col As NotesDocumentCollection
Set col = db.Search({Form="Task" & Status="Open"}, Nothing, 0)
Dim doc As NotesDocument : Set doc = col.GetFirstDocument
Do Until doc Is Nothing
  Call doc.ReplaceItemValue("Priority", "High")
  Call doc.ComputeWithForm(False, False)
  Call doc.Save(True, False)
  Set doc = col.GetNextDocument(doc)
Loop
```

（`{...}` 是 LS 字串字面值，省去雙引號跳脫。）

### OM-7

```
' (a) Search：對每筆套 formula，無索引，量大慢
Set col = db.Search({Form="Task" & Status="Open"}, Nothing, 0)
' (b) FTSearch：走全文索引，需先建 index
Set col = db.FTSearch({[Form]=Task AND [Status]=Open}, 0)
' (c) DQL：v10+ 現代做法
Dim dq As NotesDominoQuery : Set dq = db.CreateDominoQuery()
Set col = dq.Execute("Form = 'Task' and Status = 'Open'")
```

### OM-8

```
Set col = db.Search({Form="Task" & Status="Open"}, Nothing, 0)
Call col.StampAll("Priority", "High")   ' server 端批次寫，不逐筆載入
```

### OM-9

```
' 主文件
Dim parent As NotesDocument : Set parent = db.CreateDocument
Call parent.ReplaceItemValue("Form", "Task")
Call parent.ReplaceItemValue("Subject", "Parent task")
Call parent.Save(True, False)
' 回應
Dim i As Integer
For i = 1 To 2
  Dim r As NotesDocument : Set r = db.CreateDocument
  Call r.ReplaceItemValue("Form", "Comment")
  Call r.ReplaceItemValue("Body", "comment " & i)
  Call r.MakeResponse(parent)
  Call r.Save(True, False)
Next
' 走訪回應
Dim rc As NotesDocumentCollection : Set rc = parent.Responses
Dim c As NotesDocument : Set c = rc.GetFirstDocument
Do Until c Is Nothing
  Print c.GetItemValue("Body")(0)
  Set c = rc.GetNextDocument(c)
Loop
```

### OM-10

```
Dim ws As New NotesUIWorkspace
Dim uidoc As NotesUIDocument : Set uidoc = ws.CurrentDocument
Call uidoc.FieldSetText("Target", uidoc.FieldGetText("Source"))
Call uidoc.Refresh
```

### JX-1 / JX-2（Java agent）

```java
import lotus.domino.*;
public class JavaAgent extends AgentBase {
  public void NotesMain() {
    try {
      Session session = getSession();
      Database db = session.getCurrentDatabase();
      View view = db.getView("MainView");
      Document doc = view.getFirstDocument();
      while (doc != null) {
        System.out.println(doc.getItemValueString("Subject"));
        Document next = view.getNextDocument(doc);
        doc.recycle();          // JX-2：釋放底層 C handle
        doc = next;
      }
    } catch (Exception e) { e.printStackTrace(); }
  }
}
```

### JX-3 / JX-4

XPages 為 XML + SSJS，請在 Designer 的 XPage 設計器拖元件產生，骨架：

```xml
<xp:view xmlns:xp="http://www.ibm.com/xsp/core">
  <xp:this.data>
    <xp:dominoView var="view1" viewName="(All Tasks)"/>
  </xp:this.data>
  <xp:viewPanel value="#{view1}">
    <xp:this.facets><!-- 欄位略 --></xp:this.facets>
  </xp:viewPanel>
</xp:view>
```

---

## 學習路徑收尾建議

1. **Phase 1 限時 2 天**，別求精熟，目標是「查得到、看得懂」。
2. **Phase 2 是投資報酬率最高的一段**，OM-3、OM-7、OM-8、OM-10 這四題的「為什麼」比「能跑」重要——那是面試與日後 debug 的本錢。
3. **Phase 3 一定要真的做完一輪**，紙上談兵在 Domino 特別不管用（很多行為要實際存檔/重算才看得到）。
4. **Phase 4 等專案揭曉**：若是 Java agent，你 Phase 2 的物件鏈幾乎直接套用，只多學 recycle 與 Vector；若是 XPages，當成新框架重新排時間。
