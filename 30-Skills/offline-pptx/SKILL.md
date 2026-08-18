---
name: offline-pptx
description: 離線建立、讀取、修改與驗證本機 PowerPoint .pptx 檔。當主要輸入或輸出是簡報，且工作環境禁止外連 MCP、雲端字型、遠端圖片、網路模板或執行下載程式時使用；不處理或啟用 .pptm、.ppam、.potm 等含巨集格式。
---

# Offline PPTX

只使用已安裝的本機工具與使用者提供的素材處理 `.pptx`。不要連線、安裝套件、呼叫 MCP，或自動下載字型、圖片與模板。

## 安全邊界

1. 將外來簡報視為不受信任輸入；保留 Protected View，不要啟用巨集、ActiveX、OLE、外部連結或連結媒體。
2. 遇到 `.pptm`、`.ppam`、`.potm` 或其他主動內容格式時停止，說明風險並要求改用經公司程序淨化的 `.pptx`。
3. 只讀取任務明確指定的檔案與本機素材；不要掃描無關目錄。
4. 先複製到新輸出路徑再修改；不要覆寫原檔。使用者明確要求覆寫時，仍先建立可回復副本。
5. 不要解除 Office 安全警告、變更 Trust Center，或把不受信任路徑加入 Trusted Location。
6. 不要把機密內容送往外部服務；所有轉檔、渲染與檢查都留在本機。

## 工具選擇

- 優先使用環境已提供的簡報工具或 `python-pptx` 處理基本建立、讀取與修改。
- 只在已安裝且公司政策允許時，使用本機 PowerPoint 或 LibreOffice 做渲染與人工預覽。
- 缺少必要工具時停止並列出缺件；不要執行 `pip install`、`npm install`、`npx`、`curl` 或 `wget`。
- 若需求涉及工具不可靠支援的動畫、SmartArt、內嵌物件或高保真母片編輯，先說明限制，不要用破壞性近似方案偷偷取代。

## 工作流程

1. 確認輸入、輸出、受眾、投影片比例、語言與是否必須沿用既有模板。
2. 檢查副檔名、檔案大小、投影片數、版面配置、母片、備忘稿、內嵌物件與外部關聯；不要執行其中內容。
3. 編輯既有簡報時，先讀取立即相關的版面與樣式，維持既有慣例。新建簡報時，先定義一個一致的字型、色彩與間距系統。
4. 先完成敘事結構，再製作投影片。每頁表達一個主要訊息，讓標題直接陳述結論。
5. 將圖片、圖表與表格視為資訊，不要只作裝飾。只嵌入已核准的本機資產，不建立外部連結。
6. 輸出新檔後重新開啟，確認檔案可解析、投影片數正確、文字未遺失、圖片已嵌入且沒有意外外連。
7. 使用本機渲染器逐頁預覽；檢查溢位、裁切、重疊、字型替代、低對比與不一致對齊。
8. 回報使用的工具、輸出路徑、完成的驗證，以及因缺少 PowerPoint 或字型而無法驗證的項目。

## 驗收條件

- 原檔未被覆寫。
- 輸出是可重新開啟的 `.pptx`，頁數與預期一致。
- 不含巨集格式、外部資料連線、遠端圖片或新增的外部關聯。
- 已完成內容與視覺檢查；任何未檢查項目均明確揭露。

## 主要參考

- [Microsoft：Protected View](https://support.microsoft.com/en-us/office/collab-files/what-is-protected-view)
- [Microsoft：封鎖 Office 外部內容](https://support.microsoft.com/en-us/office/security-privacy/block-or-unblock-external-content-in-office-documents)
- [Microsoft：防範巨集病毒](https://support.microsoft.com/en-us/office/add-ins/protect-yourself-from-macro-viruses)
- [python-pptx 官方文件](https://python-pptx.readthedocs.io/en/stable/)
