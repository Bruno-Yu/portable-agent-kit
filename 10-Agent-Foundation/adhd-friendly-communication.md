---
title: ADHD-friendly Communication
tags:
  - agent/principles
  - accessibility
  - communication
source: https://gist.github.com/pat-eason/3181314ba637529d8c272ac7dbeae40d
last_verified: 2026-08-19
status: public-review
---

# ADHD-friendly Communication

這是輸出與對話結構的 accessibility preference：先給答案、一次一個行動、把長任務分塊，降低使用者在長段落中尋找下一步的成本。它不是醫療建議、診斷工具，也不應只因使用者說話簡短就推測其健康狀況。

## 網路上有兩種同名方向

1. [pat-eason 的 ADHD Mode Gist](https://gist.github.com/pat-eason/3181314ba637529d8c272ac7dbeae40d) 是精簡回答與步驟化輸出的 speaking style，符合本 repo 要找的用途；但 Gist 未標示明確授權，因此不直接複製。
2. [UditAkhourii/adhd](https://github.com/UditAkhourii/adhd) 是 MIT 授權的創意發散／多框架思考工具，不是說話方式。Standalone `SKILL.md` 可只用提示詞運作；完整 CLI／library 則使用 Node.js、Agent SDK 與多次模型呼叫。本 repo 因用途不同而不整合，不把兩者混為一談。

本 repo 因此提供原創的 `30-Skills/adhd-friendly-communication/SKILL.md`：只有 Markdown，只在使用者明確要求 ADHD-friendly、focus mode、短步驟或表示長文難以處理時使用。

## 使用原則

- 第一行直接給結論或下一個行動。
- 一次只要求使用者做一件事；需要選擇時一次問一個問題。
- 長任務分成「現在／接著／稍後」，每組保持少量項目。
- Debug 先列原因，再列修正與驗證。
- 必要的風險、前置條件與 skipped checks 不可為了短而省略。
- 使用者要求完整解釋時恢復所需細節，不用僵硬字數限制。
- 保持成人、尊重的語氣，不幼兒化、不製造假性急迫感。
