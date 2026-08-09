
## 一、ADO 是什麼？
- ADO = **ActiveX Data Objects**，是 Microsoft 的資料存取元件
- LotusScript 透過 `CreateObject` 呼叫 COM 元件來使用
- 可存取 SQL Server、Oracle、Access 等關聯式資料庫
- RecordSet 是 **ADO 的物件**，非 LotusScript 內建功能

---

## 二、建立連線

```vbscript
Dim Conn As Object
Set Conn = CreateObject("ADODB.Connection")

' 靜態連線字串
Conn.Open("Driver={SQL Server}; Server=MYSERVER; Uid=sa; PWD=1234; Database=HRdb")

' 動態連線字串（變數組合）
Call Conn.Open("Driver={SQL Server}; Server=" + DataSource + _
               "; Uid=" + AccountName + "; PWD=" + Password)
```

### 連線字串參數說明
| 參數 | 說明 |
|------|------|
| `Driver={SQL Server}` | 使用 SQL Server ODBC 驅動 |
| `Server=` | 資料庫伺服器名稱或 IP |
| `Uid=` | 登入帳號 |
| `PWD=` | 登入密碼 |
| `Database=` | 指定資料庫名稱（可省略，改用跨庫語法） |

---

## 三、執行 SQL 取得 RecordSet

```vbscript
Dim rs As Object
Dim strSQL As String

strSQL = "SELECT * FROM empbas..v_hrresignemp WHERE change_date >= '2003/01/23'"
'                    ↑↑
'          資料庫..View名稱（兩點 = 省略 schema，預設 dbo）
'          SQL Server 跨資料庫查詢語法

Set rs = Conn.Execute(strSQL)
```

---

## 四、RecordSet 屬性

```vbscript
rs.EOF                        ' True = 已到最後一筆（無資料 or 到底）
rs.Fields.Count               ' 欄位總數（index 從 0 到 Count-1）
```

---

## 五、取得欄位值

```vbscript
rs.Fields.Item(0).Value         ' 用索引取值（0-based）
rs.Fields.Item("emp_no").Value  ' 用欄位名稱取值（推薦，可讀性高）

' 簡寫方式（效果相同）
rs("emp_no").Value
rs(0).Value
```

---

## 六、游標移動方法

| 方法 | 說明 | 需要特殊 CursorType |
|------|------|---------------------|
| `rs.MoveNext` | 移到下一筆 | ❌ 不需要（最常用） |
| `rs.MovePrevious` | 移到上一筆 | ✅ 需要 |
| `rs.MoveFirst` | 移到第一筆 | ✅ 需要 |
| `rs.MoveLast` | 移到最後一筆 | ✅ 需要 |
| `rs.Move(n)` | 移動 n 筆（正=往後，負=往前） | ✅ 需要 |

> ⚠️ `Conn.Execute` 預設為 **Forward-Only**，只支援 `MoveNext`
> 若需雙向移動，改用以下方式：

```vbscript
Set rs = CreateObject("ADODB.Recordset")
rs.CursorType = 1   ' adOpenKeyset，支援雙向移動
rs.Open "SELECT * FROM xxxx", Conn
```

---

## 七、迴圈讀取資料

### Do While 寫法
```vbscript
Do While Not rs.EOF
    ' 處理資料...
    rs.MoveNext     ' ⚠️ 必須有，否則無窮迴圈！
Loop
```

### Do Until 寫法（效果相同）
```vbscript
Do Until rs.EOF
    ' 處理資料...
    rs.MoveNext     ' ⚠️ 必須有，否則無窮迴圈！
Loop
```

| 寫法 | 語意 | 停止條件 |
|------|------|----------|
| `Do While Not rs.EOF` | 當還沒到底就繼續 | EOF = True 時停止 |
| `Do Until rs.EOF` | 直到到底才停止 | EOF = True 時停止 |

---

## 八、完整範例

```vbscript
Dim Conn As Object, rs As Object, strSQL As String

Set Conn = CreateObject("ADODB.Connection")
Call Conn.Open("Driver={SQL Server}; Server=" + DataSource + _
               "; Uid=" + AccountName + "; PWD=" + Password)

strSQL = "SELECT * FROM empbas..v_hrresignemp WHERE change_date >= '2003/01/23'"
Set rs = Conn.Execute(strSQL)

Do Until rs.EOF
    Dim empNo As String
    empNo = rs.Fields.Item("emp_no").Value
    ' 做你的處理...
    rs.MoveNext
Loop

rs.Close            ' 關閉 RecordSet
Conn.Close          ' 關閉連線
Set rs = Nothing    ' 釋放記憶體
Set Conn = Nothing
```

---

## 九、常見錯誤提醒

| 錯誤 | 原因 |
|------|------|
| 無窮迴圈 | 迴圈內忘記呼叫 `rs.MoveNext` |
| 找不到欄位 | 欄位名稱錯誤或 SQL 未 SELECT 該欄位 |
| MoveFirst 失敗 | `Conn.Execute` 預設 Forward-Only，不支援回頭移動 |
| 連線字串錯誤 | Driver 名稱需與本機安裝的 ODBC Driver 一致 |
| 資源未釋放 | 忘記 `rs.Close` / `Conn.Close` / `Set = Nothing` |
