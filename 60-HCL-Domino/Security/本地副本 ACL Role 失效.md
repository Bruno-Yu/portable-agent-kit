---
title: 本地副本 ACL Role 失效
tags:
  - hcl-domino/security
  - nrpc
status: public-review
---

# 本地副本 ACL Role 失效

本機開啟 NSF 副本時不經 NRPC，因此不會由伺服器端 NRPC 握手建立相同的遠端身分上下文。若設計依賴伺服器 ACL role 或遠端身分判斷，本機副本的結果可能不同。

## 檢查方向

1. 確認測試是在本機 NSF 還是伺服器 NSF。
2. 比對 ACL、角色、使用者名稱解析與程式執行身分。
3. 不要把本機測試結果直接推論成伺服器授權行為。

相關：[[NRPC 認證]]
