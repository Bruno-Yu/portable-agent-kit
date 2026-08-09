
- HCL Domino Designer Product Document (https://help.hcl-software.com/dom_designer/12.0.0/basic/H_WAYS_TO_USE_OPERATORS.html)
- **DAOS**（**D**omino **A**ttachment and **O**bject **S**ervice，Domino 附件與物件服務）



## Event

資料庫提供了相對應的 event ，使我們的系統可以在適當的時機觸發

- 資料庫 script
- 表單
- 欄位
- 按鈕

### 資料庫 Script

1. Postopen
	- 提醒相關事項
	- 檢查 DB 語言文件
2. Queryclose
	- 提醒相關事項
3. Querydocumentdelete-
	- 檢查額外權限，禁止刪除動作
	- 刪除權限，一起刪除子文件

4. Queryopen (query 是 open 前)
	- 判斷有無權限開啟文件
	- LockDoc
5. Postopen - 額外計算文件資料，ex: 帶申請人資料 （post 是 open 後）
6. Querymodechange 判斷有無權限可編輯
7. QuerySave - 判斷卡控權檢查
8. QueryClose - 提醒相關事項，LockDoc 解除
9. Queryopendocument - document 點兩下時觸發

``` vb
Sub Postopen(Source As Notesuidocument)

	OnError 4412 Resunme Next
	Dim s As New Notessession
	Dim ws As New NotesUIWorkspace
	Dim db As Notesdatabase
	Dim uidoc As NotesUIdocument
	Dim doc As Notesdocument
	Set db = s.currentdatabase
	Set uidoc = Source
	Set doc = source.Document

	Dim itemUser As notesitem

	Set itemUser = doc.getFirstItem("WDocauthor")

	If itemUser.contains(s.UserName) And Not (uidoc.InPreviewPane) Then
		Let uidoc.EdiMode = True
	End If

End Sub


```

## 權限

esigner 的 **Field Type**。控制文件權限的是兩個特殊型別：

- `Readers`：控制誰能看這份文件。
- `Authors`：控制 ACL 為 Author 的使用者能否修改；列在其中的人也能讀取文件。

它們在 Designer 裡是：

`Field Properties → Field Info → Type → Readers / Authors`

**不是每個 Form 預設都有。** 必須由設計者自行新增欄位並選擇這個型別，例如：

```
欄位名稱：AllowedReaders
欄位型別：Readers
欄位值：CN=Alice Chen/O=ExampleCorp
```

欄位名稱可以任意取；真正生效的是 `Readers` 型別。普通的 `Names` 或 `Text` 欄位即使放了人名，也不會產生安全限制。

同理：

###  誰能修改文件？

|ACL 等級|Authors 欄位有你？|能否修改|
|---|---|---|
|Manager／Designer／Editor|不需要|✅|
|Author|沒有|❌|
|Author|有|✅|
|Reader|即使有|❌|

原因是：

- `Editor` 以上本來就可以修改所有「看得到的」文件。
- `Author` 只能修改 Authors 欄位中包含自己姓名、群組或角色的文件。
- `Reader` 的 ACL 上限只有讀取；Authors 欄位不能把 Reader 升級成 Author，所以仍然不能修改。

### 誰看得到文件？

|情況|能否看到|
|---|---|
|文件沒有 Readers 欄位限制，ACL 至少是 Reader|✅|
|有 Readers 欄位，但裡面沒有你|❌|
|Readers 沒有你，但 Authors 有你|✅|

這裡最容易誤會的是：
- Readers 欄位會限制所有 ACL 等級，連 Manager 都可能看不到。
- Authors 欄位除了賦予符合條件的 Author 修改權，也會同時賦予該人讀取文件的資格。
- 管理者若啟用 `Full Access Administration`，才可以繞過這種文件讀取限制。[HCL 官方：Readers 欄位規則](https://help.hcl-software.com/dom_designer/14.0.0/basic/H_ABOUT_RESTRICTING_ACCESS_TO_SPECIFIC_DOCUMENTS_USING_A_READERS_FIELD.html)

可以把它記成兩道門：

1. **ACL 是資料庫大門與權限上限**：你最多能做到什麼。
2. **Readers／Authors 是文件門禁**：在資料庫裡，你能看或修改哪些特定文件。

所以圖上「大家都看得到」其實應更精確地說：

> 所有已經通過資料庫 ACL、至少具有讀取能力的人都看得到。

不是任何人都能看到。
