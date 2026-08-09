
###  `workbench.browser.enableChatTools`

 **實驗性（Experimental）** 的設定，用來啟用「**瀏覽器 Agent 工具（Browser Agent Tools）**」，讓 **GitHub Copilot AI** 能夠深度整合 VSCode 的內建瀏覽器，自主地建立、測試、除錯網頁應用程式。

| 功能                 | 說明                                |
| ------------------ | --------------------------------- |
| 🌐 **開啟 / 導覽網頁**   | AI 可以在 VSCode 內建瀏覽器中直接開啟並瀏覽網頁     |
| 📸 **截取螢幕截圖**      | AI 可以讀取頁面內容並截圖，以判斷畫面狀態            |
| 🖱️ **與頁面元素互動**    | AI 可以點擊按鈕、填寫表單等頁面操作               |
| 🧪 **自動化測試**       | AI 可以自行跑測試，驗證功能是否正常               |
| 🐛 **自動找 Bug 並修正** | AI 可讀取 Console 錯誤，自動修復 Bug，不需人工介入 |
|                    |                                   |


### 🛡️ 隱私與安全性

- AI 操作的瀏覽器為 **獨立隔離的 Session**
- **不共享** 你平常瀏覽的 Cookie 或任何個人資料
- 確保 AI 操作不會影響到你的一般瀏覽環境

```json
{
  "workbench.browser.enableChatTools": true
}
```