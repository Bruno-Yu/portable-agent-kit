---
title: Publication Checklist
tags:
  - security
  - publishing
status: active
---

# Publication Checklist

## 每批發布前

- [ ] 公司、客戶、同事、專案與環境名稱已移除或改成假資料
- [ ] Token、Cookie、Authorization header、帳密、私鑰均不存在
- [ ] IP、hostname、內網 URL、資料庫名稱與本機使用者路徑已泛化
- [ ] 圖片已確認來源、授權、人物、瀏覽器分頁及畫面資料
- [ ] 第三方內容已改寫並附原始來源，沒有大段重製
- [ ] Wikilink 與 Markdown link 均能解析
- [ ] `gitleaks dir . --redact` 無發現
- [ ] 已閱讀完整 `git diff --cached`

## 第一次公開前

```bash
git status --short
gitleaks dir . --redact
git add .
git diff --cached --check
git diff --cached
```

> [!danger]
> 若秘密曾經進入 Git commit，只從目前檔案刪除仍不夠；應先撤銷／輪替秘密，再清理歷史。
