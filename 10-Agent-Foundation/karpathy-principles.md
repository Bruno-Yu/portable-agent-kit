---
title: Karpathy-inspired 工程原則
tags:
  - agent/principles
  - engineering
source: https://github.com/doggy8088/andrej-karpathy-skills
last_verified: 2026-08-09
status: public-review
---

# Karpathy-inspired 工程原則

核心精神是讓 Coding Agent 管理不確定、避免過度工程、精準修改，並以可驗證的成功條件驅動工作。

## 四個核心原則

1. 寫程式前先思考：說明假設、歧義與取捨。
2. 簡潔優先：只做需求必要的最小方案。
3. 精準修改：不要順手重構無關程式碼。
4. 目標驅動：先定義驗證方式，再循環到完成。

## 本 vault 的擴充

本 repo 根目錄 `AGENTS.md` 加入上下文控制、來源衝突、公開安全與失敗透明度等規則。這些是維護者的延伸，不應全部歸因於 Andrej Karpathy。

## 使用方式

- 跨平台核心：放在專案根目錄 `AGENTS.md`
- Claude Code 特有內容：另放薄版 `CLAUDE.md`
- Cursor：轉成 `.cursor/rules/*.mdc`
- 不要同時維護數份完整副本；平台檔只補差異

## 來源

- [doggy8088/andrej-karpathy-skills](https://github.com/doggy8088/andrej-karpathy-skills)
- [AGENTS.md 開放格式](https://agents.md/)
