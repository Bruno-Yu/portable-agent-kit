# HCL Notes／Domino R12 Copilot Setup Toolkit

版本：文件範本 v1；官方來源核對：2026-09-05。公司環境實測：尚未執行。

用途：讓 Copilot 盤點公司專案並建立六份共用規範。只有 Markdown 文件，不含安裝腳本、套件、MCP 或自動執行機制。使用者仍需發出一次初始化任務。

## 放在哪裡

維護版留在 portable-agent-kit 的本子目錄，沿用既有 HCL 知識來源；公司實際程式及盤點結果留在公司專案。這樣先避免兩個公開 repo 重複維護相同原則。若未來需要不同存取權、發布週期或獨立維護團隊，再拆 repo。

本子目錄可單獨打包，內部相對連結均不依賴外層知識庫。不要將公司程式、真實環境資訊或盤點結果回傳本公開 repo。

## 使用方式

### A. 目前沒有公司專案／先建立空白規範工作區

1. 將此 toolkit 整個資料夾解壓到獨立目錄，以 VS Code「開啟資料夾」直接開啟它。確認 .github 隱藏目錄也在其中。
2. 使用公司允許、具檔案讀寫工具的 Copilot 聊天模式，將 SETUP.md 加入聊天附件，貼上下面的啟動指令。
3. 空資料夾缺乏程式證據時保留待確認；日後有公司專案再使用 B 流程。不要把 portable-agent-kit 根目錄視為這個空白專案。

### B. 已有公司專案（保留原有文件）

1. 用 VS Code 開啟公司單一專案根目錄。
2. 將 toolkit 複製到該工作區的 `_copilot-setup` 資料夾，只作來源；若同名資料夾已存在，先比較，不覆蓋。這是手動放入的輸入範本，SETUP 只會建立／合併目標根目錄的六份文件。
3. 將 `_copilot-setup/SETUP.md` 加入 Chat 附件，貼上啟動指令。不要直接把範本 .github、docs、DESIGN.md 覆蓋既有同名檔案。
4. 完成後用下方驗收指令開始新 Chat。團隊共用的是目標根目錄的六份文件；來源資料夾是否保留由團隊決定，初始化不自動刪除。

### 啟動指令（複製到 Copilot Chat）

```text
請依附上的 SETUP.md，實際完成目前開啟專案的 Copilot／HCL Notes R12 規範文件初始化。
先確認唯一目標根目錄，讀取既有指令、盤點必要檔案與未提交修改，再建立或合併指定六份文件。
本次授權僅限這六份規範文件；請保留原有內容，未知標待確認，有衝突列出來源並繼續不受影響部分。
不要只提出計畫。完成後驗證相對連結與變更範圍，回報已驗證／待確認／尚未執行。
```

若 Chat 只能回答而不能編輯，這次初始化尚未完成；切換到公司允許的可編輯模式，或由人員套用其提供的六份內容。不得為此解除公司政策。

## 六份團隊文件

| 文件 | 作用 |
|---|---|
| [.github/copilot-instructions.md](.github/copilot-instructions.md) | 精簡工作區入口與讀取順序 |
| [docs/environment.md](docs/environment.md) | 版本、來源、工具、環境與衝突證據 |
| [docs/coding-style.md](docs/coding-style.md) | 已觀察慣例、建議及短例子 |
| [docs/dxl-workflow.md](docs/dxl-workflow.md) | 基準、最小修改與分層驗證 |
| [docs/debugging.md](docs/debugging.md) | 現有除錯與尚未實作診斷規格 |
| [DESIGN.md](DESIGN.md) | 實際介面證據及待核定提案 |

初始化程序見 [SETUP.md](SETUP.md)。不另外引入自訂 Agent 或 Skill discovery；本版將需要的工程原則與 HCL 注意事項整理為明確可讀文件。

## 新 Copilot Chat 驗收指令

先開全新 Chat，第一次不要手動附入口檔，用來觀察工作區指令是否自動套用：

```text
只做唯讀驗收，不改任何檔案、不執行程式或連線服務。
請分開列出「本輪可確認已自動附加的指令來源」與「為本次驗收主動讀取的文件」；
無法觀察自動附加來源時請明說，不得用猜測代替。
請讀取並引用本工作區的 Copilot 入口、environment、coding-style、dxl-workflow、debugging、DESIGN，
各給一項具體規則及路徑／章節，再回答：
1. 新增 DXL 設計元素前需要什麼？只有 XML 語法通過可以宣稱哪些事情？
2. debug mode 與 logger 現在是已存在、待確認還是尚未實作？請提供文件證據。
3. 介面類型及完整 HCL 版本是否已確認？哪些視覺規範只是提案？
4. 列出這次禁止的操作、規範衝突與尚未執行的驗證。
缺檔或連結失效就回報失敗，不自行補造內容。最後列出尚缺的最少資訊。
```

驗收看兩種證據：回答內容是否忠於六份文件，以及公司版本介面提供的 References／使用的參考資料或 Chat 診斷是否顯示工作區指令。若介面沒有可用證據，只能確認能讀取文件，不能宣稱已驗證自動載入。手動附檔成功也不等於自動載入成功。

## 已核對的 Copilot 機制與限制

VS Code 官方文件說明，工作區根目錄的 .github/copilot-instructions.md 可自動加入聊天；多份指令可能一起加入，不應以檔名或讀取順序推定可解決衝突。這類指令不適用於編輯時的 inline suggestions。詳見 [VS Code custom instructions](https://code.visualstudio.com/docs/agent-customization/custom-instructions)。

因此只下載或開啟 repo 不會自動替你初始化；本版使用手動啟動文字，避免綁定特定 prompt-file 或 Agent 介面。公司安裝版本、可編輯模式與受管政策仍要實測。規範連結失效或文件讀取被阻擋時，必須回報，不能默認內容已進入模型上下文。

## 既有資料的取用與差異

下列來源已在 portable-agent-kit 本機工作樹檢閱；這裡記錄來源路徑供維護者追溯，不是公司專案需讀取的外部相依。

| 原有來源 | 本版取用／差異處理 |
|---|---|
| AGENTS.md、10-Agent-Foundation/karpathy-principles.md 所述工程基線 | 取用先讀再改、最小範圍、證據與未完成事項明示。公司入口使用可解析的 Markdown 相對連結；工具箱索引仍保留 wikilink 慣例。 |
| 00-Quickstart/codex-company-dxl-start.md 與 templates/AGENTS.company-dxl.example.md | 原版全面唯讀、禁止初始化寫檔；本次使用者明確授權六份文件初始化，其餘 DXL／NSF 操作仍禁止。保留舊 Codex 指南，不複製 Codex profile 或路由規則。 |
| 60-HCL-Domino/Version-Control/HCL Domino 12 NSF 版控與 AI 輔助開發規劃（v2）.md | 取用 NSF 基準、Tier A 與 source／bytecode 風險；其中「Swiper 全員必裝」不納入，因本次禁止安裝。舊文待另案檢討。 |
| 60-HCL-Domino/Development/HCL Domino 12 開發學習手冊.md | 取用宣告與 Client／Server 分辨方向；Option Declare 改列新模組建議，避免直接套入所有舊程式。 |
| 60-HCL-Domino/LotusScript/LotusScript — Error Handling 筆記.md | 取用錯誤資訊與處理概念；不採納範例中普遍 Resume Next 的寫法作為團隊預設，改按可恢復性決定；語法以 HCL R12 官方文件核對。 |

本機盤點：原 repo 有 README、AGENTS、教學與 Codex 設定；未找到獨立 DXL／ODP 程式檔、LotusScript 原始檔或專案測試／建置檔。筆記中的程式碼不代表公司現況。沒有據此宣稱任何公司程式慣例或版本已確認。

## 完成條件

- 六份文件存在，相對連結有效，沒有覆蓋原有公司規範。
- 建議、待確認與尚未實作清楚分開；本次差異只涉及六份文件。
- 未安裝、未修改程式、未操作 NSF／NTF、未改設定。
- 回報所有跳過的檢查；Designer／公司 Copilot 尚未實測時不得寫成通過。

本 toolkit 維護版另含本 README 與 SETUP，共八份 Markdown。下載包不含公司來源、工具箱其他設定或可執行程式。
