# Realtime Database 安全規則 — 設定說明

這份規則讓**觀眾在資料庫層面也無法寫入**，與前端的唯讀模式（`?view`）搭配，就是真正鎖死的觀眾模式。

- 讀取：**所有人**都能讀（觀眾、投影、手機都能看即時比分）。
- 寫入：**只有登入的計分台帳號**能改比分。沒登入的人（含所有觀眾、以及任何拿到你網址的人）即使繞過前端、直接打資料庫也會被拒絕。
- 另外加了**結構驗證**：節點只能是 `{ data: 字串, updatedAt: 數字 }`，其他欄位一律擋掉，避免被塞垃圾資料。

---

## 設定步驟（約 5 分鐘）

1. **建立計分台帳號**
   Firebase 主控台 → **Authentication** → 「Sign-in method」分頁 → 啟用 **電子郵件/密碼 (Email/Password)** →
   回「Users」分頁 → **新增使用者**，輸入一個 Email 與密碼（例如 `scorer@your-jci.org`）。這就是計分台要登入的帳號。

2. **貼上規則**
   Firebase 主控台 → **Realtime Database** → 「規則 (Rules)」分頁 →
   貼上 `database.rules.json` 的內容 → 把規則裡的 `scorer@your-jci.org` 換成你上一步建立的 Email → 按「發布」。

3. **計分台登入**
   用一般網址打開計分板（**不要**帶 `?view`），右上角會出現「🔑 計分台登入」按鈕 →
   輸入剛才的帳號密碼。登入後該裝置就能改比分（Firebase 會記住登入狀態，這台之後免再登入）。

4. **發觀眾連結**
   用「🔗 觀眾連結」按鈕複製網址（帶 `?view`）分享出去。觀眾**不需登入**就能看，但完全不能改。

完成後：計分台（已登入）能改分並即時同步 → 觀眾與投影即時看到；任何未登入者都無法寫入資料庫。

---

## 多位計分員（選用）

若需要不只一位計分員，把規則裡的 `.write` 改成下面這行，改用「白名單」：

```
".write": "auth != null && root.child('writers').child(auth.uid).val() === true",
```

然後在資料庫手動建立 `writers` 節點，把每位計分員的 UID 設為 `true`（UID 可在 Authentication 的 Users 清單看到）：

```
writers/
  AbCd1234...uid1: true
  EfGh5678...uid2: true
```

並在規則最外層加上保護，避免有人改名單：

```
"writers": { ".read": false, ".write": false }
```

---

## 還沒準備好用驗證？先用「唯讀防呆 + 結構驗證」

若暫時還不想設定登入，至少別用全開的測試模式。可改用下面這版：保留前端唯讀防誤觸，並限制資料結構與大小（但此版任何人仍可寫入，僅擋格式錯誤）：

```json
{
  "rules": {
    "tournaments": {
      "$room": {
        ".read": true,
        ".write": true,
        ".validate": "newData.hasChildren(['data','updatedAt'])",
        "data":      { ".validate": "newData.isString() && newData.val().length <= 5000000" },
        "updatedAt": { ".validate": "newData.isNumber()" },
        "$other":    { ".validate": false }
      }
    },
    "$other": { ".read": false, ".write": false }
  }
}
```

> 注意：這版只擋「格式」，不擋「未授權寫入」。要真正鎖死觀眾寫入，請用上面的驗證版規則。

---

## 小提醒

- 規則檔允許 `//` 註解（Firebase 主控台與 CLI 都接受）；若你用嚴格 JSON 工具檢查，請先移除註解。
- 前端的重設密碼（5859）只是防誤觸；真正的權限是由這份資料庫規則 + 計分台登入決定的。
- 測試模式（開放讀寫）會在 30 天後自動到期；正式使用前請務必換成這份規則。
