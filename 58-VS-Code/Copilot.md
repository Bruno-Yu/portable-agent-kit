## `#` vs `@` 的本質差異

| 面向        | `@` 參與者                      | `#` 脈絡         |
| --------- | ---------------------------- | -------------- |
| **作用**    | 「用什麼能力去做」                    | 「用什麼資料當背景」     |
| **角色**    | 指定執行者 / 工具                   | 指定輸入 / 參考資料    |
| **單獨使用？** | ✅ 可以（例如 `@codebase 這是什麼專案？`） | ❌ 通常要搭 `@` 才有效 |
| **可疊加？**  | ✅ 多個 `@` 有限制（通常選一個）          | ✅ 可以多個 `#` 組合  |
💡 **黃金法則**：`@` 決定「誰做」，`#` 決定「用什麼做」。通常組合是：`#file:XXX @edit 做某事`

---

| 參與者                      | 最佳使用情境                 | 可直接貼上的範本                                                                   |
| ------------------------ | ---------------------- | -------------------------------------------------------------------------- |
| `@agent`                 | 多步驟、需要探索 + 修改的複雜任務     | `@agent 幫我在 Services/ 新增一個 OrderService，包含查詢、新增、刪除功能`                      |
| `@askQuestions`          | 需求模糊，讓 Copilot 先問清楚再動手 | `@askQuestions 我想重構登入流程，請先問我需要哪些資訊`                                        |
| `@browser`               | 查看 Swagger、前端頁面、線上文件   | `@browser 開啟 http://localhost:5050/swagger 告訴我有哪些 API 端點`                  |
| `@codebase`              | 找某功能在哪個檔案、追呼叫鏈         | `@codebase SampleController 的 Get 方法最終呼叫哪個 Service？`                       |
| `@createAndRunTask`      | 一鍵建構或跑測試               | `@createAndRunTask 建立一個 Task 執行 npm run build`                             |
| `@createDirectory`       | 初始化新模組的資料夾結構           | `@createDirectory 在 ui/src/ 下建立 pages/Order/ 資料夾`                          |
| `@createFile`            | 新增設定檔、元件、服務            | `@createFile 在 Services/ 新增 OrderService.cs，包含基本 class 骨架`                 |
| `@createJupyterNotebook` | 資料分析、實驗性程式碼            | `@createJupyterNotebook 建立一個分析 API 回應時間的 Notebook`                         |
| `@edit`                  | 局部小幅修改目前檔案             | `@edit 把這個函式的回傳型別改為 Task<IActionResult>`                                   |
| `@editFiles`             | 批次跨多檔案套用同一規則           | `@editFiles 把所有 Controller 裡的 Console.WriteLine 改成 _logger.LogInformation` |
| `@editNotebook`          | 調整既有 Notebook 儲存格      | `@editNotebook 在第 3 個儲存格加上資料清洗的步驟`                                         |
| `@execute`               | 執行建構、測試、工具命令           | `@execute 執行 dotnet build 並告訴我有沒有錯誤`                                       |
| `@workspace`             | 理解整個專案架構、全域搜尋          | `@workspace 這個專案的錯誤碼統一定義在哪裡？`                                              |
|                          |                        |                                                                            |

---

## `#` 脈絡速查（搭配 `@` 使用）


| 脈絡           | 詳細用途                          | 搭配參與者                   | 可直接貼上的範本                                                          |
| ------------ | ----------------------------- | ----------------------- | ----------------------------------------------------------------- |
| `#file`      | 指定某個檔案當脈絡，讓 Copilot 先讀這個檔案再回答 | `@edit` `@codebase`     | `#file:SampleController.cs @codebase 這個 Controller 用了哪些 Service？` |
| `#editor`    | 使用目前開啟且**聚焦**的編輯器內容           | `@edit` `@askQuestions` | `@edit #editor 幫我補上 error handling`                               |
| `#selection` | 只用目前選取的文字範圍當脈絡                | `@edit`                 | `@edit #selection 把這段改成 async/await 寫法`                           |
| `#codebase`  | 搜尋整個工作區，當做回答背景                | `@codebase` `@agent`    | `#codebase @codebase OrderService 在哪一個檔案？`                        |
| `#webSearch` | 連線到網路搜尋結果（如需查外部資訊）            | `@browser`              | `#webSearch @browser 搜尋 React 18 的新特性`                            |


| 脈絡           | 用途         | 範本                                      |
| ------------ | ---------- | --------------------------------------- |
| `#file`      | 指定某檔案當參考   | `#file:SampleController.cs 解釋這個檔案的結構`   |
| `#editor`    | 目前開啟的編輯器內容 | `@edit #editor 幫我補上缺少的 try-catch`       |
| `#selection` | 目前選取的程式碼   | `@edit #selection 把這段改成 async/await 寫法` |
| `#codebase`  | 整個工作區當脈絡   | `#codebase 這個錯誤可能是哪裡造成的？`               |
|              |            |                                         |

---

> 💡 實戰技巧：`@agent` + `#codebase` 是最強組合，適合「看懂現有程式碼後再修改」的場景。


---

## 完整 `@` 參與者實戰速查版（全部）

### **編輯 & 檔案操作**

|`@` 參與者|功能簡述|使用範例|
|---|---|---|
|`@edit`|編輯目前開啟的檔案|`@edit #selection 補上 null check`|
|`@editFiles`|批次編輯多個檔案|`@editFiles 把所有 console.log 改成 logger`|
|`@editNotebook`|編輯 Notebook 儲存格|`@editNotebook 第 3 個儲存格加上資料清洗`|
|`@createFile`|建立新檔案|`@createFile 在 Services/ 建立 OrderService.cs`|
|`@createDirectory`|建立資料夾|`@createDirectory 在 src/ 下建立 pages/Order/`|
|`@rename`|重新命名檔案或符號|`@rename 把 OldService.cs 改名為 NewService.cs`|
|`@readFile`|讀取檔案內容|`@readFile #file:SampleController.cs`|
|`@textSearch`|文字搜尋|`@textSearch 搜尋所有 "TODO" 註解`|
|`@fileSearch`|檔案搜尋|`@fileSearch 找所有 .cs 檔案`|

---

### **代碼理解 & 搜尋**

|`@` 參與者|功能簡述|使用範例|
|---|---|---|
|`@codebase`|語意搜尋整個工作區|`@codebase OrderService 在哪一個檔案？`|
|`@usages`|找某符號的所有用途|`@usages GetData 這個方法被呼叫了多少次？`|
|`@search`|全域搜尋|`@search 搜尋 "authentication"`|
|`@toolSearch`|工具搜尋|`@toolSearch 有沒有單元測試的工具？`|

---

### **任務 & 執行**

| `@` 參與者             | 功能簡述               | 使用範例                                |
| ------------------- | ------------------ | ----------------------------------- |
| `@agent`            | 委派給子代理處理複雜任務       | `@agent 幫我整個重構登入系統`                 |
| `@runCommand`       | 執行 shell 命令        | `@runCommand dotnet build`          |
| `@runInTerminal`    | 在終端執行命令            | `@runInTerminal npm run build`      |
| `@sendToTerminal`   | 傳送輸入到終端            | `@sendToTerminal npm test`          |
| `@execute`          | 執行建構 / 測試          | `@execute 執行 dotnet test`           |
| `@createAndRunTask` | 建立並執行 VS Code Task | `@createAndRunTask 建立 npm build 任務` |
| `@runTests`         | 執行測試               | `@runTests 執行所有單元測試`                |
| `@runSubagent`      | 執行子代理              | `@runSubagent Explore 快速探索架構`       |

---

### **終端 & 系統**

| `@` 參與者                | 功能簡述     | 使用範例                             |
| ---------------------- | -------- | -------------------------------- |
| `@terminal`            | 終端相關操作   | `@terminal 可以幫我解釋下目前終端機的內容嗎?`    |
| `@terminalLastCommand` | 取得終端最後命令 | `@terminalLastCommand 上一個命令是什麼？` |
| `@terminalSelection`   | 取得終端選取內容 | `@terminalSelection 告訴我選取的文字`    |
| `@getTerminalOutput`   | 取得終端輸出   | `@getTerminalOutput 顯示最近的輸出`     |
| `@killTerminal`        | 關閉終端     | `@killTerminal 關閉終端`             |

---

### **檔案管理**

|`@` 參與者|功能簡述|使用範例|
|---|---|---|
|`@listDirectory`|列出目錄內容|`@listDirectory 列出 Services/ 下的所有檔案`|
|`@openBrowserPage`|開啟瀏覽器頁面|`@openBrowserPage http://localhost:5050/swagger`|

---

### **Notebook 相關**

|`@` 參與者|功能簡述|使用範例|
|---|---|---|
|`@createJupyterNotebook`|建立新 Notebook|`@createJupyterNotebook 建立資料分析 Notebook`|
|`@getNotebookSummary`|取得 Notebook 摘要|`@getNotebookSummary 這個 Notebook 有多少個儲存格？`|

---

### **外部資源 & 查詢**

|`@` 參與者|功能簡述|使用範例|
|---|---|---|
|`@browser`|開啟瀏覽器看網頁|`@browser 開啟 Swagger 並解釋 API`|
|`@fetch`|抓取網頁內容|`@fetch 抓取 https://docs.microsoft.com 的內容`|
|`@github` / `@githubRepo`|搜尋 GitHub 代碼庫|`@githubRepo microsoft/vscode 搜尋擴充程式碼`|
|`@githubTextSearch`|GitHub 文字搜尋|`@githubTextSearch 搜尋 "React hooks" 的 example`|
|`@web`|網頁相關|`@web 搜尋最新的 TypeScript 文件`|

---

### **VS Code & 擴充**

| `@` 參與者             | 功能簡述                | 使用範例                                                                                        |
| ------------------- | ------------------- | ------------------------------------------------------------------------------------------- |
| `@vscode`           | 查詢 VS Code 功能 & 快捷鍵 | `@vscode 如何快速開啟檔案？`                                                                         |
| `@extensions`       | 查詢或管理擴充             | `@extensions 有沒有好的 Linter 擴充？`                                                              |
| `@installExtension` | 安裝擴充                | `@installExtension 安裝 ESLint 擴充`                                                            |
| `@problems`         | 查看 VS Code 問題面板     | `@problems 有什麼編譯錯誤？`                                                                        |
| `@vscodeAPI`        |                     | 專門用於 **VS Code 擴充功能開發**，詢問 `vscode` Node.js API（如 `vscode.window`、`vscode.commands` 等）的用法\| |

---

### **記憶 & 工作區**

|`@` 參與者|功能簡述|使用範例|
|---|---|---|
|`@memory`|管理 Copilot 記憶|`@memory 保存我的開發偏好`|
|`@newWorkspace`|建立新工作區|`@newWorkspace 建立 React 專案`|
|`@getProjectSetupInfo`|取得專案設定資訊|`@getProjectSetupInfo 這是什麼型態的專案？`|

---

### **工具 & 視圖**

|`@` 參與者|功能簡述|使用範例|
|---|---|---|
|`@read`|讀取檔案|`@read src/main.ts`|
|`@renderMermaidDiagram`|渲染 Mermaid 圖|`@renderMermaidDiagram 畫出系統架構圖`|
|`@resolveMemoryFileUri`|解析記憶檔案 URI|`@resolveMemoryFileUri`|
|`@todo`|待辦清單|`@todo 列出目前的任務`|
|`@selection`|取得目前選擇內容|`@selection 分析選中的程式碼`|

---

## 完整 `/` 命令實戰速查版（全部）

|`/` 命令|功能簡述|使用範例|
|---|---|---|
|`/explain`|解釋程式碼或概念|`/explain 這個函式做什麼？`|
|`/fix`|修正問題|`/fix 這個有錯誤嗎？`|
|`/new`|新建東西|`/new 建立新的 React 元件`|
|`/newNotebook`|新建 Notebook|`/newNotebook 資料分析 Notebook`|
|`/tests`|測試相關|`/tests 幫我寫單元測試`|
|`/setupTests`|設定測試環境|`/setupTests 設定 Jest`|
|`/compact`|簡潔模式|`/compact 用簡潔方式回答`|
|`/clear`|清空對話|`/clear 重新開始`|
|`/fork`|分支對話|`/fork 從這裡開始新對話`|
|`/rename`|重新命名|`/rename 把對話改名`|
|`/debug`|偵錯模式|`/debug 顯示偵錯資訊`|
|`/agents`|查看可用代理|`/agents 有哪些代理可用？`|
|`/hooks`|設定 Hooks|`/hooks 配置自定義 Hook`|
|`/instructions`|查看或設定指令|`/instructions 設定開發規範`|
|`/models`|查看或選擇模型|`/models 切換到 Claude`|
|`/plugins`|管理外掛|`/plugins 安裝外掛`|

---

> 💡 **核心用法**：`@` = 能力工具（誰做），`/` = 操作模式（怎麼做）。通常組合是 `@agent` + `/explain` 或 `@edit` + `#selection` + `/fix`