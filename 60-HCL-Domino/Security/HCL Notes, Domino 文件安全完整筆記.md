
> 主題：資料庫 ACL、文件 Readers／Authors、ACL Role，以及簽核流程中的權限變化。
> 對應影片：《7 Notes Domino 文件的安全性》。

## 1. 先建立整體模型

Domino 的文件安全不是只看一個設定，而是分成兩層：

```text
第一層：資料庫 ACL（DB 層）
  ├─ 使用者能不能進入資料庫
  ├─ 使用者的最高能力：Manager、Editor、Author、Reader…
  └─ 使用者具有哪些 ACL Role
                     ↓
第二層：文件 Readers／Authors（文件層）
  ├─ Readers：哪些人可以看到這份文件
  └─ Authors：ACL 為 Author 的人，哪些人可以修改這份文件
                     ↓
Domino 計算最終有效權限
```

最重要的記憶方式：

> ACL Level 決定「能力上限」；ACL Role 決定「身分類別」；Readers／Authors 決定「可以處理哪些文件」。

文件欄位只能在 ACL 已允許的範圍內進一步篩選，不能把使用者提升到超過 ACL 的權限。

---

## 2. 資料庫 ACL 等級

Domino 資料庫 ACL 的主要等級由高到低如下：

```text
Manager
  ↓
Designer
  ↓
Editor
  ↓
Author
  ↓
Reader
  ↓
Depositor
  ↓
No Access
```

| ACL 等級 | 主要能力（簡化） |
|---|---|
| Manager | 管理 ACL、資料庫設定，並具備較低等級的能力 |
| Designer | 修改 Form、View、Agent 等設計元素 |
| Editor | 建立文件，並修改所有自己看得到的文件 |
| Author | 建立文件；只能修改 Authors 中包含自己、所屬群組或 Role 的文件 |
| Reader | 只能讀取自己看得到的文件 |
| Depositor | 可以投入／建立文件，但通常不能讀取文件 |
| No Access | 一般情況下不能進入資料庫 |

上表是核心概念的簡化版；ACL 還有 `Create documents`、`Delete documents`、`Read public documents` 等額外權限選項。

參考：[HCL — Access levels in the ACL](https://help.hcl-software.com/dom_designer/14.5.1/basic/H_TABLE_OF_ACCESS_LEVELS_IN_THE_ACL_OVERVIEW.html)

---

## 3. Readers 與 Authors 是什麼？

`Readers` 與 `Authors` 是 Domino Designer 中的特殊欄位型別，不是普通的 Text 或 Names 欄位。

| 欄位型別 | 控制內容 |
|---|---|
| Readers | 哪些人、群組、Role 或伺服器可以讀取這份文件 |
| Authors | 哪些 ACL 等級為 Author 的使用者可以修改這份文件；列在其中的人也取得讀取資格 |

欄位名稱可以自由命名，真正產生安全效果的是欄位型別：

```text
AllowedReaders       // 欄位名稱
Readers              // 欄位型別

CurrentApprover      // 欄位名稱
Authors              // 欄位型別
```

普通 Names 欄位即使保存了使用者姓名，也不會自動產生文件安全限制。

### 每個 Form 都預設有這些欄位嗎？

不是。設計者必須自行：

- 在 Form 中建立 Readers／Authors 型別欄位；或
- 透過文件安全設定建立單份文件的讀取限制；或
- 使用 Formula、LotusScript 等程式建立對應的 Readers／Authors Item。

若文件沒有任何讀取限制，則由資料庫 ACL 決定誰可以讀取。

參考：[HCL — To create Readers and Authors fields](https://help.hcl-software.com/dom_designer/14.0.0/basic/H_RESTRICTING_WHO_CAN_READ_OR_EDIT_SPECIFIC_DOCUMENTS_.html)

---

## 4. ACL 與 Authors：誰可以編輯文件？

判斷編輯權限時，先看 ACL Level，再看使用者是否能讀到文件。

| 資料庫 ACL 等級 | Authors 是否包含使用者、群組或 Role | 可以修改文件嗎？ |
|---|---:|---:|
| Manager | 不需要 | ✅ 可以，但必須先看得到文件 |
| Designer | 不需要 | ✅ 可以，但必須先看得到文件 |
| Editor | 不需要 | ✅ 可以，但必須先看得到文件 |
| Author | 否 | ❌ 不可以 |
| Author | 是 | ✅ 可以 |
| Reader | 即使包含 | ❌ 不可以 |

### 為什麼是這樣？

- `Editor` 以上本來就可以修改所有自己看得到的文件，Authors 不會限制他們。
- `Author` 必須符合文件內至少一個 Authors 值，才可以修改該文件。
- `Reader` 的 ACL 上限只有讀取；Authors 不能把 Reader 升級成 Author。
- 即使是 Manager，若被 Readers 擋住而看不到文件，也無法直接修改；Full Access Administration 是另外的特殊管理模式。

因此 Authors 的主要用途是：

> 在所有 ACL 為 Author 的使用者之中，再決定誰可以修改這一份文件。

---

## 5. ACL 與 Readers：誰看得到文件？

| 文件狀態 | 使用者情況 | 看得到文件嗎？ |
|---|---|---:|
| 沒有 Readers／Read access 限制 | ACL 本身至少允許讀取 | ✅ 可以 |
| 有 Readers 限制 | Readers 包含使用者、所屬群組或 Role | ✅ 可以 |
| 有 Readers 限制 | Readers 沒有使用者，但 Authors 包含使用者、群組或 Role | ✅ 可以 |
| 有 Readers 限制 | Readers 與 Authors 都沒有使用者 | ❌ 不可以 |
| 有 Readers 限制 | Manager 也未被 Readers／Authors 包含 | ❌ 一般模式下仍看不到 |

需要特別注意：

- Readers 可以限制所有一般 ACL 等級，包含 Manager。
- Authors 中的名稱除了可能取得修改資格，也會取得讀取資格。
- `No Access` 仍不能靠 Readers／Authors 進入資料庫，因為文件欄位不能突破 ACL 上限。
- Server Administrator 使用 Full Access Administration 時，才可以繞過一般 Readers 限制。

所以影片中「大家都看得到」更精確的說法是：

> 文件沒有 Readers 限制時，所有已通過資料庫 ACL、具有讀取能力的人都看得到。

參考：[HCL — Using a Readers field to restrict access](https://help.hcl-software.com/dom_designer/14.0.0/basic/H_ABOUT_RESTRICTING_ACCESS_TO_SPECIFIC_DOCUMENTS_USING_A_READERS_FIELD.html)

---

## 6. 最終權限的判斷順序

遇到一位使用者與一份文件，可以依序問：

### 6.1 能不能讀？

```text
ACL 是否允許讀取？
  ├─ 否 → 看不到
  └─ 是
      ├─ 文件沒有 Readers 限制 → 看得到
      └─ 文件有限制
          ├─ 符合 Readers → 看得到
          ├─ 符合 Authors → 看得到
          └─ 都不符合 → 看不到
```

### 6.2 能不能修改？

```text
先確認看得到文件
  ↓
ACL 是 Editor 以上？
  ├─ 是 → 可以修改
  └─ 否
      ├─ ACL 是 Author，而且符合 Authors → 可以修改
      └─ 其他情況 → 不可以修改
```

---

## 7. 「DB 層控制」與 ACL Role

影片所說的 DB 層控制，是把「哪個人屬於哪個角色」集中設定在資料庫 ACL；文件只引用 Role 名稱，不在每一份文件中寫死全部人名。

例如資料庫 ACL：

| ACL Entry | ACL 等級 | 指派的 Role |
|---|---|---|
| 王主管 | Author | `[SysAdmin]` |
| 陳稽核 | Reader | `[DocReaders]` |
| User01 | Author | 無 |

文件只需要保存：

```text
CurrentApprover（Authors） = User01
SystemAuthors（Authors）   = [SysAdmin]
SystemReaders（Readers）   = [SysAdmin]、[DocReaders]
```

Domino 會把使用者在 DB ACL 中取得的 Role，拿來比對文件 Readers／Authors：

- `User01` 符合目前簽核者 Authors，因此可以修改。
- 王主管具有 `[SysAdmin]`，符合 SystemAuthors，因此可以修改。
- 陳稽核具有 `[DocReaders]`，符合 SystemReaders，因此只能閱讀。

如果系統管理員從王主管換成李主管，只需要在 DB ACL 中重新指派 `[SysAdmin]`。所有引用 `[SysAdmin]` 的文件都會套用新的角色成員，不必逐份修改文件。

### Role 也不能突破 ACL Level

- ACL 是 Reader：即使具有 `[SysAdmin]` 並符合 Authors，仍不能修改。
- ACL 是 Author：符合 `[SysAdmin]` Authors 後才可以修改。
- ACL 是 No Access：即使符合 `[DocReaders]`，仍不能進入資料庫。
- Author 若要刪除文件，除了符合 Authors，還需要 ACL 的 `Delete documents` 權限。

---

## 8. 為什麼知道 `[SysAdmin]`、`[DocReaders]` 是自訂 Role？

判斷依據如下：

1. **方括號是 Domino ACL Role 的表示法。** HCL 官方文件說明，Role 會以 `[Sales]` 這類形式顯示。
2. **Role 是單一資料庫專屬的。** Role 必須先由具有 Manager 權限的人在該資料庫 ACL 中建立，再指派給個人、群組或伺服器。
3. **它們不是 ACL Level。** Domino 固定的 ACL Level 是 Manager、Designer、Editor、Author、Reader 等；`SysAdmin` 與 `DocReaders` 並不在這套固定等級中。

因此：

```text
[SysAdmin]
[DocReaders]
```

代表這個應用／資料庫自行定義的 ACL Role。模板可能預先建立 Role，但它仍屬於該資料庫設計，不是所有 Domino 資料庫固定存在的全域 Role。

參考：[HCL — Roles in the ACL](https://help.hcl-software.com/dom_designer/14.5.1/basic/H_USING_ROLES_IN_A_DATABASE_ACL_OVERVIEW.html)

---

## 9. Readers／Authors 可以保存哪些身分？

| 類型                  | Formula Language 範例                    | 說明                   |
| ------------------- | -------------------------------------- | -------------------- |
| 個人完整階層名稱            | `"CN=User01/O=Acme"`                   | 最明確的個人名稱寫法           |
| 個人縮寫階層名稱            | `"User01/Acme"`                        | 同網域中常見               |
| Domino Directory 群組 | `"Purchasing"`                         | 群組名稱不使用方括號           |
| ACL Role            | `"[SysAdmin]"`                         | 方括號表示目前資料庫的 ACL Role |
| 目前登入者               | `@UserName`                            | 公式計算後保存當時的使用者名稱      |
| 既有多值欄位              | `Approvers`                            | 使用另一個欄位的名稱清單         |
| 伺服器                 | `"CN=Domino01/O=Acme"`                 | 需要讀取或複寫受限文件時可能加入     |
| 多種來源組合              | `@UserName : Approvers : "[SysAdmin]"` | 合併成多值文字清單            |

角色與群組最容易混淆：

```formula
"Purchasing"      // Domino Directory 群組
"[Purchasing]"    // 目前資料庫中的 ACL Role
```

兩者可能名稱相同，但管理位置不同：群組在 Domino Directory／LDAP 管理；Role 在個別資料庫 ACL 管理。

---

## 10. 為什麼多個 Role 使用冒號 `:`？

在 Domino Formula Language 中，ASCII 半形冒號 `:` 是 **List concatenation operator（清單串接運算子）**。

```formula
"[SysAdmin]" : "[DocReaders]"
```

它產生的是包含兩個成員的文字清單：

```text
1. [SysAdmin]
2. [DocReaders]
```

冒號不是被存進資料中的文字分隔符；它是建立清單的公式運算子。

同理：

```formula
@UserName : Approvers : "[SysAdmin]"
```

會把目前使用者、`Approvers` 原有清單及 `[SysAdmin]` 合併成同一個多值清單。

### 常見錯誤

```formula
"[SysAdmin],[DocReaders]"
```

這通常只是一個包含逗號的單一字串，不是兩個清單成員，可能造成沒有任何人符合文件權限。

正確寫法：

```formula
"[SysAdmin]" : "[DocReaders]"
```

補充：

- 程式碼應使用 ASCII 半形冒號 `:`，不是中文全形冒號 `：`。
- 分號 `;` 在 Formula Language 中通常用來分隔公式陳述式，不是建立清單。
- Notes Client 或 Document Properties 可能以分號或換行顯示多值欄位；那只是畫面呈現。文件實際保存的是多個文字值。

參考：[HCL — List concatenation operator](https://help.hcl-software.com/dom_designer/12.0.0/basic/H_LIST_OPERATOR.html)

---

## 11. 套回影片的簽核流程

投影片的流程為：

```text
申請人 → 直屬主管 → 經理 → IT 承辦人 → 採購人員
```

流程使用兩組文件安全欄位。

### 11.1 隨關卡變化的動態欄位

| 欄位用途 | 欄位型別 | 功能 |
|---|---|---|
| 目前簽核者 | Authors | 讓目前關卡的人修改文件 |
| 目前關卡代理人 | Authors | 讓代理人代為處理 |
| 簽核過的人 | Readers | 讓處理過的人繼續查看，但不能再修改 |
| 此文件群組讀者 | Readers | 讓指定業務群組可以查看 |

例如目前輪到 `User01`：

```text
CurrentApprover（Authors） = User01
```

進到下一關後，程式會更換目前簽核者，並可能把上一位簽核者加入 Readers。

### 11.2 不隨關卡變化的系統欄位

```text
SystemAuthors（Authors） = [SysAdmin]
SystemReaders（Readers） = [SysAdmin]、[DocReaders]
```

若這兩個值由欄位公式產生，公式結果可分別寫成：

```formula
"[SysAdmin]"

"[SysAdmin]" : "[DocReaders]"
```

這些固定 Role 是管理、救援與稽核通道，目的是避免產生無人可處理的「孤兒文件」。

假設目前簽核者離職、帳號停用，或程式將錯誤名稱寫入 Readers／Authors：

- 一般使用者可能無法再處理文件。
- Readers 設錯時，連一般模式下的 Manager 都可能看不到文件。
- `[SysAdmin]` 成員仍可介入修正或重新指派。
- `[DocReaders]` 成員仍可進行固定的查看、稽核或支援工作。

| 類別 | 用途 |
|---|---|
| 動態 Authors／Readers | 表達目前簽核流程狀態 |
| 系統 Authors／Readers | 保留固定管理、救援及稽核通道 |

Authors 本身也會賦予讀取資格，因此 `[SysAdmin]` 已在 SystemAuthors 時，純粹就「能看見文件」而言，再放進 SystemReaders 通常不是必要條件。重複列入可能是為了讓設計意圖明確、方便維護，或作為防禦性配置。

---

## 12. 綜合案例

假設 ACL 與文件內容如下：

### 資料庫 ACL

| 使用者 | ACL 等級 | Role |
|---|---|---|
| User01 | Author | 無 |
| User02 | Author | 無 |
| 王主管 | Author | `[SysAdmin]` |
| 陳稽核 | Reader | `[DocReaders]` |

### 文件安全欄位

```text
CurrentApprover（Authors）   = User01
PreviousApprovers（Readers） = User02
SystemAuthors（Authors）     = [SysAdmin]
SystemReaders（Readers）     = [SysAdmin]、[DocReaders]
```

其中 `CurrentApprover`、`SystemAuthors` 是 Authors；`PreviousApprovers`、`SystemReaders` 是 Readers。

### 最終結果

| 使用者 | 看得到？ | 能修改？ | 原因 |
|---|---:|---:|---|
| User01 | ✅ | ✅ | ACL 是 Author，且符合 CurrentApprover Authors |
| User02 | ✅ | ❌ | 只符合 PreviousApprovers Readers |
| 王主管 | ✅ | ✅ | ACL 是 Author，且具有 `[SysAdmin]` Role |
| 陳稽核 | ✅ | ❌ | ACL 是 Reader，且具有 `[DocReaders]` Role |
| 其他未列入者 | ❌ | ❌ | 文件有限制，但不符合任何 Readers／Authors |

---

## 13. 最終記憶口訣

```text
ACL Level    = 能力上限
ACL Role     = DB 層集中管理的身分類別
Readers      = 哪些人或身分能看這份文件
Authors      = 哪些 Author 能改這份文件，並同時取得讀取資格
冒號 :       = Formula Language 中建立／合併多值清單
方括號 [ ]   = ACL Role 的表示方式
```

最後用一句話總結整套設計：

> 資料庫 ACL 管理使用者的最高能力與角色歸屬；文件 Readers／Authors 利用人名、群組或 Role 細分單份文件的讀寫權；工作流程用動態欄位控制當前關卡，再以固定系統 Role 保留管理與救援通道。

## 14. 採購單簽核流程實作（OCR 整理）

### 14.1 建立流程控制 Subform

建立流程控制 Subform：`Workflow Admin`，並放到採購單最上方。

### 14.2 建立流程欄位

1. 建立關卡序號欄位：

   | 設定 | 內容 |
   |---|---|
   | 欄位名稱 | `WCurFlowNum` |
   | 欄位型別 | 文字型欄位 |
   | Default 值 | `0` |

2. 建立文件作者與讀者欄位：

   | 欄位名稱 | 安全欄位型別 | 設定 |
   |---|---|---|
   | `wDocAuthor` | Authors（作者） | 計算型、多重值，預設值為 `@UserName` |
   | `wDocReader` | Readers（讀者） | 多重值 |

> OCR 說明：原圖第 2 點將兩個欄位寫在同一行：`wDocAuthor 作者計算型欄位，多重值，預設值：@UserName / wDocReader 讀者（多重值）`。此處依斜線拆成兩個欄位整理。

### 14.3 定義採購簽核關卡

```text
0 申請人 → 1 主管 → 2 採購員 → 99 結案
```

| `WCurFlowNum` | 關卡 |
|---:|---|
| `0` | 申請人 |
| `1` | 主管 |
| `2` | 採購員 |
| `99` | 結案 |

### 14.4 建立 Submit 動作

在採購單建立 `Submit` 動作。按下 Submit 時，判斷目前的關卡序號。

#### A. 目前關卡為 `0`

1. 關卡改為 `1`。
2. `wDocAuthor` 寫入：

   ```text
   CN=user01/O=is-land
   ```

3. `WCurFlowAStatus` 寫入：

   ```text
   N1
   ```

#### B. 目前關卡為 `1`

1. 狀態改為 `2`。
2. `WDocAuthor` 寫入：

   ```text
   CN=user02/O=is-land
   ```

3. `WCurFlowAStatus` 寫入：

   ```text
   PS1
   ```

#### C. 目前關卡為 `2`

1. 狀態改為 `99`。
2. 清空 `WDocAuthor`。
3. `WCurFlowAStatus` 寫入：

   ```text
   結案
   ```

#### D. 累積文件讀者

將目前人員新增至 `wDocReader$` 多重值。

#### E. 儲存文件

執行存檔。

#### F. 關閉文件

執行關閉。

### 14.5 Submit 流程摘要

| Submit 前的關卡 | Submit 後的關卡 | 新的文件作者 `wDocAuthor` | `WCurFlowAStatus` |
|---:|---:|---|---|
| `0` 申請人 | `1` 主管 | `CN=user01/O=is-land` | `N1` |
| `1` 主管 | `2` 採購員 | `CN=user02/O=is-land` | `PS1` |
| `2` 採購員 | `99` 結案 | 清空 | `結案` |

每次 Submit 還會執行：

1. 將目前人員加入 `wDocReader$` 多重值。
2. 儲存文件。
3. 關閉文件。

### 14.6 暫存按鈕思考題

上述 Submit 動作成功控制採購單簽核狀態後，可再思考「暫存」按鈕應如何設計。

Submit 與暫存的主要差異可以先理解成：

- Submit：推進簽核關卡，更新 Authors／Readers 與流程狀態。
- 暫存：只保存目前內容，通常不應推進簽核關卡，也不應更換下一位文件作者。

> 上述「暫存」差異是依流程語意整理，原圖第 5 點只要求思考暫存按鈕的作法，未提供實作細節。

### 14.7 `WCurFlowAStatus` 下拉選單

備註：第五章作業已完成 `WCurFlowAStatus` 下拉選單。

```text
草稿|Draft
主管|N1
採購員|PS1
結案|Released
```

| 顯示文字 | 儲存值／別名 |
|---|---|
| 草稿 | `Draft` |
| 主管 | `N1` |
| 採購員 | `PS1` |
| 結案 | `Released` |

### 14.8 OCR 與命名注意事項

- 原圖在 A 使用 `wDocAuthor`，在 B、C 使用 `WDocAuthor`，大小寫並不一致；實作時建議統一欄位名稱。
- 原圖在 B、C 寫「狀態改 2／99」；依前文「Submit 時判斷目前關卡序號」推斷，這裡應是將 `WCurFlowNum` 更新為 `2／99`，但本 OCR 未直接改寫原文。
- 原圖的 `wDocReader$` 帶有 `$`，看起來像 LotusScript 的字串變數名稱；若實際文件欄位名稱是 `wDocReader`，實作時需區分變數與文件 Item。
- A、B 的階層名稱原圖行尾帶有逗號；此處視為句子標點，未納入名稱值。
- C 的 `WCurFlowAStatus` 原圖寫入「結案」，但下拉選單的別名為 `Released`；實作前應確認該欄位實際保存顯示文字還是別名。
