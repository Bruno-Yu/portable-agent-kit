---
title: Karpathy-inspired 工程原則
tags:
  - agent/principles
  - engineering
source: https://github.com/doggy8088/andrej-karpathy-skills
last_verified: 2026-08-19
status: public-review
---

# Karpathy-inspired 工程原則

核心精神是讓 Coding Agent 管理不確定、避免過度工程、精準修改，並以可驗證的成功條件驅動工作。

## 原始四個核心原則

1. 寫程式前先思考：說明假設、歧義與取捨。
2. 簡潔優先：只做需求必要的最小方案。
3. 精準修改：不要順手重構無關程式碼。
4. 目標驅動：先定義驗證方式，再循環到完成。

這四條是 doggy8088／multica-ai 將 Andrej Karpathy 對 Coding Agent 常見失敗的觀察，整理成可執行規則的版本；不是 Andrej 親自發布的「八條守則」。

## 社群追加的八條

後來常見的完整版本是「原始 4 條 + 社群擴充 8 條 = 12 條」：

5. 模型只處理需要判斷的工作；routing、retry 與確定性轉換交給程式。
6. Token budget 是硬限制；避免重複讀取，context 漂移前先摘要。
7. 發現衝突就指出並選擇較新或較有測試證據的模式，不混合兩套做法。
8. 寫之前先讀 exports、直接 callers、共用 utilities 與相鄰測試。
9. 測試要驗證需求意圖，不只是讓目前實作顯示綠燈。
10. 重要步驟後建立 checkpoint：已完成、已驗證、尚未完成。
11. 遵循 codebase 慣例；認為慣例有害時先提出，不默默另創一套。
12. Fail loud：skipped checks、部分完成與不確定性必須明確揭露。

本 repo 根目錄 `AGENTS.md` 採用完整 12 條版本。不要把後八條全部歸因於 Andrej Karpathy，也不要引用未經原始測試資料支持的「錯誤率下降」數字。

## 可攜 Skill

需要在支援 Skill discovery 的 Agent 使用時，複製 `30-Skills/karpathy-guidelines/`；它只有一份 `SKILL.md`，不執行 hooks、scripts 或網路請求。

## 本 vault 的其他擴充

公開安全、內容語言與 publication checks 是本 repo 的維護規則，不屬於上述 12 條。

## 使用方式

- 跨平台核心：放在專案根目錄 `AGENTS.md`
- Claude Code 特有內容：另放薄版 `CLAUDE.md`
- Cursor：轉成 `.cursor/rules/*.mdc`
- 不要同時維護數份完整副本；平台檔只補差異

## 來源

- [doggy8088/andrej-karpathy-skills](https://github.com/doggy8088/andrej-karpathy-skills)
- [社群 12 條整理版本](https://github.com/twj515895394/andrej-karpathy-skills-12)
- [AGENTS.md 開放格式](https://agents.md/)
