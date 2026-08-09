
## 1. 先理解 View 是什麼

在 Notes/Domino 中，Document 是真正儲存資料的單位；View 則是把符合條件的 Document 挑出來，整理成清單。

例如員工 Document 中有：

```text
姓名：王小明
部門：DP01
```

View 可以使用選取公式決定要收錄哪些文件：

```formula
SELECT Form = "Employee"
```

Notes 不會在每次開啟 View 時，都從頭掃描所有 Document。它會先建立一份 **View Index（視圖索引）**，記錄哪些文件應該出現在 View 中，以及它們的排序和分類結果。

因此可以把 View 理解為：

```text
View 設計
  ├─ Selection Formula：挑選哪些 Document
  ├─ Columns：顯示哪些欄位
  ├─ Sort／Category：如何排序、分類
  └─ View Index：依照上述設計產生的結果清單
```

---

## 2. Shared View

Shared View 是多人共用的 View。一般使用者只要具有足夠的資料庫讀取權限，就可以使用它。

它最重要的特性是：

```text
一個 Shared View 設計
一份共用的 View Index
多人讀取相同的索引結果
```

例如「全部員工」View：

```formula
SELECT Form = "Employee"
```

這個條件對所有人都相同，很適合使用 Shared View。無論王小明或李小華開啟它，View 都是列出全部員工。

### Shared View 適合的情況

- 所有人應該看到相同的清單結構。
- 選取條件不會因目前使用者而改變。
- 需要集中維護 View 設計。
- 希望設計更新後，所有使用者都能取得新版設計。

---

## 3. 為什麼 Shared View 不適合直接用 `@UserName`

假設要建立「我的文件」View，選取公式寫成：

```formula
SELECT Owner = @UserName
```

直覺上可能認為，每位使用者打開 View 時，`@UserName` 都會變成該使用者的名稱。但 Shared View 的 View Index 是共用的，選取公式並不會為每位開啟者各自重新執行一次。

可能發生的過程如下：

```text
1. Tom 先開啟 Shared View。
2. Notes 建立或更新共用 View Index。
3. 當時計算出的 @UserName 是 Tom。
4. 索引因此收錄 Tom 的文件。
5. Jay 後來開啟同一個 Shared View。
6. Jay 讀取的是已存在的共用索引。
7. 索引內容仍然是依 Tom 建立的結果。
```

問題不是 `@UserName` 不知道目前登入者是誰，而是：

> `@UserName` 在建立 View Index 時已經被計算；其他使用者開啟 Shared View 時，通常只是讀取既有的共用索引。

一份共用索引不可能同時是 Tom、Jay 和 Amy 三人的個人化結果：

```text
同一份 Shared View Index
  ├─ Tom 希望它只包含 Tom 的文件
  ├─ Jay 希望它只包含 Jay 的文件
  └─ Amy 希望它只包含 Amy 的文件

→ 三種條件互相衝突
```

因此，含有 `@UserName` 的個人化 View Selection Formula，通常應搭配 Private View 或 Private on First Use。

---

## 4. Private View

Private View 是某一位使用者自己的 View，其他使用者不能直接使用這份私人 View。

它具有自己的 View 設計或私人副本，也有自己的索引，因此可以依擁有者產生個人化結果：

```text
Tom 的 Private View
  └─ Tom 的 View Index

Jay 的 Private View
  └─ Jay 的 View Index
```

如果選取公式是：

```formula
SELECT Owner = @UserName
```

各自的結果便可以是：

```text
Tom 的私人索引 → @UserName = Tom → 收錄 Tom 的文件
Jay 的私人索引 → @UserName = Jay → 收錄 Jay 的文件
```

### Private View 適合的情況

- 每位使用者需要自己的文件清單。
- View Selection Formula 會依 `@UserName` 改變。
- 使用者需要個人化的欄位、排序或分類方式。
- 不希望個人的 View 設定影響其他使用者。

---

## 5. Shared, Private on First Use

`Shared, Private on First Use` 可以理解為：

> 設計者先發布一份共用範本；每位使用者第一次開啟時，Notes 再替他建立私人副本。

流程如下：

```text
設計者建立共用 View 範本
        │
        ├─ Tom 第一次開啟 → 建立 Tom 的 Private View
        ├─ Jay 第一次開啟 → 建立 Jay 的 Private View
        └─ Amy 第一次開啟 → 建立 Amy 的 Private View
```

它解決了兩個需求：

1. 設計者可以一次把 View 範本提供給所有人。
2. 每位使用者仍然能擁有獨立索引，正確使用 `@UserName`。

### 私人副本存放位置

一般的 `Shared, Private on First Use`：

- 使用者有權在資料庫建立私人 View 時，私人副本通常存在 NSF 資料庫內。
- 使用者沒有相關權限時，Notes 可以把私人副本存入使用者本機的 Desktop 檔案。

---

## 6. Shared, Desktop Private on First Use

`Shared, Desktop Private on First Use` 的運作方式與上一種相同：使用者第一次開啟時，會得到私人副本。

差異只在私人副本的存放位置：

| View 類型 | 私人副本位置 |
|---|---|
| Shared, Private on First Use | 通常在 NSF 內；權限不足時可能放在本機 Desktop 檔案 |
| Shared, Desktop Private on First Use | 強制放在使用者本機的 Desktop 檔案 |

這裡的 **Desktop／Local** 是指 View 的私人設計及索引存在使用者電腦，不代表 View 只能顯示本機 NSF 裡的 Document。

可能造成的結果是：

- 使用者換一台電腦時，原本的本機私人 View 不一定會跟過去。
- 私人 View 不會像一般共享設計一樣由伺服器集中提供。

---

## 7. Designer 中的鑰匙圖示

Designer 裡的鑰匙是「私人 View 狀態」的提示，不是 Document 之間的關聯 Key。

一般可這樣理解：

```text
普通鑰匙
→ Private View

帶有小「1」的鑰匙
→ Private on First Use
```

需要注意：`Shared, Private on First Use` 與 `Shared, Desktop Private on First Use` 的原始設計圖示可能沒有清楚差異。因此不能只根據鑰匙判斷私人副本究竟存在 NSF 還是本機，仍應查看建立 View 時選擇的 View Type。

鑰匙與前面 Embedded View 使用的 `ProjectNo`、`empDeptCode` 完全是不同概念：

| 名稱 | 用途 |
|---|---|
| Designer 的鑰匙圖示 | 表示 View 是 Private 或 Private on First Use |
| `ProjectNo`、`empDeptCode` | 用來配對或篩選不同 Document |

---

## 8. Private View 不是安全機制

Private View 只代表「這份 View 屬於某位使用者」，不代表裡面的 Document 已受到安全保護。

例如某份 Private View 沒有顯示機密文件，不代表使用者無法經由其他 View、搜尋或程式找到該文件。

真正的資料存取安全應使用：

- Database ACL
- Reader Names 欄位
- Author Names 欄位
- 其他 Notes/Domino 安全性設定

可以這樣區分：

```text
View Selection Formula
→ 決定清單怎麼顯示

ACL／Reader Names
→ 決定使用者有沒有權限讀取 Document
```

---

## 9. 設計更新的注意事項

使用者的 Private on First Use 副本建立後，就成為獨立的私人設計，通常不會自動繼承原始 View 後續的設計更新。

例如設計者之後在範本中新增一個欄位：

```text
共用範本：已增加新欄位
Tom 的既有私人副本：仍然是舊設計
```

若要取得新版設計，通常需要刪除舊的私人副本，再重新開啟原始的 Private on First Use View，讓 Notes 重新建立私人副本。

---

## 10. 快速比較

| 特性 | Shared View | Private View | Shared, Private on First Use | Shared, Desktop Private on First Use |
|---|---|---|---|---|
| 誰能使用 | 多位使用者 | 單一擁有者 | 每人第一次使用後取得私人副本 | 每人第一次使用後取得本機私人副本 |
| View Index | 多人共用 | 個人獨立 | 個人獨立 | 個人獨立、存於本機 |
| 適合 `@UserName` 個人化選取 | 不適合 | 適合 | 適合 | 適合 |
| 集中發布初始設計 | 是 | 否 | 是 | 是 |
| 後續自動繼承範本更新 | 是 | 否 | 私人副本通常不會 | 私人副本通常不會 |
| 私人副本位置 | 不適用 | NSF 或本機，視權限而定 | 通常 NSF；權限不足時可能為本機 | 強制為本機 Desktop 檔案 |

---

## 11. 一句話記憶

```text
Shared View
= 大家共同看同一份整理好的清單

Private View
= 每個人有自己的清單和索引

Private on First Use
= 公司先發一份範本，第一次使用時再影印成個人版本

Desktop Private on First Use
= 個人版本固定放在自己的電腦
```

最核心的判斷方式是：

> 如果選取結果必須因使用者身分而不同，就不能只依賴一份共用的 Shared View Index；需要讓每位使用者擁有自己的私人索引。

## 參考資料

- [HCL Notes：Shared and private views and folders](https://help.hcl-software.com/notes/12.0.2/client/fram_types_of_views_r.html)
- [HCL Domino Designer：Shared and private views](https://help.hcl-software.com/dom_designer/10.0.1/basic/H_ABOUT_SHARED_VIEWS.html)

---

## 12. 補充：View 製作的重要注意事項

影片最後特別提醒兩件事：

1. View 的 Selection Formula 與 Column Formula 不支援 `@DbColumn`、`@DbLookup`。
2. 在 View 中使用 `@Now`、`@Today` 會影響 View Index 的執行效率。

這兩項限制都和前面介紹的 **View Index** 有關。設計 View 時，應優先讓公式只依賴目前正在建立索引的 Document 內已有且相對穩定的欄位。

可以先記住以下原則：

> View 適合整理 Document 已經存好的資料，不適合在建立索引時，再到其他 View 查資料或持續詢問現在時間。

### 12.1 View 公式有兩個主要位置

這裡說的「View 公式」主要包括：

| 公式位置 | 作用 |
|---|---|
| View Selection Formula | 決定哪些 Document 要進入 View Index |
| Column Formula | 決定每個 Document 在欄位中顯示什麼值 |

例如：

```formula
SELECT Form = "Employee" & Status = "Active"
```

這是 Selection Formula；它決定哪些員工文件會進入 View。

```formula
empName + " / " + empDeptName
```

這是 Column Formula；它決定該列顯示的文字。

`@DbColumn` 與 `@DbLookup` 在這兩種公式中都不支援。

---

## 13. 為什麼 View 不能使用 `@DbColumn`、`@DbLookup`

### 13.1 兩個函數原本是做什麼的

`@DbColumn` 用來取出某個 View 的整欄資料：

```formula
@DbColumn("Notes"; ""; "DepartmentLookup"; 1)
```

`@DbLookup` 則是用 Key 到另一個 View 中尋找相符資料：

```formula
@DbLookup("Notes"; ""; "DepartmentLookup"; empDeptCode; "DeptName")
```

這些函數很適合用在 Form 欄位、關鍵字清單或互動式公式中，但 HCL 明確規定它們不能用在 View 的 Selection Formula 或 Column Formula。

### 13.2 從 View Index 的角度理解

建立 View Index 時，Notes 要逐份處理 Document：

```text
讀取 Document A → 判斷是否進入 View → 計算各欄位
讀取 Document B → 判斷是否進入 View → 計算各欄位
讀取 Document C → 判斷是否進入 View → 計算各欄位
```

如果公式又要求到另一個 View 查資料，就會形成額外依賴：

```text
目前 View 的 Index
        ↓ 依賴
另一個 View 的 Index
        ↓
另一個 View 可能也正在更新或尚未建立
```

這會讓索引之間產生複雜的依賴、更新順序及一致性問題。因此 Notes 的 View Formula 執行環境直接不支援 `@DbColumn` 和 `@DbLookup`。

### 13.3 錯誤設計範例

不要在 Selection Formula 中寫：

```formula
SELECT Form = "Employee" &
       empDeptName = @DbLookup(
           "Notes"; ""; "DepartmentLookup";
           empDeptCode; "DeptName"
       )
```

也不要在 Column Formula 中寫：

```formula
@DbLookup(
    "Notes"; ""; "DepartmentLookup";
    empDeptCode; "DeptName"
)
```

這不是單純「效能不好」，而是該公式環境不支援這些函數。

### 13.4 建議的替代做法

最直接的做法，是在 Employee Document 儲存當下需要的值：

```text
Form = Employee
empDeptCode = DP01
empDeptName = 資訊處
```

View 便可以直接使用：

```formula
SELECT Form = "Employee" & empDeptCode = "DP01"
```

Column Formula 也只要寫：

```formula
empDeptName
```

依實際需求，也可以使用以下方式：

- 儲存 Document 時，先用 `@DbLookup` 算好結果並寫入欄位。
- 使用 Agent 定期查詢其他資料，再把結果寫回 Document。
- 若只是要顯示關聯明細，使用分類 View、Embedded View 與 Single Category。
- 若需要即時跨資料庫查詢，放在 Form、按鈕、Agent 或程式碼中執行，而不是放進 View Index Formula。

這種做法有一個取捨：若部門名稱從「資訊處」改成「資訊中心」，已存於員工文件的 `empDeptName` 不會自動改變。此時應根據需求決定：

- 只在員工文件存 `empDeptCode`，顯示時於 Form 查名稱。
- 或由 Agent 批次更新相關員工文件中的部門名稱。

不要為了避開限制，就默默接受資料可能不同步；同步策略需要明確設計。

---

## 14. 為什麼 `@Now`、`@Today` 會拖慢 View

### 14.1 View Index 假設結果可以保持穩定

一般 View Selection Formula 依賴 Document 中存好的欄位：

```formula
SELECT Form = "Employee" & Status = "Active"
```

只要 Document 沒有被新增或修改，判斷結果就不會變，因此 Notes 可以繼續使用原本的 View Index。

但是以下公式依賴目前時間：

```formula
SELECT Form = "Employee" & DateLine < @Now
```

即使任何 Document 都沒有修改，時間仍然不斷前進。某份文件在 11:59 還不符合條件，到 12:00 可能就突然符合條件。

因此索引很難回答：

```text
Document 沒變，但時間變了。
現在這份 View Index 還有效嗎？
```

### 14.2 `@Now` 的影響

`@Now` 包含日期和時間，值持續改變：

```text
10:00:00
10:00:01
10:00:02
```

使用 `@Now` 的時間敏感 View 很容易一直處於需要更新的狀態。資料量大時，會增加伺服器 CPU、磁碟 I/O、View Index 更新工作與使用者等待時間。

### 14.3 `@Today` 的影響

`@Today` 只取日期，看似一天只變一次，但它仍然是時間敏感公式：

```text
2026-07-20 23:59 → @Today = 2026-07-20
2026-07-21 00:00 → @Today = 2026-07-21
```

到了午夜，即使 Document 沒有改動，View 的選取結果也可能改變。HCL 文件特別提醒，在 Column Formula 或 Selection Formula 使用 `@Today` 可能降低效率，並讓 View Refresh Indicator 持續出現。

`@Today` 通常比精確到秒的 `@Now` 容易控制，但不能因此把它視為沒有成本。

---

## 15. 時間條件的替代設計

### 15.1 最推薦：用 Agent 更新狀態欄位

不要讓 View 每次根據現在時間重新判斷：

```formula
SELECT DueDate < @Today
```

可以每天由 Scheduled Agent 更新文件：

```text
DueDate < 今天 → IsExpired = "1"
DueDate >= 今天 → IsExpired = "0"
```

View Selection Formula 改成穩定欄位：

```formula
SELECT Form = "Task" & IsExpired = "1"
```

資料流會變成：

```text
排程 Agent 每天計算一次時間條件
             ↓
把結果存入 Document 的 IsExpired
             ↓
View 只根據 IsExpired 建立索引
```

優點是計算時間與頻率可以控制，也容易檢查為什麼某份文件被判定為逾期。

### 15.2 若必須使用時間公式：控制 Refresh 頻率

若業務需求允許 View 直接使用時間條件，應讓索引更新頻率配合資料精度：

```text
只需要每天更新
→ 可以考慮 Auto, at most every 24 hours

只需要每小時更新
→ 可以考慮 Auto, at most every 1 hour
```

不要設定比實際業務需求更頻繁的更新。例如「每日到期清單」沒有必要每分鐘重建。

需要注意：這種設定代表資料可能在兩次更新之間暫時不是最新狀態。必須先確認業務可以接受這段延遲。

---

## 16. View Formula 的實用設計原則

### 原則一：優先使用目前 Document 已儲存的欄位

推薦：

```formula
SELECT Form = "Task" & Status = "Open"
```

避免讓 View 在建立索引時查詢其他 View、其他資料庫或不斷變動的外部狀態。

### 原則二：把複雜計算放在適合的執行位置

| 需求 | 較適合的位置 |
|---|---|
| 顯示 Document 已存欄位 | View Column Formula |
| 決定哪些 Document 進入清單 | View Selection Formula |
| 使用者開啟 Form 時查詢資料 | Form Computed Field／事件 |
| 儲存文件前補齊資料 | Form 驗證、QuerySave 或其他儲存邏輯 |
| 跨文件或跨資料庫批次計算 | Agent、LotusScript 或 Java |
| 依 Key 顯示相關明細 | Categorized View＋Embedded View＋Single Category |
| 依日期定期變更狀態 | Scheduled Agent |

### 原則三：先確認函數支援的 Formula Context

看到一個 `@Function` 可以在 Form 使用，不代表它也可以在 View 使用。實作前應查看該函數文件中的 **Usage**，確認它是否支援：

- Selection Formula
- Column Formula
- Field Formula
- Agent Formula
- Web 或 Notes Client

### 原則四：避免隱藏的更新成本

View 看起來只是清單，但背後需要建立和維護索引。以下設計都應特別注意：

- 依目前使用者變化的 `@UserName`
- 依時間變化的 `@Now`、`@Today`
- 試圖查詢其他 View 的 `@DbLookup`、`@DbColumn`
- 非必要卻設定非常頻繁的 Index Refresh
- 很少使用但一直自動維護的大型 View

共同問題都是：View 的結果不再只依賴目前 Document 的穩定資料，索引便更難共用、快取或增量更新。

---

## 17. View 上線前檢查表

- [ ] Selection Formula 是否只依賴目前 Document 的欄位？
- [ ] Column Formula 是否避免使用不支援的函數？
- [ ] 是否誤用了 `@DbLookup` 或 `@DbColumn`？
- [ ] 是否使用 `@Now` 或 `@Today`？如果有，真的必要嗎？
- [ ] 時間條件能否改由 Scheduled Agent 更新狀態欄位？
- [ ] Index Refresh 頻率是否符合實際業務需求？
- [ ] 是否因 `@UserName` 而需要 Private on First Use？
- [ ] 大型且很少使用的 View 是否仍需要 Automatic Refresh？
- [ ] 複雜計算是否應移到 Form、Agent、LotusScript 或 Java？
- [ ] 是否已明確設計重複資料的同步方式？
- [ ] 測試時是否使用接近正式環境的 Document 數量，而不只是少量測試資料？

## 18. 這一段的最短記憶法

```text
View Index 喜歡：
目前 Document 已存好、穩定、可重複計算的資料。

View Index 不喜歡：
臨時查別的 View、因使用者改變、因現在時間持續改變的資料。
```

因此影片最後兩個注意事項可以濃縮成：

> 不要在 View Formula 中使用 `@DbColumn`、`@DbLookup`；也要盡量避免 `@Now`、`@Today`。如果需求確實存在，先把資料算好並存回 Document，或以可控制頻率的 Agent／Index Refresh 處理。

## 補充參考資料

- [HCL Domino Designer：@DbColumn](https://help.hcl-software.com/dom_designer/10.0.1/basic/H_DBCOLUMN_NOTES_DATABASES.html)
- [HCL Domino Designer：@DbLookup](https://help.hcl-software.com/dom_designer/14.5.1/basic/H_DBLOOKUP_NOTES_DATABASES.html)
- [HCL Domino Designer：@Today](https://help.hcl-software.com/dom_designer/11.0.1/basic/H_TODAY.html)
- [HCL Domino Designer：Refreshing view indexes](https://help.hcl-software.com/dom_designer/14.5.0/basic/H_ABOUT_REFRESHING_VIEW_INDEXES.html)
