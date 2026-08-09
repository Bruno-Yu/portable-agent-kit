


## Domino URL

* Host : DNS名稱 or Domino伺服器IP位址
* Database : 資料庫位於Data底下的相對路徑
* Domino Object : a database, view, document, form, navigator, agent
* Action : ?OpenDatabase, ?OpenView, ?OpenView, ?openDocument, ?EditDocument...
* Arguments : qualifiers of the action
    * - ?OpenForm
    * - ?OpenView&Count = 10 : 開啟View並顯示十筆
    * - xxx.nsf?logout : 登出

Ex:
https://www.xred.com.tw/names.nsf?opendatabase
https://www.xred.com.tw/names.nsf/person?openform
https://www.xred.com.tw/names.nsf/people?openview
https://host/dabase/0/unid?openDocumnet

## Domino Xpages URL

* Host : DNS名稱 or Domino伺服器IP位址
* Database : 資料庫位於Data底下的相對路徑
* PageName : Portal.xsp
* ?documentId=unid
* &action=openDocument/editDocument
* 以上大小寫有差別

http://host/Databse/PageName?documentId=unid&action=openDocument

## 開發者常用指令介紹

* 指令大小寫沒差，但是若server 裝在linux 系統則NSF名稱需區分大小寫。
* show Server:查看server資訊。
* show task:查看目前正在執行的Task 有哪些。
* res ser : restart Server 的縮寫，可以重新啟動Domino Server .
* q:停止server
* tell http q /load http: 停用/啟用http
* tell amgr q/load amgr:排程代理程式
* tell amgr run “xxx.nsf” ‘agent’ :手動執行排程代理程式
* Load compact xxx.nsf : 啟動資料庫壓縮(參數至help查詢 )
* Load updall xxx.nsf :更新資料庫view 索引 (參數至help查詢 )


### 字串

* @Right
* @RightBack
* @Left
* @LeftBack
* @Trim
* @ProperCase
* @UpperCase
* @LowerCase

---

### 陣列

* @Elements:回傳陣列數量
* @Explode : 把文字用分隔符號換成陣列
* @Implode : 把陣列轉換為字串
* @Unique
* @Contains : 陣列是否包含字串元素
* @Subset:取出陣列的前/後幾個元素
* @Sort:排序
* @Member:回傳字串在陣列中的第幾個元素


### 型態轉換

* @Text
* @TextToNumber
* @TextToTime
* @ToNumber
* @ToTime

*(左下方截圖部分可辨識之程式碼)*
```text
@TextToTime(@Explode(
@TextToTime(@Text(a) + "-" + @Text( b ))
))

```


## 一定要用LotusScript場合

1. ACL
2. 進入/離開套表與欄位時執行想做的動作
3. 新建/刪除資料庫
4. 處理RTF(如存取修改其內容等)
5. 完整的檔案處理I/O(如刪除檔案，讀取特定目錄等)
6. 跨資料庫寫入資料

- **RTF** 指的是 **Rich Text Field（富文本欄位 / 豐富文字欄位）**

## 一定要用@Formula的場合

1. 視界設計
    * 選擇/視界直欄(column)/套表公式
2. 套表設計
    * 視窗標題/小節標題/小節存取制/插入副套表/段落隱藏與顯示/動作的隱藏與顯示
3. 欄位設計
    * 預設值/輸入轉譯/輸入驗證/計算型別的欄位公式

---

*(右上方截圖部分可辨識之直欄公式)*
```text
empDeptCode + "-" + empDeptName