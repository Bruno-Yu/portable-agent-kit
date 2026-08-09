- AI Credit & 企業方案說明 : [# Usage-based billing for organizations and enterprises]([Usage-based billing for organizations and enterprises - GitHub Docs](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-organizations-and-enterprises))
- 保哥分享:  **[GitHub Copilot CLI 模型使用成本一覽表.md](https://gist.github.com/doggy8088/1fc2fe9dfdbba01fb74d088034f2db15)**

GitHub 官方文件明確寫到：2026-06-01 起，從舊的 premium request-based billing 改成 usage-based billing。

 - GitHub Copilot 計費已正式改為「Usage-based billing（用量計費）」主軸，單位是 GitHub AI Credits（AIC），1 AIC = 0.01 USD。
- 模型用量牌價（每百萬 token 的 input/output/cached input，轉成 AIC）
- 組織/企業方案是共用池，不是每人固定不可共享桶
- 可設額外超額策略與預算上限（user/cost center/enterprise/org）

| Plan               | Total AI credits per user per month |
| ------------------ | ----------------------------------- |
| Copilot Business   | 1,900                               |
| Copilot Enterprise | 3,900                               |

現在 2026/06/01 - 09/01 是促銷期 (按照如下方式計算)

| Plan               | Total AI credits per user per month |
| ------------------ | ----------------------------------- |
| Copilot Business   | 3,000                               |
| Copilot Enterprise | 7,000                               |

企業方案：
- 每個 seat 提供的 AIC 會匯入「billing entity 的共享池」
- 官方範例：100 個 Copilot Business 使用者，形成 190,000 AIC 共享池（不是 100 個彼此隔離的小桶）
- 新增席次(seat)：池子立即增加
- 移除席次：通常下個 cycle 才反映縮減
- 是否可超額由政策控制：
	- 允許超額：按 published per-credit rates 計費
	- 不允許超額：用完就阻擋到下 cycle
	- 還可設 user/cost center/enterprise/org 多層預算控管
### 價格

- 價格: [# Models and pricing for GitHub Copilot]([Models and pricing for GitHub Copilot - GitHub Docs](https://docs.github.com/en/copilot/reference/copilot-billing/models-and-pricing))

官方明確說：

- 所有價格表是「每 1 million tokens」
- 依模型有不同 input/output/cached input 單價
- 超過包含額度後，以這些費率換算 AIC 收費（1 AIC = 0.01 USD）

你可以把它理解成：

- 訂閱牌價 = 入場費 + 每月含量
- 模型牌價 = 超過含量或做成本估算時的單位成本表

補充差異要點（官方文件有提）：

- 自動模型選擇在特定情境可有 10% 成本折扣（文件以 individuals 頁面為準）
