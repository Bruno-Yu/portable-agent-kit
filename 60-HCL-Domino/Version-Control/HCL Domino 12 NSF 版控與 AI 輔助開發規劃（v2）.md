

> 目標：在**不破壞既有可運行二進制**的前提下，為公司大量 NSF 導入版控，並讓 AI 能安全地加速理解與開發。 v2 變更：新增完整的「**編得過、跑才死**」風險分類全表（第三章），並據此強化 Phase 2 分級流程與掃描 checklist。

---

## 一、核心觀念：ODP 有兩種用途，我們只強制需要其一

現行卡關的根源，是把 ODP（On-Disk Project）當成「**必須匯回 Designer 重編**的建置來源」。實際上 ODP 有兩種彼此獨立的用途：

|用途|資料流方向|得到什麼|風險|
|---|---|---|---|
|**A. 唯讀源碼鏡像**|Designer 匯出 → Git（**單向，永不匯回**）|版控、diff、code review、**AI 可讀的原始碼**|**零**（從不 import，就不會觸發重編）|
|**B. 建置來源**|Git ↔ Designer（雙向，import 會 recompile）|從源碼重建 NSF|重編可能爆錯（目前遇到的狀況）|

**結論：我們要的「版控 + AI 輔助」，只靠用途 A 就能全數取得，且對所有 NSF 零風險。**

對於「重編就跑不動」的 NSF：二進制永遠是真實來源（source of truth）；ODP 只是唯讀文字鏡像，供 diff / AI / 歷史追蹤，**永不匯回 Designer**。

---

## 二、最重要的新洞察：源碼與 Bytecode 漂移（Source–Bytecode Drift）

在列出全部風險前，先強調一個**很可能是貴公司「重編就壞」的最大元兇**，因為它完美解釋「沒人動過的地方也跳錯」的現象：

**NSF 的每個設計元件同時儲存兩份東西：原始碼（source）與編譯後的物件碼（bytecode）。這兩份可以不一致。**

漂移的成因：

- 多年來有人改了 source 但**存檔失敗／只存了一半／從未成功重編**，舊的、能跑的 bytecode 被保留下來繼續服役。
- 曾用 DXL、第三方工具或程式化方式修改設計元件，只動到 source 不動 bytecode。
- 被停用（disabled）的 agent、被隱藏的設計元件，其 source 早已腐爛，但因為從不執行也從不重編，所以無人發現。
- 複製元件標版號的舊版控方式，讓大量「殭屍元件」帶著壞掉的 source 存活在 NSF 裡。

**後果**：正式機上跑的是「舊 bytecode」，而 Recompile All 編的是「現在的 source」。兩者根本不是同一份程式。重編後：

1. 壞掉的 source 編不過 → 連帶讓相依元件在**執行期**炸出 `Error loading USE or USELSX module`；
2. 或 source 編得過但邏輯與舊 bytecode 不同 → **本來能跑的地方行為改變／跳錯**。

> 這代表：貴公司的「重編就壞」很可能**不全是 HCL 12 的錯**，而是 NSF 裡的 source 早就不是正式機上實際運行的那份程式。這一點直接影響分級策略——見第四章 Phase 2 的「漂移偵測」步驟。

---

## 三、「編得過、跑才死」完整風險分類表

> 定義：Recompile All **成功**（或根本不需重編），但部署後在**執行期**才失敗——包括崩潰、報錯、或「跑了但行為錯誤」。後者最陰險，因為連錯誤訊息都沒有。

### 分類 A：平台／位元數（32-bit → 64-bit）

|#|成因|為何編得過|何時炸|典型症狀／錯誤|檢測方式|修復|
|---|---|---|---|---|---|---|
|A1|`Declare ... Lib` 用 `Long` 存 handle／指標|`Long` 是合法型別，編譯器不知道它裝的是指標|該 API 被呼叫、且指標值超出 32-bit 範圍時|隨機崩潰、Domino server crash、值錯亂|掃 ODP：`Declare` + 參數/回傳為 `Long` 且語意是 handle/pointer|改 `LongPtr`（條件編譯保留相容）|
|A2|呼叫的 DLL 本身是 32-bit|編譯只檢查語法，不載入 DLL|第一次呼叫該 `Declare` 函式時|`Error loading external library`、File not found（DLL 明明在）|用 `dumpbin /headers` 或 PE 檢查工具驗 DLL 位元數|取得 64-bit 版 DLL，或改走 Web service/中介層|
|A3|`Type` 結構傳給 C API：欄位對齊(padding)/長度在 64-bit 下不同|結構定義語法合法|API 呼叫時 stack/記憶體毀損|資料錯亂、間歇性崩潰、極難重現|掃 `Type ... End Type` 有無用於 `Declare` 參數|依 64-bit 對齊重排結構、驗證欄位長度|
|A4|Notes C API handle（`NOTEHANDLE`、`HANDLE`）尺寸假設|同 A1|API 呼叫時|崩潰或 handle 無效|掃 `Declare` 指向 `nnotes.dll`/`libnotes`|改 `LongPtr` + 對照新版 C API 文件|
|A5|呼叫慣例（calling convention）不符|編譯器不驗證|呼叫時 stack 毀損|崩潰、參數值錯位|對照 DLL 文件|修正 `Declare` 宣告|

### 分類 B：執行環境相依（新舊 server 環境差異）

|#|成因|為何編得過|何時炸|典型症狀|檢測|修復|
|---|---|---|---|---|---|---|
|B1|`CreateObject()` COM/OLE 自動化（Excel、Word…）|編譯不解析 ProgID|執行到 `CreateObject` 時|`Cannot create automation object`|掃 `CreateObject`、`GetObject`|目標機安裝對應軟體＋位元數一致；或改用檔案格式函式庫|
|B2|ODBC／LS:DO：driver 未裝、只有 32-bit driver、或 DSN 不存在|連線字串只是字串|開連線時|`ODBC driver not found`、connect 失敗|掃 `ODBCConnection`、`UseLSX "*LSXODBC"`；盤點新 server 的 64-bit DSN|安裝 64-bit driver、重建 DSN|
|B3|`UseLSX` 引用的 LSX 模組未安裝在新 server|`UseLSX` 是執行期載入|agent 啟動載入時|`Error loading USE or USELSX module`|掃 `UseLSX` 清單，逐一比對新 server 安裝|安裝 LSX 或移除相依|
|B4|寫死的 server 名稱、replica ID、檔案路徑|都是字串常數|存取該資源時|找不到 DB／檔案、`GetDatabase` 回 Nothing 後炸 NullPointer|掃字串常值中的 server/path 樣式|抽成設定文件（profile doc / notes.ini）|
|B5|依賴特定 `notes.ini` 變數（`Environ`/`GetEnvironmentString`）|讀不到只是回空值|後續邏輯用到空值時|行為錯誤、無錯誤訊息|掃 `Environ`、`GetEnvironment`|盤點並遷移 ini 設定|
|B6|`Shell`/OS 命令呼叫：目標程式不存在或路徑不同|字串常數|執行時|命令失敗、無聲失敗|掃 `Shell(`|盤點外部程式相依|
|B7|OS 層地區設定（日期格式、小數點符號）不同|與編譯無關|字串↔日期/數字轉換時|日期錯亂、`Type mismatch`|掃 `CDat`、`Format$`、字串日期解析|改用 `NotesDateTime`、明確格式|

### 分類 C：JVM／Java（Domino 11→12 JVM 變更）

|#|成因|為何編得過|何時炸|典型症狀|檢測|修復|
|---|---|---|---|---|---|---|
|C1|**`jvm/lib/ext` 不再自動載入**（Domino 11 起）|agent 內程式碼不需 import 即編過（jar 在編譯 classpath）|執行期找 class 時|`NoClassDefFoundError`／`ClassNotFoundException`|盤點舊 server `jvm/lib/ext` 內容|jar 改附加到 agent、或依新版建議路徑部署|
|C2|新 JDK 移除的模組：`javax.xml.bind`(JAXB)、CORBA、`javax.activation` 等|若 jar 有附則編得過；反射載入更完全看不到|用到該 API 時|`NoClassDefFoundError`、`ClassNotFoundException`|掃 Java source import 清單|補上獨立函式庫 jar|
|C3|反射／內部 API（`sun.*`、`com.sun.*`）在新 JVM 被封鎖|反射是執行期行為|反射呼叫時|`IllegalAccessError`、`InaccessibleObjectException`|掃 `Class.forName`、`setAccessible`|改用公開 API|
|C4|LS2J 橋接在新 JVM 行為改變|LotusScript 端只看到字串類別名|`CreateJavaSession`/呼叫 Java 方法時|LS2J 錯誤、找不到類別|掃 `Uselsx "*javacon"`、LS2J 用法|逐一測試、必要時改寫|
|C5|舊 TLS 協定／弱加密在新 JVM 預設停用|網路行為與編譯無關|HTTPS 連線交握時|`SSLHandshakeException`、連線被拒|盤點 agent 對外 HTTPS 端點與其 TLS 版本|端點升級 TLS 1.2+；不得已才調 JVM 安全設定|
|C6|第三方 jar 與新 JVM／新 Domino 內建 jar 版本衝突|編譯用到的是附加 jar|classloader 解析時|`NoSuchMethodError`、`LinkageError`|盤點附加 jar 與 Domino 內建版本|升級/對齊 jar 版本|

### 分類 D：動態程式碼——編譯器根本看不到

|#|成因|為何編得過|何時炸|典型症狀|檢測|修復|
|---|---|---|---|---|---|---|
|D1|`Evaluate()`／`Execute()` 內的公式與程式碼是字串|字串不會被編譯|執行該字串時|公式錯誤、參照不存在的欄位/檢視|掃 `Evaluate(`、`Execute(`|逐一人工審閱（AI 可加速）|
|D2|以名稱動態取用設計元件：`GetView("...")`、`GetAgent("...")`、`GetForm`|名稱是字串|該元件被改名/刪除後執行時|回 Nothing → 後續 `Object variable not set`|掃 GetView/GetAgent/GetDocumentByKey 的字串常數，比對現存設計元件|建立設計元件名稱對照表|
|D3|Computed field／hide-when 公式參照已刪除的 shared field、欄位|公式在文件開啟時才求值|開表單/存文件時|表單報公式錯誤|DXL 掃公式引用 vs 現存欄位|清理殭屍引用|
|D4|`@DbLookup`/`@DbColumn` 指向的檢視／伺服器變動|公式參數是字串|求值時|回錯誤值、下拉選單空白|掃公式中的 view 名稱|對照檢視清單|

### 分類 E：源碼–Bytecode 漂移（詳見第二章）

|#|成因|為何「以前跑得動」|重編後何時炸|典型症狀|檢測|修復|
|---|---|---|---|---|---|---|
|E1|source 曾被改壞但從未重編，舊 bytecode 續命|執行的是舊 bytecode|Recompile All 讓壞 source 生效（或編不過）|沒人動過的地方跳錯|測試 replica 上 Recompile All，收集全部編譯錯誤清單|修 source 或從 golden 二進制反查正確版本|
|E2|某 Script Library 重編失敗 → 相依 agent 執行期連鎖炸|舊 bytecode 彼此相容|agent 載入 library 時|`Error loading USE or USELSX module`（**這是執行期錯誤**）|建 `Use` 相依圖，由底層往上重編|先修底層 library|
|E3|停用中的 agent／殭屍元件帶壞 source|從不執行|Recompile All 波及；或有人誤啟用|編譯錯誤清單被殭屍元件灌爆，掩蓋真問題|先盤點停用/殭屍元件，決定刪或修|清理後再重編，訊噪比才會出來|
|E4|曾用 DXL/工具改設計，source 與 bytecode 不同步|同 E1|同 E1|行為改變但無錯誤訊息（最陰）|關鍵 agent 逐一「舊環境行為 vs 新編譯行為」對照測試|以 golden 二進制行為為準修 source|

### 分類 F：簽章與安全語意——「跑得動，但行為錯」

|#|成因|為何編得過|何時炸|典型症狀|檢測|修復|
|---|---|---|---|---|---|---|
|F1|Agent 簽章者改變 → 有效使用者改變 → **Readers/Authors 欄位讓 agent 看不到文件**|與編譯無關|agent 執行、搜尋文件時|agent「正常結束」但處理 0 筆；資料靜默漏處理|盤點含 Readers/Authors 欄位的 DB × 在其上跑的 agent 簽章者|新簽章 ID 加入對應 Readers/Authors 群組|
|F2|新簽章者無 Unrestricted 權限（檔案系統、網路、`Shell`…）|與編譯無關|執行受限操作時|`Agent signer not authorized`、AMgr log 報權限錯|對照 Server doc → Security → Agent 執行權設定|將 build ID 加入 unrestricted 名單|
|F3|新簽章者不在目標 DB 的 ACL 或權限不足|同上|開 DB/寫文件時|`Not authorized`、寫入失敗|盤點 agent 存取的所有 DB ACL|調整 ACL|
|F4|加密欄位綁定舊 ID 的加密金鑰|同上|讀加密欄位時|解密失敗、欄位空白|掃含加密欄位的表單|金鑰移轉／重新加密策略|
|F5|ECL 不信任新簽章|同上|client 端執行嵌入程式碼時|ECL 警示、被拒|檢查 ECL 管理政策|政策推送信任新 ID|

### 分類 G：Web／XPages／整合層

|#|成因|為何編得過|何時炸|典型症狀|檢測|修復|
|---|---|---|---|---|---|---|
|G1|XPages：OSGi 外掛／Extension Library 版本不符|建置用的是 Designer 端 plugin|server 端 classloader 解析時|執行期 `ClassNotFoundException`、頁面 500|比對 Designer 與 server 的 plugin 版本|對齊部署 plugin|
|G2|傳統 Domino Web 渲染引擎跨版本差異|與編譯無關|頁面渲染時|版面跑掉、JS 失效|關鍵頁面視覺/功能回歸測試|逐頁修正|
|G3|Web service consumer：對方端點 TLS/WSDL 變動|stub 早已生成|呼叫時|交握失敗、SOAP fault|盤點 WS consumer 端點|更新 stub、TLS 對齊|
|G4|DECS／NotesSQL／外部連線設定未隨環境遷移|與編譯無關|連線時|外部資料讀不到|盤點整合設定文件|遷移連線設定|

### 分類 H：資料與雜項

|#|成因|為何編得過|何時炸|典型症狀|檢測|修復|
|---|---|---|---|---|---|---|
|H1|未加 `Option Declare`：打錯的變數名編譯照過|無宣告檢查|用到該變數時|`Type mismatch`、邏輯錯|掃缺 `Option Declare` 的元件|逐步補上（會逼出大量隱藏問題，需排程處理）|
|H2|Variant/Null 傳播：`Null` 進到不接受 Null 的運算|Variant 合法|資料剛好為 Null 時|間歇性 `Type mismatch`|掃 `IsNull` 缺漏的取值路徑|防禦式取值封裝|
|H3|DBCS/LMBCS：以 byte 為單位處理中文字串|字串函式合法|處理中文資料時|亂碼、截斷|掃 `LeftB`、`MidB`、`LenB`|改用字元版函式|
|H4|設計繼承（template inheritance）：夜間 Design task 把改動蓋回去|與編譯無關|部署隔夜後|「改好的又變回去了」|檢查 DB 屬性的 template 繼承設定|規劃 template 更新路徑，勿直接改繼承中的 DB|
|H5|Agent 逾時／server 資源設定差異|與編譯無關|長跑 agent 執行時|agent 被砍、跑一半|比對新舊 server 的 AMgr 設定|對齊設定或拆批|

---

## 四、四階段執行藍圖（v2 強化版）

### Phase 0：建立二進制安全網（動任何東西之前）

- 對**每一顆現役 NSF** 建立 golden 二進制快照，存入物件儲存（MinIO / S3 或 ADO artifact feed）。
- metadata：來源 Domino 版本、簽章 ID、編譯日期、ODS 版本、繼承 template。
- ❌ 不用 Git LFS（對二進制無實益、綁定 host）。
- **v2 新增**：golden 快照除了「檔案」，也要保存「**行為基準**」——關鍵 agent 的輸入輸出樣本、關鍵表單的畫面截圖。分類 E4／F1 那種「跑得動但行為錯」的問題，只有行為基準能抓到。

### Phase 1：全量匯出 ODP 唯讀鏡像，進 ADO Git（零風險）

- Designer 匯出 ODP → 推進自架 ADO Git（沿用網頁團隊那套）。
- 全員裝 **Swiper** 濾除 `DesignerVersion`、checksum 等雜訊。
- 此步不匯回、不重編 → 零風險，立即取得版控歷史 + AI 可讀源碼。

### Phase 2：分級 Triage（v2 強化：三段式）

**2a. 靜態掃描（在 ODP 文字上做，AI 可大幅加速）** 依第三章分類表逐項掃描，對每顆 NSF 產出風險評分卡：

- 分類 A（`Declare`/`Type`/C API）命中數
- 分類 B（環境相依）命中數
- 分類 C（Java/JVM）命中數
- 分類 D（動態程式碼）命中數
- 分類 F（簽章語意：Readers/Authors × agent 簽章者）命中數

**2b. 漂移偵測（測試 replica 上）**

- Recompile All → **收集完整編譯錯誤清單**（不是只看過/不過）。
- 錯誤先分流：殭屍元件（E3，直接標記待清）vs 現役元件（E1/E2，真風險）。
- 建 Script Library `Use` 相依圖，從底層往上重編，定位連鎖爆點。

**2c. 行為比對（冒煙測試）**

- 關鍵 agent：新編譯結果 vs Phase 0 行為基準逐一對照。
- 特別驗 F1：新簽章下 agent 處理的文件筆數是否與舊環境一致（抓 Readers 欄位靜默漏件）。

分級結果：

- **通過 2b+2c → Tier B（源碼為主）**：雙向 round-trip、PR、AI 直接改碼。
- **未過 → Tier A（二進制為主）**：二進制 canonical，ODP 唯讀鏡像；記錄卡在哪個分類編號，作為日後修復的 backlog。

### Phase 3：AI 在不觸發 round-trip 的前提下加速

|情境|AI 使用方式|
|---|---|
|**Phase 2a 靜態掃描**|AI 批次掃 ODP：分類 A~F 的樣式全部可自動標記，含 `Evaluate` 字串內公式的人工審閱加速|
|**Tier B**|AI 直接改 ODP 文字，正常 round-trip|
|**Tier A**|AI 讀鏡像做理解／盤點／文件化；改動經 Designer 手動套用，或先修復阻塞分類再升級 Tier B|
|**終局：現代化遷移**|AI 將 LotusScript 邏輯翻譯至 React + .NET；ODP 鏡像即最佳輸入|

---

## 五、Tier A / Tier B 分級判定表

|判定維度|Tier A（二進制為主）|Tier B（源碼為主）|
|---|---|---|
|canonical source of truth|二進制 NSF|ODP 源碼|
|ODP 用途|唯讀鏡像|雙向建置來源|
|能否匯回 Designer 重編|否（會壞）|是（已通過 2b+2c）|
|版控（diff / 歷史）|✅|✅|
|Code review / PR|讀為主，改用手動|✅ 完整|
|AI 讀取理解|✅|✅|
|AI 直接改碼並 sync|❌|✅|
|典型卡點|分類 A/C/E/F 命中|掃描乾淨、行為比對通過|
|改動流程|Designer 手動 → 重編 → 更新 golden 快照＋行為基準|分支 → PR → merge ODP → Designer sync → 編譯|

---

## 六、掃描 Checklist（v2：對應第三章分類編號）

### 6.1 靜態掃描（ODP 文字層，可腳本化）

- [ ] A1/A4：`Declare` 且參數/回傳含 `Long`（語意為 handle/pointer）
- [ ] A2：列出全部 `Declare ... Lib "xxx.dll"` → 逐一驗 DLL 位元數與新 server 存在性
- [ ] A3：`Type ... End Type` 是否用於 API 參數
- [ ] B1：`CreateObject` / `GetObject`
- [ ] B2：`ODBCConnection` / `UseLSX "*LSXODBC"` / DSN 名稱
- [ ] B3：全部 `UseLSX` 清單 → 比對新 server 安裝
- [ ] B4：字串常數中的 server 名／絕對路徑／replica ID
- [ ] B5：`Environ` / `GetEnvironmentString`
- [ ] B6：`Shell(`
- [ ] C1/C2：Java agent import 清單、附加 jar 清單、舊 server `jvm/lib/ext` 盤點
- [ ] C3：`Class.forName` / `setAccessible`
- [ ] C4：LS2J（`Uselsx "*javacon"`）
- [ ] C5：agent 對外 HTTPS 端點清單
- [ ] D1：`Evaluate(` / `Execute(` → 抽出字串內容人工/AI 審閱
- [ ] D2：`GetView(` / `GetAgent(` / `GetDocumentByKey` 字串 vs 現存設計元件
- [ ] H1:缺 `Option Declare` 的元件清單
- [ ] H3：`LeftB` / `MidB` / `LenB`

### 6.2 簽章與權限盤點（分類 F）

- [ ] canonical build ID 確定（建議專用 service ID）
- [ ] F1：含 Readers/Authors 欄位的 DB × 其上排程 agent 的簽章者對照表
- [ ] F2：Server doc → Security → Agent 執行權涵蓋 build ID
- [ ] F3：agent 觸及的所有 DB ACL 涵蓋 build ID
- [ ] F4：含加密欄位的表單清單
- [ ] F5：ECL 政策信任 build ID

### 6.3 漂移偵測（測試 replica，分類 E）

- [ ] Recompile All 完整錯誤清單（非只看過/不過）
- [ ] 錯誤分流：殭屍元件（待清）vs 現役元件（真風險）
- [ ] Script Library `Use` 相依圖，底層先編
- [ ] 停用 agent／複製版號殭屍元件盤點清單

### 6.4 行為基準與冒煙測試（分類 E4/F1/G/H）

- [ ] 關鍵 agent 輸入輸出樣本（Phase 0 錄基準）
- [ ] 新編譯後 agent 處理筆數 vs 基準（抓 Readers 靜默漏件）
- [ ] 主要表單開啟、儲存、計算欄位
- [ ] 排程 agent 於測試 server 正常觸發
- [ ] Web 頁面（若有）渲染與提交
- [ ] H4：DB template 繼承設定確認（防夜間 Design task 蓋回）

---

## 七、一句話路線總結

> 先鋪二進制＋**行為基準**安全網（Phase 0）→ 全量匯出唯讀 ODP 進 ADO，零風險拿到版控 + AI 可讀源碼（Phase 1）→ 三段式分級：靜態掃描、漂移偵測、行為比對（Phase 2）→ Tier B 走完整流程，Tier A 保二進制 canonical 但仍享 diff / AI（Phase 3）。

「重編就壞」不是玄學：不是**平台位元數（A）**、**環境相依（B）**、**JVM 變更（C）**、**動態程式碼（D）**，就是**源碼漂移（E）**或**簽章語意（F）**。每一項都有對應的檢測與修復路徑——分級清單建立後，哪些 NSF 永遠留 Tier A、哪些值得修上 Tier B，就是成本效益問題而非技術死結。

---

## 附錄：工具與資源

|工具|用途|備註|
|---|---|---|
|HCL Domino Designer 12|ODP 匯出 / 匯入、編譯|公司已有授權|
|Swiper（OpenNTF）|過濾 ODP diff 雜訊|免費，**全員必裝**|
|自架 ADO Git|版控 host|沿用網頁團隊既有環境|
|MinIO / S3 / ADO artifact feed|二進制 golden 快照＋行為基準儲存|取代 Git LFS|
|Copilot / Claude|靜態掃描（6.1）、讀碼、翻譯、盤點|讀 Phase 1 的 ODP 鏡像|
|Dockerized headless compile|CI 自動編譯 ODP → NSF artifact|Phase 2+ 再接|
|`dumpbin` / PE 檢查工具|驗 DLL 位元數（A2）|Windows SDK 內建|