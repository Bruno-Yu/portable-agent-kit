---
name: karpathy-guidelines
description: 在撰寫、修改、重構或審查程式碼時，約束 Agent 先管理不確定性、採最小方案、精準修改並用成功條件驗證；也適用需要長時間、多步驟執行的 coding task。不要用於單純翻譯或不涉及工程判斷的一般問答。
---

# Karpathy Guidelines

以原始四條 Karpathy-inspired 規則為核心，再加入八條適合長時間 Agent workflow 的社群擴充。前四條來自社群對 Andrej Karpathy 公開觀察的整理；不要把完整十二條描述成 Andrej 親自發布。

## Core four

1. **Think before coding.** 明確說明會影響實作的假設、歧義與取捨。不確定且不同解讀會改變結果時，先問；有更簡單方案時直接提出。
2. **Simplicity first.** 只做需求所需的最小方案。不要加入推測功能、一次性抽象、未要求的彈性或套件。
3. **Surgical changes.** 只碰必要檔案與行數，只清理本次改動造成的殘留。不要順手重構、重新格式化或刪除既有無關程式。
4. **Goal-driven execution.** 先把任務轉成可驗證的成功條件，再執行、檢查並迭代到達成。

## Eight workflow extensions

5. **Model for judgment.** 分類、摘要、草擬與模糊判斷可交給模型；routing、retry、精確轉換與其他確定性工作交給程式或工具。
6. **Respect budgets.** 使用者指定的 token／時間預算是硬限制。避免重複載入大段內容；context 開始漂移時先摘要狀態，但不可因此省略必要驗證。
7. **Surface conflicts.** 兩種 pattern 衝突時，不要平均成第三種。依較新、較近、較有測試證據的來源選擇，並指出另一種待清理。
8. **Read before writing.** 修改前讀目標 export、直接 callers、共用 utilities、設定與相鄰測試。不了解結構原因時先查證。
9. **Test intent.** 測試應在需求邏輯被破壞時失敗，並說明為何該行為重要；不要只證明目前實作可以執行。
10. **Checkpoint.** 重要步驟後摘要已完成、已驗證與剩餘工作。不能描述目前狀態時，停止並重新盤點。
11. **Follow conventions.** Codebase 既有慣例優先於個人偏好。認為慣例有害時提出證據，不要靜默建立競爭模式。
12. **Fail loud.** Skipped、partial、flaky、未執行與不確定都要明確回報；有任何未完成項目時，不宣稱全部完成或全部通過。

## Application

- 簡單 typo 或明顯單行修改可縮短流程，但仍要精準修改與驗證。
- 使用者明確選定的方案優先；本 Skill 不得擴張 scope 或自行新增外部操作權限。
- 高風險工作仍需遵守專案安全、審批與資料邊界；「簡潔」不代表跳過必要控制。

## Sources

- [doggy8088/andrej-karpathy-skills（原始四條，MIT）](https://github.com/doggy8088/andrej-karpathy-skills/tree/0bf99012a4e63f3370e8027215a715f1bed91059)
- [本 repo 的歸因與版本說明](../../10-Agent-Foundation/karpathy-principles.md)
