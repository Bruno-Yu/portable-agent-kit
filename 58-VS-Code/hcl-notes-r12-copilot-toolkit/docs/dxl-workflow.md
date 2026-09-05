# DXL／ODP 工作流程

## 專案狀態與依據

目前分類：Tier A／待確認，DXL／ODP 預設唯讀。下列是團隊工作原則，不宣稱 HCL 保證所有設計元素可 round-trip。

NSF／NTF 與由 Designer 匯出的基準是核對設計變更的依據；由專案負責人確認哪一份是核准基準。匯出文字不能取代實際 NSF 行為證據。基準來源、匯出版本／設定／日期、涵蓋元素、比對方式與回復負責人皆記入 [環境盤點](environment.md)，目前待確認。

DXL 是 Domino 的 XML 表示；ODP 是 Designer 的磁碟專案工作方式。不能假設 ODP 每個檔案都是 DXL，或 DXL 匯入等同 ODP 同步。實際檔案結構、同步方向、內容與 metadata 配對需從本專案證據確認。

## 變更前

1. 本次初始化只編寫文件，以下修改及驗證步驟均是未來任務，不在本次執行。
2. 未有明確修改授權及專案驗證流程前，保留 Tier A，只讀、比對、回報。
3. 日後允許文字修改時，先確認可還原副本與既有未提交修改，鎖定單一設計元素及呼叫／共用依賴。
4. 新增元素先由 Designer 建立並匯出最小基準。基準缺少時列出需取得的元素種類與屬性，停止該新增，不從零猜 XML 骨架。
5. 保留不相關 XML 宣告、DOCTYPE、命名空間、版本、識別資訊、`noteinfo`、`$` 內部欄位、編碼／BOM、換行及元素順序。不得為降噪刪除 metadata。
6. 不套用全檔 XML 自動格式化，不順便重排 `par`／`pardef` 或 DXL 版面，不改不相關屬性。

## 比對與驗證關卡

| 關卡 | 必須留下的證據 | 本範本狀態 |
|---|---|---|
| 基準核對 | Designer／NSF 基準代號、匯出方式、目標元素及日期 | 待確認 |
| 最小文字 diff | 改動檔案、設計元素、事件及原／新行為、共用依賴影響 | 尚未執行；本次無程式改動 |
| XML 語法（適用時） | 已核准解析器、版本、結果；禁止外部實體解析與網路 DTD 存取 | 尚未執行 |
| Designer 匯入或 ODP 同步（依實際流程） | 隔離 TEST 副本、實際操作、匯入日誌、成功／失敗元素 | 尚未執行；本次禁止 |
| Designer 編譯 | 適用元素及 Script Library 依賴、錯誤／警告、版本 | 尚未執行；本次禁止 |
| Notes／Domino 行為 | Client／Server 情境、正常／錯誤／權限案例、預期及實際結果 | 尚未執行；本次禁止 |
| 回復及審核 | 回復基準可用性、審核者與尚未解決事項 | 待確認 |

XML 語法有效不代表匯入成功；匯入成功不代表編譯與執行成功。只完成文字檢查時必須直說「僅靜態檢查」。不適用的關卡要寫理由，不能當作通過。若未執行，必須標「尚未執行」，不能勾選完成。

即使某元素驗證成功，也不能推論整個 NSF 可 round-trip。原始碼、編譯結果及實機行為有差異時，保留證據交由負責人判定；不全面重編試錯。簽署、ACL／ECL 變更與部署需要另案流程及授權。

## 官方來源及適用限制

核對日期：2026-09-05。

- [HCL Designer 12：NotesDXLImporter.InputValidationOption](https://help.hcl-software.com/dom_designer/12.0.0/basic/H_INPUTVALIDATIONOPTION_PROPERTY_IMPORTER.html) 說明輸入驗證選項；這不是本專案可成功 round-trip 的證明。
- [HCL Designer 12：Programmer's pane](https://help.hcl-software.com/dom_designer/12.0.0/basic/H_WRITING_LOTUSSCRIPT_IN_THE_PROGRAMMER_S_PANE.html) 說明 Designer 的程式編輯、編譯及錯誤顯示功能；本機是否可用仍需確認。
