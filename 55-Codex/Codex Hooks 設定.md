
[你還在用超長的 System prompt 來讓 Agent 聽話？馬上學會 Hooks，像個高手一樣駕馭 Agent 吧！]([你還在用超長的 System prompt 來讓 Agent 聽話？馬上學會 Hooks，像個高手一樣駕馭 Agent 吧！ | Patreon](https://www.patreon.com/posts/ni-huan-zai-yong-158551055))

Hooks 的使用場景


- Codex 開始一個 session → 自動加入工作提醒
- Codex 要執行 terminal 命令前 → 自動檢查是否危險
- Codex 回合結束後 → 自動提醒它是否要跑測試
- 你送出 prompt 前 → 自動檢查有沒有 API key
全域 hooks 會套用到所有 Codex 專案，位置通常是

> ~/.codex/hooks.json。

專案 hooks 只會在該專案生效，位置通常是

> <你的專案>/.codex/hooks.json。

- **SessionStart：** 是在 Codex session 開始時觸發。這很適合拿來加背景提醒，例如工作語言、回覆風格、改動前先簡短說明，或者任何你希望它一開始就記住的規矩。

- **UserPromptSubmit：** 是在你送出 prompt 時觸發。這種 hook 很適合做送出前檢查，例如看看 prompt 裡有沒有敏感資訊、有沒有把不該貼的 key 或 token 一起送出去。

- **PreToolUse：** 它是在 Codex 使用「支援的工具」之前觸發，例如 Bash、apply_patch、MCP tools。它很適合攔截危險 command，但官方也說它是 guardrail，不是完整安全邊界，因為不是所有工具路徑都一定會被攔截，例如 WebSearch 不屬於這類攔截範圍。

- **PostToolUse：** 是在工具執行之後觸發。這類 hook 比較適合拿來做後續檢查，例如某個步驟完成後自動補一層驗證，或者在特定工具跑完之後提醒你留意結果。它是在工具已經執行完之後觸發，所以**不能取消已經發生的副作用**。例如命令已經改了檔案，PostToolUse 不能把時間倒回去；它只能提醒、檢查輸出、要求後續處理。

- **Stop：** 是在 Codex 準備停下來時觸發。它最像最後一道關卡，所以很適合拿來提醒測試、提醒確認改動範圍，或者檢查這次任務是不是真的已經完成。