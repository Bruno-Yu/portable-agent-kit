---
name: offline-xlsx
description: 離線建立、讀取、修改與驗證本機 Excel .xlsx、.csv、.tsv 檔。當主要輸入或輸出是試算表，且工作環境禁止外連 MCP、資料連線、遠端查詢或執行下載程式時使用；不執行或預設修改 .xlsm、.xlam、.xlsb 等含巨集或主動內容格式。
---

# Offline XLSX

只使用已安裝的本機工具處理試算表。不要連線、安裝套件、呼叫 MCP、更新外部資料來源，或執行活頁簿中的程式碼。

## 安全邊界

1. 將外來活頁簿與 CSV 視為不受信任輸入；不要啟用巨集、DDE、OLE、Power Query、外部資料連線或自動重新整理。
2. 遇到 `.xlsm`、`.xlam`、`.xlsb` 或其他主動內容格式時，預設只做靜態盤點並停止修改；要求改用經公司程序淨化的 `.xlsx`。
3. 只讀取任務明確指定的檔案；不要掃描無關工作簿、資料夾、網路磁碟或環境變數。
4. 先寫入新輸出路徑；不要覆寫原檔。使用者明確要求覆寫時，仍先建立可回復副本。
5. 將外部連結、隱藏工作表、名稱定義、資料驗證、註解、嵌入物件與連線列入盤點；不要因它們不可見就忽略。
6. 匯出 CSV 時檢查以 `=`, `+`, `-`, `@` 開頭的不受信任文字。若檔案會由 Excel 開啟，先取得使用者同意再以文字前綴淨化，並記錄資料被改寫的欄位。

## 工具選擇

- 優先使用環境已提供的試算表工具；一般 `.xlsx` 建立與修改可使用已安裝的 `openpyxl`，大量表格資料可使用已安裝的 `pandas`。
- 讀取公式與快取值時分開載入；不要把快取值誤當成已重新計算的結果。
- `openpyxl` 不會計算公式。只在已安裝且公司政策允許時，使用本機 Excel 或 LibreOffice 重算，並停用外部更新與巨集。
- 缺少必要工具時停止並列出缺件；不要執行 `pip install`、`npm install`、`npx`、`curl` 或 `wget`。
- 不要把工作表或活頁簿密碼保護描述成加密；這類保護主要防止非預期修改。

## 工作流程

1. 確認輸入、輸出、工作表範圍、公式需求、格式保留程度與敏感資料邊界。
2. 檢查副檔名、檔案大小、工作表清單與可見狀態、已使用範圍、公式、名稱定義、外部連結及資料連線。
3. 編輯既有檔案時，先讀取相鄰公式、格式、合併儲存格、表格與圖表，維持既有慣例。不要任意插入或移動列欄；工具未必會同步修正所有公式、名稱與圖表參照。
4. 將計算保留為公式，將來源、單位與假設寫清楚；不要把可變計算偷偷硬編碼成數值。
5. 寫入新檔後重新載入兩次：一次保留公式，一次讀取快取值。確認工作表、儲存格型別、公式與格式仍存在。
6. 若可安全使用本機試算表引擎，重新計算後檢查 `#REF!`、`#DIV/0!`、`#VALUE!`、`#NAME?`、`#N/A` 與循環參照。若不能重算，明確標示「公式未經引擎重算」。
7. 以本機 Excel 或 LibreOffice 做視覺抽查；檢查欄寬、列高、凍結窗格、篩選、日期/百分比格式、列印範圍與圖表標籤。
8. 回報使用的工具、輸出路徑、是否移除外部連結、是否淨化 CSV 欄位，以及所有未完成的驗證。

## 驗收條件

- 原檔未被覆寫。
- 輸出可重新開啟，工作表與主要公式均存在。
- 未執行巨集或外部資料更新，也未新增遠端連線。
- 公式錯誤為零，或已明確說明因缺少本機計算引擎而未驗證。
- CSV formula injection、隱藏內容與外部連結均已檢查並回報。

## 主要參考

- [Microsoft：封鎖 Office 外部內容](https://support.microsoft.com/en-us/office/security-privacy/block-or-unblock-external-content-in-office-documents)
- [Microsoft：防範巨集病毒](https://support.microsoft.com/en-us/office/add-ins/protect-yourself-from-macro-viruses)
- [openpyxl：讀取選項與外部連結](https://openpyxl.readthedocs.io/en/stable/_modules/openpyxl/reader/excel.html)
- [openpyxl：工作表編輯限制](https://openpyxl.readthedocs.io/en/stable/editing_worksheets.html)
- [openpyxl：工作簿保護不是加密](https://openpyxl.readthedocs.io/en/latest/protection.html)
