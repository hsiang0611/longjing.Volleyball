# 青商盃排球計分板 — 完整專案說明

一個**單一 HTML 檔**的排球賽事計分系統，免安裝、用瀏覽器開啟即用，並可選擇接上 Firebase 做跨裝置即時同步。

---

## 檔案清單

| 檔案 | 用途 | 是否必要 |
|------|------|----------|
| **青商盃排球計分板.html** | 主程式（完整功能版，已內嵌 JCI 標誌、已填入你的 Firebase 設定） | ★ 主檔 |
| database.rules.json | Firebase Realtime Database 安全規則（觀眾唯讀、計分台登入才能寫） | 上線正式用時 |
| Firebase規則設定說明.md | 規則與計分台登入的設定步驟 | 參考 |
| 排球計分板_簡易版.html | 最早的精簡版（無 Firebase、無對戰樹），保留備查 | 選用 |

> 平常只需要「青商盃排球計分板.html」這一個檔案。

---

## 功能總覽

- **16 隊 4 組單循環**自動賽程；五局三勝（1–4 局 25 分、決勝局 15 分，淨勝 2 分），自動換局與勝負判定。
- **四個分頁**：計分板（即時比數＋賽程）、積分榜、對戰樹、後台計分。
- **小組積分榜**：依勝場→局數差→得分差排名，前兩名標示「晉級」。
- **淘汰賽對戰樹**：四組前兩名進八強，交叉編排，勝隊自動晉級，SVG 連接線。
- **後台計分**：＋/− 加減分、開始/結束/重設、隊名即時編輯、狀態篩選。
- **復原（↶）**：可連續退回上一步（含跨局、誤觸重設），最多 60 步。
- **分數翻牌特效**：上下分離翻牌，只在數字真正變動時觸發。
- **大螢幕模式**：投影專用放大比分頁（右上「🔲 大螢幕」）。
- **品牌客製**：JCI 標誌已內嵌；賽事名稱、副標、版權人皆可點擊編輯；可另上傳分會會徽。
- **資料儲存**：localStorage 自動保存（重開不遺失）＋ JSON 匯出/匯入備份。
- **Firebase 即時同步**（選用）：多裝置同步同一份比分，狀態標籤顯示連線狀況。
- **唯讀觀眾模式**：網址加 `?view`，隱藏所有可改動控制項，前端＋資料庫雙層唯讀。
- **計分台登入**：搭配安全規則，只有登入帳號能改分。
- **重設密碼保護**：重設類動作需輸入密碼（預設 5859，提示「子墉車牌」），防誤觸。
- **版權頁尾**：© 年份自動帶當年。

---

## 三種使用層級

### A. 單機（最簡單，免設定）
直接用瀏覽器打開主檔即可計分，資料存在本機瀏覽器。要備份或換電腦用「⬇ 匯出 JSON / ⬆ 匯入 JSON」。
同一台電腦多個視窗（例如後台＋投影）會自動同步。

### B. 雲端即時同步（多裝置）
本檔已填入你的 Firebase 設定，上傳到網站打開後右上角會顯示「雲端已同步」。
不同裝置開同一網址即同步；用 `?room=賽事代號` 區分不同賽事/場地。
> ⚠️ 別讓資料庫長期停在「測試模式」（開放寫入）。請完成下方 C 的規則設定。

### C. 鎖定觀眾寫入（正式賽會建議）
1. **Authentication** → Sign-in method → 啟用「電子郵件/密碼」→ Users → 新增計分台帳號（Email＋密碼）。
2. 把該 Email 填進 `database.rules.json` 的 `.write` 那行 → 到 Realtime Database「規則」頁發布。
3. **Authentication → Settings → Authorized domains** 確認有 `hsiang0611.github.io`。
4. 計分台用一般網址打開 → 按「🔑 計分台登入」登入後才能改分。
5. 觀眾用「🔗 觀眾連結」按鈕產生的網址（帶 `?view`）只能看、不能改。

詳見 `Firebase規則設定說明.md`。

---

## 你的專案資訊（已套用）

- 網址：`https://hsiang0611.github.io/longjing.Volleyball/`
- Firebase 專案：`volleyballscoreboard-a497a`（Realtime Database 區域：asia-southeast1）
- 建議的 API key 參照網址限制（Google Cloud Console → 憑證）：
  ```
  https://hsiang0611.github.io/longjing.Volleyball/*
  https://hsiang0611.github.io/*
  http://localhost:*/*
  ```

---

## 常用網址

- 計分台：`https://hsiang0611.github.io/longjing.Volleyball/?room=longjing2026`
- 觀眾（唯讀）：`https://hsiang0611.github.io/longjing.Volleyball/?room=longjing2026&view`
- 大螢幕：開啟後按右上「🔲 大螢幕」（會以 `#board` 開新分頁）

---

## 重要觀念

- Firebase 的 `apiKey` 等設定**本來就是公開的**，放在 GitHub 沒問題；安全靠的是「資料庫規則＋計分台登入＋API key 網域限制」。
- 重設密碼（5859）只是前端防誤觸，不是權限控管；真正權限由資料庫規則決定。
- 唯一不可公開的是 Firebase Admin SDK 的 service account JSON——本專案沒用到。
