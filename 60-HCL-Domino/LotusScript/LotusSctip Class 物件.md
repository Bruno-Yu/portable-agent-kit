
``` plaintext
Lotus Notes 物件模型 (Object Model)

📦 前端類別 (Front-end UI Classes) - 處理畫面與使用者互動
 ├── 📄 NotesUIWorkspace (UI 的起點，代表當前的 Notes 工作區)
 │    ├── 📄 NotesUIDatabase (目前開啟的資料庫畫面)
 │    ├── 📄 NotesUIView (目前開啟的視圖畫面)
 │    └── 📄 NotesUIDocument (目前開啟的文件畫面)
 |	 	  ├── 📄 NotesUIScheduler
 |		  |	  └── NotesDateTime ── NotesDateRange (可以從 view 取)
 |		  └── B2E NotesDocument
 └── 📄 其他介面元件: Button, Field, Navigator, NotesOutline...

📦 後端類別 (Back-end Classes) - 處理背景資料、邏輯與系統核心
 └── ⚙️ NotesSession (後端的起點，代表目前的連線工作階段)
      ├── 📂 NotesDbDirectory (伺服器上的資料庫目錄)
	  |   └── 📂 NotesDatabase (核心：資料庫物件)
      ├── 📂 NotesDatabase (核心：資料庫物件)
      │    ├── 🔐 NotesACL ── NotesACLEntry (權限控制)
      │    ├── 📑 NotesNoteCollection (集合)
      │    ├── 📑 NotesForm (表單設計)
      │    ├── 🗂️ NotesView (視圖/資料夾)
      │    │    ├── NotesViewNavigator (視圖導覽器) 可以拿到具體 view 上的資訊 ex: categories
      │    │    ├── NotesViewEntryCollection
      │    │    ├── NotesViewColumn (欄位)
      │    │    ├── NotesViewEntry (單筆條目)
      │    │    ├── 📚 NotesDocumentCollection (文件集合)
      │    │    │    └── 📄 NotesDocument (單筆文件)
      │    │    ├── 📄 NotesDocument (核心：文件資料)
      │    │    │    ├── 🏷️ NotesItem (普通欄位值)
      │    │    │    │   └── NotesMIMEEntry ── NotesMIMEHeader
	  │    │    │    │   └── 📝 NotesRichTextItem (Rich Text 欄位)
	  │    │    │    │        ├── NotesRichTextStyle
	  │    │    │    │        ├── NotesRichTextTable (表格)
	  │    │    |    |        ├── NotesRichTextDocLink (文件連結)
	  │    │    │    │        └── NotesRichTextNavigator ── NotesRichTextSection
	  │    │    │    ├──  NotesName
	  │    │    │    ├──  NotesRegistration
	  │    │    │    ├──  NotesTimer
	  │    │    │    ├──  NotesInternational
	  │    │    │    └── 📎 NotesEmbeddedObject (直接附在文件上的夾檔)
      │    └── 🔄 NotesReplication (抄寫設定)
      ├── ⏱️ NotesDateTime / NotesDateRange (時間處理)
      ├── 🤖 NotesAgent (代理程式)
      ├── 👤 NotesName (使用者名稱解析)
      └── 📝 NotesLog (系統日誌)

```

``` mermaid
graph LR
    subgraph 前端類別 [前端類別 UI Classes]
        UIW[NotesUIWorkspace] --> UIDB[NotesUIDatabase]
        UIW --> UIV[NotesUIView]
        UIW --> UIDoc[NotesUIDocument]
    end

    subgraph 後端類別 [後端類別 Back-end Classes]
        Session[NotesSession] --> DB[NotesDatabase]
        DB --> View[NotesView]
        DB --> DocCol[NotesDocumentCollection]
        DB --> Doc[NotesDocument]
        DocCol -->|包含| Doc
        Doc --> Item[NotesItem]
        Doc --> RTItem[NotesRichTextItem]
        Doc --> Embed[NotesEmbeddedObject]
    end

    %% 前後端對應關係
    UIDB -. 對應後端資料 .-> DB
    UIV -. 對應後端資料 .-> View
    UIDoc -. 對應後端資料 .-> Doc

    classDef ui fill:#e6f3d5,stroke:#4a7a2a,stroke-width:2px,color:#000;
    classDef backend fill:#fff6d5,stroke:#b38a1a,stroke-width:2px,color:#000;

    class UIW,UIDB,UIV,UIDoc ui;
    class Session,DB,View,DocCol,Doc,Item,RTItem,Embed backend;

```


## 前端物件

1. 以 NotesUIWorkspace 為首，取得 uidoc 後，取得欄位
2. 有 preview 可顯示

註1: 可用於 Notes Client 端
註2  前端 UI 物件不得使用於排程代理程式 (因為 agennt 沒有前端)
註3: 在 web 上也沒有 Notes 前端物件

``` vbscript
Sub Click(Source As Button)
	On Error Goto Errhandle
	Dim ws As New NotesUIWorkspace
	Dim uidoc As NotesUIDocument
	Dim tName As String

	Set uidoc = ws.CurrentDocument

	tName = uidoc.FieldgetText("CName") '取值

	tName = tName + "/O=XRedSchool"

	Call uidoc.FieldSetText("CName", tName) '設定欄位


Terminate: ' 正常流程執行的終點，防止流程進入 Errhandle 區塊
	Exit Sub
Errhandle:
	Messagebox "button " & Err() & ": " & Error() + " Error Line" + Cstr(Erl())
	Exit Sub
End Sub

```

以上同樣的規則，在 Formula 的做法

``` vbscript

FIELD CName = CName + "/O=XRedSchool";
@True

```


前端的 NotesUIDocument 可以拿到後端的 NotesDocument
用後端 document 寫邏輯，用前端 ui document 做儲存

``` vbscript
Sub Click(Source As Button)
	On Error Goto Errhandle
	Dim ws As New NotesUIWorkspace
	Dim uidoc As NotesUIDocument
	Dim doc As NotesDocument
	Dim tName As String

	Set uidoc = ws.CurrentDocument
	Set doc = uidoc.Document ' 這是後端的NotesDocument
%REM
	tName = uidoc.FieldgetText("CName")
	tName = tName + "/O=XRedSchool"
	Call uidoc.FieldSetText("CName", tName)
%END REM

	'在 Notes 後端（`NotesDocument`）的世界裡，有一個鐵律：**「所有的欄位資料（`NotesItem`）預設都是陣列 (Array)」**。
	tName = doc.Cname(0)
	Msgbox tName

	tName = tName + "/O=XRedSchool"
	'其實是 LotusScript 為了方便開發者而提供的一種「語法糖（捷徑）」，正式名稱叫做「擴充類別語法 (Extended Class Syntax)」。
	'它知道自己必須存成陣列，它會**自動把你給的字串，包裝成一個陣列**，然後存進去
	'硬要寫成 `doc.cName(0) = tName`，反而會報錯，因為這種「捷徑語法」不允許你直接針對陣列的特定索引去賦
	doc.cName = tName


Terminate:
	Exit Sub
Errhandle:
	Messagebox "button " & Err() & ": " & Error() + " Error Line" + Cstr(Erl())
	Exit Sub
End Sub

```




``` vb
Sub Click(Source As Button)
	On Error Goto Errhanle
	Dim ws As New NotesUIWorkspace
	Dim uidoc As NotesUIDocument
	Dim doc As NotesDocument
	Dim tID As String

	Set uidoc = ws.CurrentDocument
	If uidoc.EditMode Then '可判斷編輯模式與否 uidoc.EditMode，會回傳 Boolean (true/false)
		Msgbox "編輯模式"
	Else
		Msgbox "Read Mode"
	End If

Terminate:
	Exit Sub
Errhandle:
	Messagebox "button " & Err() & ": " & Error() + " Error Line" + Cstr(Erl())
	Exit Sub
End Sub

```


## 後端物件

``` vb
Session.currentdatabase '目前 Database
NotesUIDocument '取得UI上的文件
view.Getdocymentbykey(key, true) 'View 第一欄為排序即可依照key值取得文件
view.GetNextDocument(doc) '依目標找到下一筆
view.Getalldocumentsbykey 'View 中依key取得文件集合
db.Getdocumentbyunid '依unid取得文件
db.Unprocesseddocuments '畫面中選取文件
Submit Action

```

### 完全比對 & 不完全比對

`view.Getdocymentbykey(key, true)` 'View 第一欄為排序即可依照key值取得文件
這裡第二參數為 Boolean，true 代表完全符合，false 代表部分符合（從前面開始比）=> 建議都加上 true 避免錯誤

``` vb
Sub Click(Source As Button)
	On Error Goto Errhandle
	Dim ss As New NotesSession
	Dim db As NotesDatabase
	Dim HRdb As NotesDatabase
	Dim HRView As NotesView
	Dim HRdoc As NotesDocument

	Set db = ss.CurrentDatabase '先取得目前 db 為了拿到 server
	Msgbox "CurrentDatabase" + db.Server + "|" + db.Title

	' 從該 server 中拿到 hr 的 db
	Set HRdb = ss.GetDatabase(db.Server, "teacher/WFSample/EmpHR.nsf")

	If HRdb.isOpen Then
		Msgbox HRdb.Title
		Set HRView = HRdb.GetView("vwDeptCode")
		Set HRdoc = HRView.GetDocumentByKey("DP2001", true)
		Do Until HRdoc is Nothing
			Msgbox "部門名稱" + HRdoc.DeptCode(0) + " " + HRdoc.DeptCName(0)
			Set HRdoc = HRView.GetNextDocument(HRdoc)
		Loop
	Else
		Msgbox "找不到資料庫"
	End If


Terminate:
	Exit Sub
Errhandle:
	Messagebox "button " & Err() & ": " & Error() + " Error Line" + Cstr(Erl())
	Exit Sub
End Sub

```

`view.Getalldocumentsbykey`
``` vb
Sub Click(Source As Button)
	On Error Goto Errhandle
	Dim ss As New NotesSession
	Dim db As NotesDatabase
	Dim dc As NotesDocumentCollection '有個特型，同樣 collection 內他並不會照原本 view 中顯示的順序排序，而是照建立日期排序
	'若要照 view 排序建議用原本的 GetDocumentByKey 而不是 GetAllDocumentByKey

	Set db = ss.CurrentDatabase
	Set view = db.GetView("vEmpPerson03")
	Set dc = view.GetAllDocummentByKey("DP01", True)

	If dc.Count = 0 Then
		Msgbox "找不到資料"
	Else
		Msgbox "count - "& dc.Count

		For i = 1 To dc.Count
			Set doc = dc.GetNthDocument(i)
			MsgBox doc.empName(0) + " " + doc.UniversalID
		Next
		'或一次性全改，若本來沒有欄位，則會新增欄位
		Call dc.StampAll("IsupdateSAP", "1")
	End If
Terminate:
	Exit Sub
Errhandle:
	Messagebox "button " & Err() & ": " & Error() + " Error Line" + Cstr(Erl())
	Exit Sub
End Sub

```

`db.Getdocumentbyunid`
- 在 Note 中每份文件都有自己的 unid，copy 也會是不同的 unid ，換句話說在 copy database 時，若是用 unid 取文件，可能會取不到
	=> 除非用 Replication -> New replica 複製 unid 才不會變
	=> 除非用 Application -> New Copy 複製出去 unid 不會變
- 若 Getdocumentbyunid 沒抓到目標，會報錯，程式就不會進行下去

`db.Unprocesseddocuments` 通常用在 agent 中，算是 agent 的 to-do list ex: user 畫面選取的文件 / 排成時，尚未處理過的文件
但需要注意的是，為確保 Note 知道這已經處理了，需要搭配 Call agent.UpdateProcessedDoc(doc) 來確保 agent 知道這已處理了

``` vb
Dim ss As New NotesSession
Dim db As NotesDatabase
Dim dc As NotesDocumentCollection
Dim doc As NotesDocument
Dim i As Integer

Set db = ss.CurrentDatabase
Set dc = db.Unprocesseddocuments

If dc.Count = 0 Then
	MsgBox "找不到資料"
Else
	For i = i To dc.Count
		Set doc = dc.Getnthdocument(i)

		MsgBox doc.form(0) + " " + doc.Universalid
		doc.docNo = Format(Now, "ddhhmmss")
		Call doc.save(False, True) '我要存檔。但是如果系統發現剛剛已經有人先改過這份文件了，請不要蓋掉他的資料（`False`），但也別讓我的修改白費，請把我的版本存成一份衝突文件附在它下面（`True`），讓使用者事後自己來決定要保留哪一份
		'doc.Save
		'第一個參數
			' True 霸道模式，強制將我的版本覆蓋過去;
			' False: 「從我讀取這份文件到現在，有沒有其他人改過並存檔了？」。如果沒有，就正常存檔；如果「有」，系統就會攔截下來，並根據第二個參數來決定下一步
		'第二個參數
			' True (保留證據)： 當發生存檔衝突時，系統不會直接把你的存檔作廢，而是把你的這個版本，存成一份「回應文件 (Response Document)」。這就是在 Notes 視圖裡非常經典、左邊會帶有一個「菱形圖示」的「存檔衝突 (Save Conflict)」文件
			' False (直接放棄)：當發生衝突時，系統直接取消這次存檔動作（`Save` 函數會回傳 `False`），你的修改會像一陣風一樣消失，不會留下任何痕跡。
	Next
End If

Terminate:
	Exit Sub
Errhandle:
	Messagebox "button " & Err() & ": " & Error() + " Error Line" + Cstr(Erl())
	Exit Sub
End Sub
```


`Submit Action`

``` vb
@Command([ToolsRunMacro]; "lesson0803")
```

``` vb
On Error Goto Errhandle
Dim ws As New NotesUIWorkspace
Dim uidoc As NotesUIDocument
Dim doc As NotesDocument

Dim tName As String

 tName = doc.cname(0)
 MsgBox tName

 tName = tName + "/O=XRedSchool"

 doc.cname(0) = tName

 Call uidoc.save
 Call uidoc.Close


Terminate:
	Exit Sub
Errhandle:
	Messagebox "button " & Err() & ": " & Error() + " Error Line" + Cstr(Erl())
	Exit Sub
End Sub


```


## 存取文件中的欄位值

- 讀取：valueArray = NotesItem.Values
- 寫入：NotesItem.Values = valueArray

``` vb
Dim doc As NotesDocument
Set doc = ...
Set FieldItem = Doc.GetfirstItem("Students")

ForAll v IN FoeldItem.Values
	MessageBox(v)
End ForAll
MsgBox UBound(doc.students)

```

### 存取 NotesItem.Value 時的建議
在 Lotus Notes 的底層世界裡，不管欄位裡面裝的是一個字、還是一百個字，它「永遠」都會把它當成「陣列 (Array)」來儲存。

- 欄位不為多重值，存取此欄位要用 `NoteItem.Value(0)`，或簡單用 `doc.fieldname(0)`
``` vb
Dim doc As NotesDocument
Dim userName As String

' 錯誤寫法：會報錯，因為 doc.UserName 吐出來的是一個陣列
' userName = doc.UserName

' 正確寫法：明確指名要陣列的第 0 個位置 (也就是第一個值)
userName = doc.UserName(0)
Msgbox "使用者的名字是：" & userName
```
- 欄位允許多重值又不知道有多少筆時，可用 `Ubound` 函數來決定 `NotesItem.Values` 陣列的元素數目，再配合 `For I = 0 to Ubound(doc.fieldname)` 來取出所有元素
	在現代程式語言我們通常用 `array.length` 來看長度，但在 LotusScript 裡，我們用 **`Ubound()` (Upper Bound，最大索引值)**。
	```vb
	Dim doc As NotesDocument
	Dim i As Integer

	' 假設 doc.Owners 裡面有 ["Alice", "Bob", "Charlie"]
	' 此時 Ubound(doc.Owners) 的結果會是 2

	For i = 0 To Ubound(doc.Owners)
	    ' 迴圈會跑 3 次：i = 0, i = 1, i = 2
	    Msgbox "負責人包含：" & doc.Owners(i)
	Next

	```

- 不知道欄位是否允許多重值時，測試 `Ubound` 函數的回傳值是否為 0 即可（不為 0 表示允許多重值的欄位）
	-  如果 `Ubound()` 是 `0` 👉 代表最大索引值是 0 👉 陣列裡只有 1 個值 (單值)。
	- 如果 `Ubound()` 大於 `0` (例如 1, 2, 3...) 👉 代表陣列裡有好幾個值 (多值)。

雖然官方文件教你用 `For i = 0 To Ubound(...)`，但在實務開發上，我們更推薦用 **`Forall`** 迴圈。這個寫法有點像 JavaScript 的 `forEach` 或 `for...of`，你完全不需要去管 `Ubound` 是多少，也不用管它是不是多重值，系統會自動幫你掃過陣列裡的每一個值：

``` vb
Dim doc As NotesDocument

' 直接把欄位資料倒出來，不管是 1 個還是 100 個值，Forall 都會自動處理好
Forall person In doc.Owners
    Msgbox "負責人：" & person
End Forall

```

## Profile Document

存全域變數所用的，Notes 裡的 Profile Document 就等同於 **全域環境變數 (`.env`)**、**Global State (全域狀態)**，或是瀏覽器裡的 **`localStorage`**
需要存一些「整個系統共用的設定」**，或是**「每個使用者專屬的偏好設定」

> Profile 不存在 View

普通的 Notes 文件存檔後，會乖乖出現在某個視圖 (View) 列表裡讓大家點擊。但是 **Profile Document 是一份「隱形文件」
它被隔離在系統最底層，**無法顯示在任何普通的 View 裡面**。使用者永遠沒辦法像逛資料夾一樣找到它

`GetProfileDocument`，就把它當作是在撈「隱形的系統設定檔」**或**「隱形的個人偏好檔」就對了！

#### `GetProfileDocument` 的黑魔法
``` vb
Set notesDocument = notesDatabase.GetProfileDocument( profilename$ [, uniqueKey$] )

'Call ws.EditDocument(True, notesDocument)
```

這個函數有一個非常作弊的特性：**它結合了「讀取」與「新建」的功能。**（這就是圖中英文寫的 _Retrieves or creates a Profile document_）。當你執行這行程式碼時，系統會先去找有沒有這份設定檔；如果找不到，系統會**立刻自動在背景幫你生出一份全新的空文件**，絕對不會報錯說找不到！

它的兩個參數決定了你要抓的是哪種設定檔：
#### 模式 A：系統全域設定 (不放第二個參數)

如果你只放第一個參數（設定檔的表單名稱），這代表你要抓的是**全公司共用的唯一設定檔**。

``` vb
' 抓取名叫 "SysConfig" 的系統設定檔
Dim profileDoc As NotesDocument
Set profileDoc = db.GetProfileDocument("SysConfig")

' 把裡面設定的 API 網址抓出來用
apiURL = profileDoc.GetItemValue("API_Endpoint")(0)
```

#### 模式 B：個人化專屬設定 (放入第二個參數)

這就是圖中括號裡的 `[, uniqueKey$]` (選擇性參數)。如果你在這裡塞入「使用者的帳號名稱」，系統就會為這個人建立一個**他專屬的隱形設定檔**。


``` vb
' 抓取當前登入者專屬的設定檔
Dim userProfile As NotesDocument
Dim userName As String
userName = session.UserName ' 取得目前登入者的名字

' 傳入名字作為 uniqueKey
Set userProfile = db.GetProfileDocument("UserPreferences", userName)

' 讀取這個人自己設定的偏好語系
userLang = userProfile.GetItemValue("Language")(0)
```

Notes 為了讓系統跑得飛快，**會把 Profile Document 瘋狂地快取 (Cache) 在記憶體裡**。 這會導致一個經典的慘劇：如果系統管理員在後台修改了「系統設定檔」裡的某個值，一般使用者的電腦因為還吃著舊的快取，畫面上可能還是舊資料。


## 傳送 Email

Agent

``` vb
On Error Goto Errhandle
Dim ss As New NotesSession
Dim db As NotesDatabase
Dim dc As NotesDocumentCollection

Dim doc As NotesDocument

Set db = ss.CurrentDatabase
Set dc = db.Unprocesseddocuments
Set doc = dc.GetfirstDocument() '來源文件，只先取第一個

Dim maildoc As NotesDocument
Set maildoc = db.Createdocument()
Dim sendTo(1) As String
sendTo(0) = "admin"
sendTo(1) = "user01"

maildoc.Form = "Memo"

maildoc.SendTo = sendTo

'maildoc.copyto = sendTo 'cc
'maildoc.BlindCopyTo =sendTo '密件副本
'maildoc.Principal = "Admin" '寄件人

'ClassName 是 lotus script 提供的擴充寫法（Extended Syntax）用來呼叫文件上叫 ClassName 的欄位 （Item/Field）
maildoc.Subject = "請簽核文件: " + doc.ClassName(0)
Dim MailBody As NotesRichTextItem

Set MailBody = New NotesRichTextItem(maildoc, "Body")
Call mailBody.Appendtext("簽核文件：")
Call mailBody.Addnewline(2)
Call mailBody.Appendtext("DocLink:")
Call mailBody.AppendDocLink(doc, "點此處已開啟文件")
Call maildoc.send(False)
' ' 將郵件寄出。False 代表傳遞時「不要」將 Memo 表單本身的設計夾帶過去 (可節省容量)

Terminate:
	Exit Sub
Errhandle:
	Messagebox "button " & Err() & ": " & Error() + " Error Line" + Cstr(Erl())
	Exit Sub
End Sub

```

建立者可以選擇同部門的人員
- 提示： `ws.PickListCollection()`...
- View 在人員組織資料庫 `vwEmpDept`
- 只 show 該部門： `PickListCollection` 最後一個參數傳 `doc.RequesterDeptCode(0)` 來顯示單一類別
``` vb
@Command([ToolsRunMacro];" ") '它的主要功能是：「觸發並執行一個名為 " " 的代理程式 (Agent)。
```




---
## 補充



### 1. 核心規則：括號的愛恨情仇

在 LotusScript 中，呼叫一個 `Sub`（或沒有要接收回傳值的 `Function`）時，加不加 `Call` 決定了你**能不能使用括號**把參數包起來。

**1. 如果你使用 `Call` (必須加括號)**

這就是你提供的程式碼寫法。這也是很多老手習慣的寫法，因為括號可以把參數整齊地包起來，視覺上比較清楚。

程式碼片段

```
Call uidoc.FieldSetText("CName", tName)  ' ✅ 正確寫法
```

**2. 如果你不使用 `Call` (絕對不能加括號)**

如果你不想打 `Call` 這四個字母，你就**不能**用括號把參數包起來（參數之間直接用逗號隔開）。

程式碼片段

```
uidoc.FieldSetText "CName", tName      ' ✅ 正確寫法 (這行跟上面那行結果一模一樣)
```

**3. 最常犯的語法錯誤 (省略 Call 卻加了括號)**

如果你把 `Call` 拿掉，卻保留了括號，系統編譯時就會報錯，這也是為什麼很多開發者乾脆全部加上 `Call` 來避免麻煩。

程式碼片段

```
uidoc.FieldSetText("CName", tName)     ' ❌ 語法錯誤！系統會看不懂
```

### 那 `Function` 呢？

`Function` 的特色是「會回傳一個值」。你可以直接看你貼的程式碼中的這一行：

程式碼片段

```
tName = uidoc.FieldgetText("CName") '取值
```

`uidoc.FieldgetText` 就是一個道道地地的 **Function**。

當你要**接收** Function 的回傳值（把它存進變數 `tName`）時：

1. **不能**使用 `Call`。

2. **必須**使用括號 `()` 把參數包起來。


### 總結對照表

為了讓你未來寫 Code 時不再困惑，可以記住這個對照表：

| **你的目的**              | **語法結構**                | **範例**                                    |
| --------------------- | ----------------------- | ----------------------------------------- |
| **執行 Sub (有括號)**      | `Call` 巨集名稱`(參數1, 參數2)` | `Call uidoc.FieldSetText("CName", tName)` |
| **執行 Sub (無括號)**      | 巨集名稱 `參數1, 參數2`         | `uidoc.FieldSetText "CName", tName`       |
| **執行 Function (要取值)** | 變數 = 函數名稱`(參數1, 參數2)`   | `tName = uidoc.FieldgetText("CName")`     |



### 2. 多值賦值陣列


Notes 的欄位天生就是用來裝陣列的，那麼把「多個值」寫進去其實非常直覺。

在 LotusScript 中，處理多值欄位（Multi-Value Field）的核心觀念就是：**「先在程式裡準備好一個陣列，然後整包塞給欄位。」**

以下為你介紹三種最常見的做法，從最標準的寫法到進階的附加寫法：

#### 方法一：標準寫法（先建陣列，再 ReplaceItemValue）

這是最嚴謹、最推薦的寫法。我們先宣告一個陣列變數，把值塞進去後，整包倒進 `ReplaceItemValue` 中。



```vbscript
Dim doc As NotesDocument
' ...(假設前面已經取得 doc 物件)...

' 1. 準備一個陣列
Dim myRoles(2) As String
myRoles(0) = "Admin"
myRoles(1) = "Manager"
myRoles(2) = "User"

' 2. 將整個陣列賦值給欄位 (標準寫法)
Call doc.ReplaceItemValue("UserRoles", myRoles)
```

#### 方法二：捷徑寫法（直接用點號賦值）

延續上一篇提到的「語法糖」，你一樣可以直接把陣列用等號賦值給欄位名稱。系統一樣會整包吃進去。

程式碼片段

```vbscript
Dim myRoles(2) As String
myRoles(0) = "Admin"
myRoles(1) = "Manager"
myRoles(2) = "User"

' 將整個陣列賦值給欄位 (捷徑寫法)
doc.UserRoles = myRoles
```

#### 💡 實戰技巧：如何快速產生陣列？

在實務上，一行一行寫 `myRoles(0)`, `myRoles(1)` 有點麻煩。我們通常會搭配 LotusScript 的 `Split` 函數，把一個包含逗號或分號的字串，瞬間切成陣列並存入欄位：

程式碼片段

```vbscript
Dim roleString As String
Dim roleArray As Variant

roleString = "Admin,Manager,User"

' Split 會根據逗號把字串切成陣列
roleArray = Split(roleString, ",")

' 直接寫入欄位
Call doc.ReplaceItemValue("UserRoles", roleArray)
```

#### 方法三：附加寫法（不覆蓋原本資料，只在後面新增）

有時候你不是要「覆蓋」整個欄位，而是想在原本已經有多個值的欄位中，「再追加」一個新值進去。 這時候你不能直接用 `doc.FieldName = ...`，因為那會把舊資料洗掉。你必須先取得 `NotesItem` 物件，然後使用 **`AppendToTextList`** 方法：

程式碼片段

```vbscript
Dim item As NotesItem

' 1. 取得該欄位的 Item 物件
Set item = doc.GetFirstItem("UserRoles")

' 2. 檢查欄位是否存在，如果存在就追加新值
If Not item Is Nothing Then
    Call item.AppendToTextList("SuperVisor") ' 在原本的陣列最後面加上 "SuperVisor"
Else
    ' 如果原本沒有這個欄位，就直接新建一個
    Call doc.ReplaceItemValue("UserRoles", "SuperVisor")
End If
```

**總結：**

- **要塞多個值：** 把變數宣告成 Array 或 Variant（並裝入陣列），直接整包塞給 `ReplaceItemValue` 或是 `doc.欄位名`。

- **要追加值：** 抓出 `NotesItem`，使用 `AppendToTextList`。


### 3. 為什麼 `doc.UserRoles` 可以直接賦值，取值卻要寫 `(0)`？其實可以直接取陣列！

這是一個天大的誤會，其實 **`doc.UserRoles` 是可以直接取出陣列的！**

前面會寫成 `doc.Cname(0)`，是因為在那個情境下，工程師只想抓出「第一個字串」來做字串相加。如果你想把整個陣列抓出來，完全可以直接等號給值：

程式碼片段

```vbscript
Dim myRoles As Variant ' 宣告為 Variant 才能接收未知的陣列

' 【取值】把欄位裡面的陣列整包倒出來
myRoles = doc.UserRoles

' 測試印出陣列的第一個和第二個值
Msgbox myRoles(0)
Msgbox myRoles(1)
```

**為什麼設計成這樣？**

這種寫法叫做「擴充類別語法 (Extended Class Syntax)」。這其實是 Lotus 早期為了讓寫程式「看起來更簡單」而發明的捷徑。

- **當你寫 `doc.UserRoles = "Admin"` 時：** 底層會自動幫你轉成 `ReplaceItemValue("UserRoles", "Admin")`。

- **當你寫 `v = doc.UserRoles` 時：** 底層會自動幫你轉成 `v = doc.GetItemValue("UserRoles")`，並回傳一個陣列。


#### 2. 後端 (NotesDocument) 的正規做法是什麼？

如果不用上面的「捷徑寫法」，在後端 (`NotesDocument`) 最嚴謹的做法，要看你「想要拿什麼東西」：

- **情境 A：我只要裡面的「資料 (值)」** 👉 使用 `GetItemValue`

    它會永遠回傳一個「陣列」，就算裡面只有一個值，也是裝在陣列裡。

    程式碼片段

    ```vbscript
    Dim rolesArray As Variant
    rolesArray = doc.GetItemValue("UserRoles")
    ```

- **情境 B：我要整個「欄位物件」來操作** 👉 使用 `GetFirstItem`

    回傳的是一個 `NotesItem` 物件。當你需要修改欄位的屬性（例如：設定它能不能被讀者看到 `IsReaders`，或是用 `AppendToTextList` 加資料）時才用這個。

    程式碼片段

    ```vbscript
    Dim item As NotesItem
    Set item = doc.GetFirstItem("UserRoles")
    Call item.AppendToTextList("NewRole")
    ```

- **有 `GetItems` 嗎？**

    沒有這個函數！但是 `NotesDocument` 有一個屬性叫做 `Items`。如果你想掃描這份文件「所有的欄位」，你可以這樣寫：

    程式碼片段

    ```vbscript
    Forall item In doc.Items
        Msgbox item.Name ' 印出這份文件所有欄位的名稱
    End Forall
    ```


#### 3. 前端 (NotesUIDocument) 的做法就是 `FieldGetText` 嗎？

**沒錯！前端 UI 的世界非常單純。**

前端 (`NotesUIDocument`) 在乎的是「畫面上顯示了什麼字」，它不管你後端到底存的是數字、日期還是陣列。

- **取值：`uidoc.FieldGetText("欄位名")`**

    這永遠只會回傳一個 **字串 (String)**。如果畫面上的欄位是多值的（例如顯示為 `"Admin, User"`），它抓出來的就是包含逗號的一整串文字，**不會是陣列**。

- **賦值：`uidoc.FieldSetText("欄位名", "字串")`**

    你也只能塞字串給它。


#### 總結對照表：後端 vs 前端

為了方便你記憶與放入筆記，我幫你整理了這張表：

| **動作**      | **後端 (NotesDocument) - 真實資料層**                                            | **前端 (NotesUIDocument) - 畫面層**                                    |
| ----------- | ------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| **取值 (正規)** | `doc.GetItemValue("欄位名")`<br><br>  <br><br>_(回傳陣列 Variant)_               | `uidoc.FieldGetText("欄位名")`<br><br>  <br><br>_(回傳純字串 String)_     |
| **取值 (捷徑)** | `doc.欄位名`<br><br>  <br><br>_(回傳陣列 Variant)_                               | 無此寫法                                                              |
| **賦值 (正規)** | `Call doc.ReplaceItemValue("欄位名", 值)`<br><br>  <br><br>_(可塞字串或陣列)_        | `Call uidoc.FieldSetText("欄位名", "值")`<br><br>  <br><br>_(只能塞純字串)_ |
| **賦值 (捷徑)** | `doc.欄位名 = 值`<br><br>  <br><br>_(可塞字串或陣列)_                                | 無此寫法                                                              |
| **抓物件**     | `Set item = doc.GetFirstItem("欄位名")`<br><br>  <br><br>_(回傳 NotesItem 物件)_ |                                                                   |

### LotusScript 陣列的三大重點

#### 1. `Dim myRoles(2) As String` 這是標準宣告嗎？

這是標準的「固定長度陣列 (Fixed Array)」宣告方式。

但這裡有一個超級容易踩坑的細節：括號裡的數字 `2` 代表的是 **「最大索引值 (Upper Bound)」**，而不是陣列長度！ 因為陣列索引是從 `0` 開始的，所以 `Dim myRoles(2)` 其實是在記憶體裡挖了 **3 個格子** 給你（分別是 0, 1, 2）。

#### 2. 我必須要用固定長度的陣列嗎？

**完全不需要！** 在實務上，我們更常使用「動態陣列 (Dynamic Array)」，也就是一開始先不決定大小，等程式執行時再依據資料量來長大。

有兩種主流做法可以處理動態陣列：

- **做法 A：先宣告空括號，再用 `Redim` 調整大小** 如果你確定陣列裡都是同一種資料（例如都是字串），你可以這樣寫：
    ```vbscript
    Dim myRoles() As String ' 括號裡面留空，代表它是動態陣列

    ' ... 程式執行中發現需要 3 個空間 ...
    Redim myRoles(2)
    myRoles(0) = "Admin"

    ' ... 後來發現又需要增加空間 ...
    Redim Preserve myRoles(3) ' 加上 Preserve 保留舊資料
    myRoles(3) = "User"
    ```

- **做法 B：直接用 `Variant` 搭配 `ArrayAppend` (最接近現代語言的寫法)** 如果你不想一直算大小、寫 `Redim`，你可以把變數宣告為 `Variant`，然後把它當作 JavaScript 的 `.push()` 來用：

    ``` vbscript
    Dim myRoles As Variant

    ' 初始化一個只有一個元素的陣列
    myRoles = Split("Admin", ",")

    ' 直接把新值塞到最後面，系統會自動幫你擴充陣列
    myRoles = Arrayappend(myRoles, "Manager")
    myRoles = Arrayappend(myRoles, "User")
    ```


#### 3. 陣列中可以放哪些型別？

LotusScript 的陣列非常彈性，幾乎什麼都能放。主要分為以下三種層次：

- **基本資料型別：** 最常見的就是 `String` (字串)、`Integer` (整數)、`Long` (長整數)、`Double` (浮點數)。這類陣列裡面的東西必須「純度很高」，宣告 `String` 就絕對不能塞數字進去。

- **物件型別 (Objects)：** 你可以宣告一個專門裝 Notes 物件的陣列。例如你搜出了一堆文件，你想把它們存起來：

    程式碼片段

    ```vbscript
    Dim docArray(10) As NotesDocument ' 這個陣列專門用來裝 NotesDocument 物件
    ```

- **萬用型別 (Variant) 🌟：** 這是 Notes 開發中最靈活、也最常跟表單欄位互動的型別。`Variant` 就像是 TypeScript 裡的 `any`，當你宣告一個 `Variant` 陣列時，它的每一個格子都可以裝不同的東西（第一個格子裝字串，第二個格子裝數字，第三個格子裝物件都沒問題）。

    **💡 實務重點：** 當你使用 `doc.GetItemValue("欄位名")` 或 `doc.欄位名` 從資料庫把多值欄位抓出來時，Notes 系統回傳的**一定是一個 `Variant` 陣列**。因此在承接後端資料時，用 `Variant` 來接是最安全且不會報錯的標準起手式。


### 5. Arrayappend 與 AppendToTextList 差異

這兩個函數看起來都在做「新增資料到陣列/清單」這件事，非常容易讓人混淆！但要區分它們其實很簡單，你只需要記住一個核心觀念：**它們活在完全不同的世界。**

- **`Arrayappend`** 是 **「記憶體」** 裡的陣列操作工具（針對變數）。

- **`AppendToTextList`** 是 **「Notes 資料庫」** 裡的欄位操作工具（針對 `NotesItem` 物件）。


我們來詳細拆解這兩者的區別：

#### 1. `Arrayappend` (純粹的 LotusScript 陣列合併工具)

它是一個 LotusScript 內建的通用函數。它的唯一工作，就是把兩個陣列（或變數）**黏在一起，變成一個更大的新陣列**。它完全不知道什麼是 Notes 文件或欄位，它只在乎記憶體裡的資料。

- **作用對象：** `Variant` 變數或陣列。

- **寫法特點：** 它會「回傳」一個新的陣列，所以你必須把結果重新指派給一個變數（通常是自己）。


**實戰範例：**

程式碼片段

```vbscript
Dim arr1 As Variant
Dim arr2 As Variant
Dim finalArray As Variant

arr1 = Split("A,B", ",") ' 此時 arr1 是一個包含 "A", "B" 的陣列
arr2 = "C"

' 將 arr1 和 arr2 黏起來，並把結果存入 finalArray
finalArray = Arrayappend(arr1, arr2)

' 此時 finalArray 變成了 ["A", "B", "C"]，但這一切只發生在記憶體裡
' Notes 文件完全沒有任何改變！
```

#### 2. `AppendToTextList` (專屬 Notes 欄位的附加工具)

這是 `NotesItem` (也就是 Notes 後端欄位物件) 專屬的方法。當你從文件中抓出一個欄位物件後，你可以用這個方法直接把新字串「塞進」這個欄位現有的清單最後面。

- **作用對象：** `NotesItem` 物件（而且必須是文字或文字清單型別）。

- **寫法特點：** 它是物件的一個「動作 (Method)」，不需要回傳值，所以通常會搭配 `Call` 來使用。它會直接修改這個欄位物件的內容。


**實戰範例：**

程式碼片段

```vb
Dim doc As NotesDocument
Dim item As NotesItem

' 1. 從文件中把 "UserRoles" 這個欄位「物件」抓出來
Set item = doc.GetFirstItem("UserRoles")

' 2. 直接對這個物件下達「附加」的指令
Call item.AppendToTextList("NewRole")

' 執行完這行，doc 裡面的 UserRoles 欄位就已經多出一個 "NewRole" 了
' (註：最後還是要記得執行 doc.Save(True, True) 才會真的存進硬碟)
```

### 🚀 終極對照表

|比較項目|`Arrayappend`|`item.AppendToTextList`|
|---|---|---|
|**它是什麼？**|LotusScript 內建語法函數|`NotesItem` 物件的專屬方法|
|**操作對象**|`Variant` 型別的**變數/陣列**|`NotesItem` **欄位物件**|
|**標準寫法**|`變數 = Arrayappend(變數A, 變數B)`|`Call item.AppendToTextList("值")`|
|**是否直接影響文件？**|**否**，它只改變記憶體裡的變數。要寫入文件還要再呼叫 `doc.ReplaceItemValue`|**是**，它直接修改了記憶體中的 NotesDocument 欄位資料。|
|**使用情境**|當你在程式邏輯中，需要把好幾個陣列湊成一大包資料時。|當你已經確定要針對某個實體欄位，在它的舊資料後面追加新資料時。|

**簡單的總結：** 如果你正在「切菜、準備食材」（處理變數），用 `Arrayappend`。 如果你準備要把食材「丟進特定的鍋子裡」（寫入指定的 Notes 欄位），把鍋子拿出來 (`GetFirstItem`)，然後用 `AppendToTextList` 丟進去！