---
title: NRPC 認證
tags:
  - hcl-domino/security
  - notes
  - authentication
aliases:
  - Notes Remote Procedure Call
created: 2026-08-03
status: public-review
---

# NRPC 認證

> [!abstract] 一句話 NRPC（Notes Remote Procedure Call）是 Notes/Domino 的私有協定（TCP **1352**）。「NRPC 認證」是連線建立時，雙方用 **ID 檔中的憑證與私鑰**互相證明身分的握手過程。它是 Domino 一切授權判斷的**身分來源**。

## 1. 適用範圍

|情境|走 NRPC|
|---|:-:|
|Notes / Designer / Admin Client 連伺服器|✅|
|Server ↔ Server 複寫、郵件路由|✅|
|開**本機** NSF|❌ 直接 file I/O|
|瀏覽器連 Domino HTTP|❌ 走 HTTP + Internet Password / SSO|

→ 本機無 NRPC，即無身分主體，故 [[本地副本 ACL Role 失效]]。

## 2. 認證的兩個階段


```
Notes Client ──── TCP 1352 ────> Domino Server
   user.id                          server.id
      └──── 互相驗證身分 ────────────┘
              ↓
   Server 得到「CN=Alice Chen/OU=IT/O=ExampleCorp」這個確定的身分
              ↓
   才有辦法去查 ACL、算 Roles
```

### 2.1 Validation — 我憑什麼相信你的公鑰

階層式憑證信任，三條規則：

1. 信任自己 ID 檔中任何憑證的**發行者（certifier）公鑰**
2. 信任該 certifier 簽發的所有憑證
3. 信任被信任 certifier 的**下層**

做法：雙方交換憑證鏈，往上找**共同祖先**。

```
Alice :  /O=ExampleCorp ← /OU=IT  ← CN=Alice Chen
Server:  /O=ExampleCorp ← /OU=Srv ← CN=Domino-App01
共同祖先 = /O=ExampleCorp → 信任成立
```

> [!warning] 跨組織 不同組織樹（/O=ExampleCorp vs /O=PartnerCorp）無共同祖先 → 必須建立 **Cross Certificate**。

### 2.2 Authentication — 你是否真的持有私鑰

**雙向** Challenge / Response：

```
Server → Client : 亂數 R1
Client → Server : 以私鑰運算 R1
Server          : 用已驗證的 Client 公鑰驗算 ✅

Client → Server : 亂數 R2
Server → Client : 以 server.id 私鑰運算 R2
Client          : 驗算 ✅（確認非假伺服器）
```

> [!important] 兩個關鍵事實
>
> - **私鑰不離開 ID 檔，密碼從不傳輸**。密碼只用於本機解密 ID 檔。
> - **原生雙向**，天生防中間人假冒伺服器，不依賴外部 CA。

## 3. 完整握手序列

```
1. TCP connect :1352
2. 協定版本 / 能力協商
3. 交換憑證 → Validation
4. 雙向 Challenge/Response → Authentication
5. 協商 session key（Port Encryption 開啟則後續全程加密）
6. 建立 session context：
     解析階層名稱 CN=Alice Chen/OU=IT/O=ExampleCorp
     讀取 Alternate Name
     展開 Name Variants（*/OU=IT/O=ExampleCorp、*/O=ExampleCorp …）
     查 Domino Directory 展開巢狀群組
     → 產生完整 name list
7. 檢查 Server Document 的 Server Access（Access / Deny list）
8. 開 NSF 時以 name list 比對 ACL → access level + Roles
```

> [!note] 步驟 6–8 是關鍵 這串就是 `@UserRoles` 的產生來源。缺 1–5，就沒有 6，也就沒有 8。

## 4. 密碼學細節

|項目|說明|
|---|---|
|憑證格式|Notes 私有格式，**非 X.509**（X.509 另用於 S/MIME 與 TLS）|
|非對稱演算法|RSA，長度依 certifier 建立時而定（舊 512/630 bit，建議 2048）|
|Session 加密|對稱式，現代為 AES（早期 RC4/RC2）|
|密碼作用|僅本機解鎖 ID 檔|
|ID 檔內容|私鑰、憑證鏈、secret keys、Internet cert|

> [!caution] 舊環境陷阱 從舊版升上來的 Domino，certifier 可能仍是短金鑰。升級需 **certifier key rollover**，影響所有下層 ID，屬大工程。「R12 但金鑰強度停在十幾年前」很常見。

## 5. R12 實務要點

- **ID Vault**：ID 集中託管於伺服器 vault NSF；認證流程不變，僅取得方式改為首次登入下載。使用者量大時值得導入。
- **Port Encryption**：`Server Document → Ports → Notes Network Ports → Encrypt network data`。**預設可能為關**——握手安全但**後續文件內容為明文**。跨網段務必開啟。
- **密碼品質 / CA Process**：透過 Security Settings Policy 下發。
- **Anonymous**：Server Document 若允許，則跳過 Authentication，身分為 `Anonymous`，僅能命中 ACL 的 `Anonymous` 條目。

## 6. NRPC vs HTTPS/TLS

|面向|NRPC|HTTPS (TLS)|
|---|---|---|
|信任錨|組織自有 certifier（階層式）|公開 / 企業 CA|
|憑證格式|Notes 私有|X.509|
|認證方向|**原生雙向**|預設僅驗伺服器（mTLS 才雙向）|
|使用者身分|協定層直接取得階層名稱|應用層另行處理|
|加密|需啟用 Port Encryption|預設即加密|

## 7. 開發者結論

1. `session.EffectiveUserName` 的可信度**來自 NRPC 認證**，是密碼學保證，強於一般 Web session cookie。
2. **離開 NRPC，保證即消失**。將 Notes 資料以 REST/Web API 曝露時，必須自建認證（JWT / Entra ID / mTLS），不可假設 Domino 會擋。
3. **本機副本完全繞過此機制** → 安全性不可建立在 ACL Role 上。

---

Related: [[本地副本 ACL Role 失效]]
