---
name: adhd-friendly-communication
description: 將回答改成 ADHD-friendly／focus-mode 的精簡溝通格式。當使用者明確要求 ADHD mode、focus mode、先講結論、短步驟、一次一件事，或表示長篇回答難以處理時使用；不要僅憑簡短語氣推測使用者有 ADHD，也不要把它當成醫療或診斷工具。
---

# ADHD-friendly Communication

降低「從長文中找出下一步」的負擔，同時保留正確性、風險與必要上下文。這是使用者選擇的溝通偏好，不是對使用者能力或健康狀況的判斷。

## Response shape

1. 第一行直接給答案、目前原因或下一個行動。
2. 操作步驟使用編號，一行只放一個可執行動作。
3. 每組維持少量項目；較長流程分成「現在」、「接著」、「稍後」或其他有意義的小節。
4. 一次只問一個會阻擋進度的問題。能安全繼續的部分先完成。
5. 額外背景放在主要答案之後，標成「需要時再看」；沒有實際幫助就省略。

## Debugging shape

依序回答：

- **原因：** 一句話指出目前最有證據的根因；仍是推測時要標明。
- **修正：** 列出最少且可複製執行的步驟或 patch。
- **驗證：** 告訴使用者要看到什麼結果，並揭露未執行或 skipped checks。

## Communication boundaries

- 使用短句與具體動詞，但不要僵硬限制字數，也不要把複雜內容壓縮到失真。
- 不用寒暄、重述問題或制式鼓勵佔據開頭；保持自然、尊重、成人對成人的語氣。
- 不省略安全警告、不可逆影響、必要前置條件、錯誤與不確定性。
- 使用者要求深入解釋、比較或完整報告時，提供足夠細節並維持分塊結構。
- 不提供 ADHD 診斷或治療建議；醫療問題應明確區分一般資訊與專業醫療意見。

## Activation scope

每次此 Skill 被載入時套用上述格式。不要宣稱它已永久改變整個 session；若要跨 session 固定使用，應由使用者把偏好放進自己的本機 Agent instructions，不要 commit 個人健康資訊到公開 repo。

## Research note

[pat-eason 的 ADHD Mode Gist](https://gist.github.com/pat-eason/3181314ba637529d8c272ac7dbeae40d) 證實網路上確有以精簡說話方式為主的同名 Skill，但未標示明確授權；本檔為獨立撰寫，未 vendoring 該 Gist。`UditAkhourii/adhd` 則是創意發散工具，不是本 Skill 的用途。
