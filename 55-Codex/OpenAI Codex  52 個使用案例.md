
https://developers.openai.com/codex/use-cases
https://cooljerry-chang.github.io/codex-case/
[harness engineering](https://openai.com/index/harness-engineering/)

OpenAI Codex 官方整理了 52 個使用案例，幫你知道Codex能怎麼實際應用。每個案例的頁面結構都一樣：說明適合對象、列出操作步驟、標註需要哪些 Skill 外掛（例如 Gmail、Figma、GitHub、Vercel 等），最後附上一條可以直接貼進去跑的範例 Prompt。52 個案例全部都有範例 Prompt，官方案例連結我整理成一個可以搜尋篩選的中文翻譯網頁在下方，也可以從那邊下載整理好的sheet案例表格

整理後大概分這幾類：
- 自動化（Automation）
- 跨工具串接（Integrations）
- 資料處理（Data）
- 工程開發（Engineering）
- 前端開發（Front-end）
- iOS / macOS 開發
- 評估與品質（Evaluation / Quality）



- [Codex 使用思維入門指南]( https://cdn.openai.com/pdf/6a2631dc-783e-479b-b1a4-af0cfbd38630/how-openai-uses-codex.pdf) :   多種團隊內部使用的案例和最佳實踐。
	把 Codex 的用途從「幫我寫程式」擴展成「幫我理解、重構、測試、探索、維持開發節奏」這種更成熟的使用方式。
	1. 先用 Ask Mode 想清楚，再用 Code Mode 執行: 這很重要。不要一開始就叫 Codex 改一堆檔案，而是先讓它分析、規劃、指出風險，再進入實作。
	2. 把 prompt 寫得像 GitHub Issue : 包含檔案路徑、背景、期望行為、參考模組、限制條件。這比「幫我修一下」有效很多。
	3. Codex 很適合做「理解陌生程式碼」 : 新手常以為 Codex 只是生成功能，其實它更大的價值之一是幫你快速讀懂 repo、追資料流、找核心邏輯。
	4. 用它補測試、找邊界案例 : 這對工程品質很實際，尤其是你不確定哪裡該測、哪些 failure path 容易漏掉時。
	5. http://AGENTS.md 的概念很關鍵 : 讓 Codex 長期理解專案規則、命名慣例、業務邏輯，而不是每次都從零開始猜。 ([中文版]([繁體中文翻譯：OpenAI Codex 使用思維入門指南 PDF ，OpenAI 團隊內部是怎樣使用 Codex 的？ | Patreon](https://www.patreon.com/posts/158824884))