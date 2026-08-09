

> 給有 React / .NET / Oracle 背景、用 Mac、現在要維護傳統 client app、未來要做 REST/Node 現代化的工程師。

---

## 0. 怎麼用這份手冊

你的情況有兩個階段，學習順序刻意這樣排：

- **階段一（現在的任務）**：維護既有的傳統 Notes client 應用 → 必學 **NSF 資料模型 + @Formula + LotusScript + Forms/Views/Agents**。這是你最陌生、最該先補的。
- **階段二（未來的任務）**：用 REST API / Node 接 Domino → 對你來說是「回到熟悉的世界」（HTTP/JSON/React/Node），真正的門檻是 server 端的部署與權限模型，而不是寫 code。

**一句話心法**：階段一花力氣理解「Domino 怎麼想事情」，階段二就只是把這套資料用你熟悉的方式接出來而已。

建議排程在最後一節（第 7 節）。

---

## 1. 最重要的觀念：NSF 不是關聯式資料庫

這是整個體系最大的認知衝擊，先搞懂這個，後面都好說。

你習慣的 Oracle：table、schema、column 型別、JOIN、normalization、SQL。 Domino 的 NSF（`.nsf` 檔）完全不是這套：

|關聯式（你熟悉的）|Domino NSF|說明|
|---|---|---|
|Database (schema)|**NSF 檔**|一個 `.nsf` 就是一個「應用程式」，含資料**和**設計（程式碼）|
|Table|（沒有對應物）|沒有 table。靠 **Form** 決定一筆資料「長什麼樣」|
|Row|**Document（文件）**|一筆資料。同一個 NSF 裡不同 document 可以有完全不同的欄位|
|Column|**Item / Field**|欄位是掛在 document 上的，**沒有強制 schema**，可隨時新增|
|`SELECT ... WHERE`|**View（視圖）**|預先建好的索引 + 排序 + 篩選，不是即時查詢|
|Index|View 本身就是索引|View 會被 server 預先計算與快取|
|Stored Procedure|**Agent（代理程式）**|排程或事件觸發的程式（LotusScript/@Formula/Java）|
|Schema migration|改 **Design**，套用到既有 document|改了 Form 不會自動改舊 document 的資料|

關鍵差異，務必內化：

1. **No schema enforcement**：document 上有什麼欄位，是「當初存的時候放了什麼」。Form 只是「檢視/輸入時的樣板」，不是約束。舊 document 可能缺欄位 → 程式要防呆。
2. **沒有布林型別**：Notes 沒有 boolean。慣用 `"true"/"false"` 字串，或 `"0"/"1"`。
3. **View ≈ 預先算好的查詢**：你不會「即時跑 SQL」。要列資料，通常是讀對應的 View；要查得更彈性，才用 DQL（見下）。
4. **資料與程式同住一個檔**：design element（Forms/Views/Agents/Script Libraries）跟資料都在 `.nsf` 裡。這也是為什麼開發 = 開 Designer 編輯那個 `.nsf`。
5. **Replication（複製）是核心特性**：同一個 NSF 可以在多台 server / client 間雙向同步。維護時要意識到「你改的東西會複製出去」。

> **DQL（Domino Query Language）**：v10 之後加入，讓你能寫比較像查詢語句的條件式抓 document，不必每種查法都先建一個 View。階段二用 REST API 時也會用到（`POST /query`）。先知道它存在即可。

---

## 2. Mac 開發環境（這是你最現實的卡點）

**核心事實：Domino Designer 是 Eclipse-based，而且只有 Windows 版，沒有原生 Mac/Linux 版。** 這是 HCL 長年未解的痛點，別浪費時間找 Mac 版 Designer。

你的可行選項：

### 選項 A：Windows 虛擬機（v12 最穩，建議先用這個）

- **Apple Silicon (M1/M2/M3/M4)**：用 **Parallels Desktop** 裝 Windows 11 ARM，Notes/Designer 是 x86，靠 Windows ARM 的 x86 模擬層跑得動（偶爾要調設定，但社群實證可行）。
- **Intel Mac**：Parallels 或 VMware Fusion 裝 Windows 10/11 都行。
- 裝好 Windows 後，安裝 **HCL Notes（含 Domino Designer）client**，用你的 Notes ID 連到團隊的 Domino 12 server 開發。

### 選項 B：Nomad Web 在瀏覽器跑 Designer（要看版本）

- 這條路**需要 Domino 14 + Nomad 1.0.10 以上**才支援「在瀏覽器裡跑 Designer」。
- 如果團隊 server 還鎖在 v12，多半用不了。**值得做的事**：跟團隊確認 server 端有沒有上 Nomad、之後會不會升 v14。若會升，這會大幅改善你的 Mac 體驗。

### 選項 C：純做階段二（現代化）時

- 寫 React / Node 那部分，你的 Mac 原生就能做，不需要 Designer。
- 只有「改 NSF 內的 design element」才一定要 Designer（= 要 Windows）。

**務實結論**：先用選項 A 的 VM 把 Designer 跑起來應付階段一；同時推動團隊評估 Nomad / v14。

---

## 3. 語言體系全景（先建立地圖，不用一次學完）

Domino 的程式語言是**分層、依用途選用**的。下面這張地圖比任何細節都重要：

```
┌─────────────────────────────────────────────────────────┐
│  傳統 Client / Web App（階段一）                            │
│                                                           │
│  @Formula  ── 宣告式小語言：欄位計算、view 欄位、簡單動作      │
│  LotusScript ── VBA 式 OOP：agent、表單事件、後端自動化       │
│  Java       ── 同樣能存取 Domino 物件模型（lotus.domino）     │
│  XPages+SSJS── XML Web 框架 + Server-Side JavaScript（偏舊）  │
└─────────────────────────────────────────────────────────┘
                          │  同一份 NSF 資料
                          ▼
┌─────────────────────────────────────────────────────────┐
│  現代化存取（階段二）                                        │
│                                                           │
│  Domino REST API (KEEP) ── OpenAPI/JWT，任何語言都能接 ★主線  │
│  AppDev Pack (domino-db) ── Node 模組 + Proton（gRPC）舊路    │
└─────────────────────────────────────────────────────────┘
```

對你而言的學習難度（憑你的底子）：

- **JavaScript → SSJS**：幾乎無痛（但 XPages 你維護時才碰，未必要主動深學）
- **OOP → LotusScript**：語法 1～2 天，難的是背後的 **Notes 類別樹**
- **@Formula**：語法很短但「慣用法」要花時間，這是最「Domino 味」的部分
- **REST/Node 現代化**：對你最簡單，門檻在 server 部署與權限，不在 code

---

## 4. 階段一：維護傳統 Client App（現在）

### 4.1 @Formula Language（公式語言）

宣告式、單運算式導向。用在：欄位的預設值/計算值、View 欄位、表單上的 hide-when、簡單的按鈕動作、formula agent。

特徵：

- 函式以 `@` 開頭：`@Today`、`@Name`、`@DbLookup`、`@If(...)`、`@DbColumn`
- 指令以 `@Command` 開頭，對應 UI 動作：`@Command([FileSave])`、`@Command([EditDocument]; "0")`、`@Command([RefreshWindow])`
- 多個運算式用 `;` 分隔，最後的值就是結果
- 沒有迴圈的傳統寫法（用 list/@For 等處理多值）

典型按鈕公式（存檔 → 切回唯讀 → 重整視窗）：

```
@Command([FileSave]);
@Command([EditDocument]; "0");
@Command([RefreshWindow])
```

最常用、一定要熟的：`@If`、`@DbLookup` / `@DbColumn`（跨 view 查值）、`@Name`、`@Text`、`@Trim`、`@Unique`、`@SetField` / `@GetField`、`@Prompt`（彈窗）。

### 4.2 LotusScript（主力語言）

類似 VBA / BASIC 的物件導向語言，是傳統 app 的工作馬。當 @Formula 不夠用（需要邏輯判斷、迴圈、改後端資料、複雜流程）就用它。

最關鍵的觀念是**兩套類別**：

**Back-end classes（後端：操作資料本身，不碰畫面）**

- `NotesSession` — 進入點，代表目前 session
- `NotesDatabase` — 一個 NSF
- `NotesView` — 一個 view
- `NotesDocument` — 一筆 document
- `NotesDocumentCollection` — 一組 document（查詢結果）
- `NotesItem` — 欄位

**Front-end classes（前端：操作使用者正在看的 UI）**

- `NotesUIWorkspace` — 工作區
- `NotesUIDocument` — 使用者目前開著的那筆文件畫面
- `NotesUIView` — 目前的 view 畫面

語法重點：

- `Dim` 宣告變數，`New` 初始化，`Set` 指派物件
- `Option Declare`（強制宣告，務必開，等於 VB 的 Option Explicit）放檔案最上方
- 事件式：表單/按鈕的程式碼掛在事件上（如 `Sub Click(...)`、`Sub Querysave(...)`）

典型按鈕（從唯讀狀態改後端欄位再存檔）：

```vbscript
Sub Click(Source As Button)
    Dim ws As New NotesUIWorkspace
    Dim uidoc As NotesUIDocument
    Dim doc As NotesDocument
    Set uidoc = ws.CurrentDocument        ' 前端：目前畫面文件
    Set doc = uidoc.Document              ' 取得後端版本
    doc.completed = "true"                ' 改欄位（注意：字串，沒有布林）
    Call doc.save(True, False)            ' 存檔
    Call ws.ReloadWindow                  ' 等同 @Command([RefreshWindow])
End Sub
```

後端批次處理的典型骨架（agent 常見）：

```vbscript
Dim session As New NotesSession
Dim db As NotesDatabase
Dim view As NotesView
Dim doc As NotesDocument
Set db = session.CurrentDatabase
Set view = db.GetView("MainView")
Set doc = view.GetFirstDocument()
While Not (doc Is Nothing)
    ' ...處理 doc...
    Call doc.Save(True, False)
    Set doc = view.GetNextDocument(doc)
Wend
```

> **學習重點**：語法你很快會，**真正要花時間的是把上面那棵類別樹背熟**——知道「我要改資料用後端、要動畫面用前端」，以及兩者之間的轉換（`uidoc.Document`）。

### 4.3 設計元件（Design Elements）

你在 Designer 裡看到的 NSF 由這些組成，維護時要認得：

- **Form**：document 的輸入/檢視樣板（決定欄位佈局、按鈕、hide-when）
- **View**：document 列表（欄位、排序、選取公式 `SELECT ...`）
- **Subform / Shared Field / Shared Action**：可重用元件
- **Agent**：排程或事件觸發的程式（LotusScript/Formula/Java/Simple Action）
- **Script Library**：可共用的 LotusScript/JS 程式庫
- **Page / Outline / Framesets**：傳統 web/導覽元件
- **XPages / Custom Controls**：較新的 web 層（若該 app 有用到）

### 4.4 除錯

- LotusScript 有內建 **Debugger**（在 Designer / Notes client 啟用），可下中斷點、看變數。
- Eclipse-based 的 **LotusScript Editor in Eclipse (LSEE)** 是另一個編輯選項，享有 Eclipse 的編輯功能。
- 早期最克難的除錯是 `MessageBox` / `Print`（輸出到 server console 或 status bar）——維護舊 code 時你會常看到。

---

## 5. 階段二：REST / Node 現代化（未來）

好消息：這一段對你最輕鬆，因為它就是 HTTP + JSON + 你熟的前後端。要學的不是新語言，是**架構與部署**。

### 5.1 Domino REST API（前身 Project KEEP）— 建議主線 ★

這是 HCL 目前主推、最現代、最該投資的路線。

- **本質**：在 Domino server 上跑的一個 task（`nrestapi`），對外提供 **OpenAPI 規格的 REST API**，用 **JWT** 認證。
- **版本門檻（很重要）**：作為 add-on，支援 **Notes/Domino 12.0.2 以上**；server 端可跑在 Windows / Linux / Docker，client 端（開發評估用）連 macOS 都行。 → **先確認你們的 server 是不是 12.0.2+**。若是 12.0.0 / 12.0.1，要嘛升 FP，要嘛只能走 AppDev Pack（見 5.2）。
- **任何語言都能接**：官方明列支援 ReactJS / VueJS / Svelte / Angular 前端，以及 Node / Spring / Quarkus / PHP 等後端，也能用 Java / C# / Rust / Python 桌面程式直連。**你的 React + .NET 都在清單內。**
- **安全模型是賣點**：權限「移出程式、放進設定」——用 JWT + Scopes，可整合企業 IdP（Keycloak / AD），甚至能做到 field-level 的讀寫控管。對重視權限控管的企業環境很關鍵。
- **DQL 查詢**：可透過 `POST /query`（以及較新的 QueryResultsProcessor JSON 端點）對 NSF 下查詢，拿回 JSON。
- **規格文件**：啟動後開它的 URL 就能看到「跑在你們 server 上的即時 Swagger/OpenAPI 文件」，永遠對應實際版本。

**你會怎麼用**：React 前端 / .NET 或 Node 後端 → 打 Domino REST API → 拿 NSF 裡的 document（JSON）→ 不用再寫 LotusScript。這就是「把傳統 app 的資料現代化接出來」的標準做法。

### 5.2 AppDev Pack（domino-db + Proton）— 你可能會遇到的舊路

比 REST API 更早的 Node 存取方案，知道它存在、能看懂即可：

- 組成：server 端 **Proton**（task）+ Node 的 **`domino-db`** npm 模組 + **IAM**（OAuth2.0 身分服務）。
- 走 **gRPC**，提供 document 的 CRUD 與 DQL 查詢。
- Domino 12.0 trial server 內含 AppDev Pack 1.0.8；但 `domino-db` 當年只測到 **Node 10.x**，相對老。
- **建議**：新東西優先用 Domino REST API（5.1）；只有在維護既有用 `domino-db` 的整合、或 server 卡在 <12.0.2 時才碰它。

### 5.3 從你的世界看過去（心智對照）

|你熟悉的|Domino 現代化對應|
|---|---|
|.NET Web API controller|Domino REST API 端點（設定式，非寫死）|
|EF / Dapper 查 Oracle|打 REST API 取 document JSON / DQL|
|JWT + OAuth2|KEEP 內建，整合企業 IdP|
|React 打自家 API|React 打 Domino REST API（同樣 fetch/axios）|
|Swagger / OpenAPI|KEEP 直接提供即時 OpenAPI 文件|

---

## 6. 速查與資源清單

**官方文件（對準 12.0.2）**

- LotusScript 官方教學（三課實作）：`help.hcl-software.com/dom_designer/12.0.2/...`（搜尋 "LotusScript Classes Tutorial"）
- Domino Designer 使用手冊與語言參考：`help.hcl-software.com/dom_designer/`
- Domino REST API 文件：`opensource.hcltechsw.com/Domino-rest-api/`
- AppDev Pack 文件：`doc.cwpcollaboration.com/appdevpack/`

**速查字典（寫 code 時當查表）**

- All @Functions A–Z、All @Commands A–Z、LotusScript Classes A–Z、Java/CORBA Classes A–Z（都可在 DominoNews / 官方 wiki 找到）

**動手教學**

- Paul Withers《Domino To-Do》：`paulswithers.github.io/domino_todo`（從零做 app，同時帶 @Formula + LotusScript，最適合建立第一手手感）

**社群與範例**

- 官方 Domino Designer Wiki（`ds-infolib.hcltechsw.com`）：XPages / DXL / Java / JS / LotusScript 文章
- **OpenNTF**：開源範例與工具（有 GitHub），遇到問題先翻這裡
- Planet Lotus（部落格聚合）、XPages.info、XPageDeveloper.com
- 部落客：Paul Withers、Oliver Busse、Daniel Nashed、Per Henrik Lausten 等
- DominoNews.com：追版本/FP 釋出狀態

---

## 7. 建議學習排程（約 6～8 週，可依工作量調整）

**第 0 週｜環境**

- VM 裝好 Windows + Notes/Designer，連到團隊 server，能打開一個既有 NSF 並瀏覽 design element。
- 確認 server 版本（是否 12.0.2+，影響階段二）。

**第 1 週｜心智模型 + @Formula**

- 讀完第 1 節，能用自己的話解釋「NSF vs 關聯式」5 個差異。
- @Formula：熟 `@If`、`@DbLookup`、`@DbColumn`、`@Command`，能看懂既有按鈕/欄位公式。

**第 2～3 週｜LotusScript**

- 跑完官方 LotusScript 三課教學。
- 把後端/前端兩套類別樹背熟；能寫「迴圈處理一個 view 的所有 document」骨架。
- 跟著 Paul Withers To-Do 教學做一遍。

**第 4 週｜設計元件 + 除錯 + 讀既有 code**

- 認得 Form / View / Agent / Subform / Script Library。
- 會用 LotusScript Debugger。
- 開始讀你接手的那支 app，畫出它的資料流。

**第 5 週｜階段二打底**

- 讀 Domino REST API 文件，在開發環境啟用 `restapi`，用 Swagger UI 打第一個 GET。
- 用 React/Node 寫一個小 demo：fetch 一個 view 的 document → 顯示成清單。

**第 6 週起｜現代化實作**

- 把某個傳統功能用 REST API 重新接出來（你最擅長的部分）。
- 研究 JWT + Scopes 權限、企業 IdP 整合（為受管制的企業環境鋪路）。

---

### 一頁總結

1. **觀念**：NSF 是文件資料庫，沒 schema、沒 JOIN、View 是預算好的索引。
2. **環境**：Designer 只有 Windows → Mac 用 VM；留意 v14 + Nomad 可在瀏覽器跑 Designer。
3. **階段一**：@Formula（小邏輯）+ LotusScript（主力，背類別樹）。
4. **階段二**：Domino REST API (KEEP) 為主線，需 server 12.0.2+；你的 React/.NET 直接能接。
5. **資源**：官方 12.0.2 教學 + Paul Withers To-Do + OpenNTF。
