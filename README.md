# 吉伊卡哇收藏紀錄（第一版）

單頁靜態網站（GitHub Pages）＋ Firebase。你一個人可編輯，其他人拿到連結只能檢視。

## 檔案
| 檔案 | 用途 |
|---|---|
| `index.html` | 網站本體 |
| `config.js` | **唯一要改的設定檔**（你的 Gmail、Firebase 金鑰） |
| `firestore.rules` | 資料庫規則（只有你能寫，所有人能讀） |
| `sample-products.csv` | 17 筆範例商品，也是匯入格式的範本 |

## 先試看看（不用任何設定）
config.js 留空就是「示範模式」：載入範例商品，勾選只存在你的瀏覽器。
本機預覽請用 `python3 -m http.server` 之類開啟（直接點檔案開會因為模組限制而失敗）。

## 正式上線步驟
1. **Firebase**：可沿用你創作者代碼網站的專案。Authentication → Sign-in method 啟用 Google；
   Authentication → Settings → Authorized domains 加入 `你的帳號.github.io`。
2. **規則**：Firestore → Rules。把 `firestore.rules` 裡的 `isOwner()` 函式和兩個 `match` 區塊
   **加進**既有規則（不要整份覆蓋，會弄壞創作者代碼網站），email 換成你的 Gmail，發布。
3. **config.js**：填入 `ownerEmail` 與 Firebase 網頁應用程式設定（apiKey 等）。
   apiKey 本來就是公開的，真正的保護是第 2 步的規則。
4. 放進 GitHub 新 repo → Settings → Pages 開啟。把網址給別人即可。
5. 用你的 Gmail 開網站，點「擁有者登入」，就會出現編輯模式與「擁有者工具」。

## 資料結構
- `chiikawa_products/{JAN}`：商品（name、character、series、type、url、image、batch、stock）
- `chiikawa_meta/purchased`：單一文件 `{ jans: { "JAN": true/false } }`，所有勾選都在這裡
  → 別人打開網站只要讀 1 份勾選＋商品清單；本機有快取，不會每次重讀全部商品。

## 匯入商品（CSV 或從 Google 試算表複製貼上）
欄位名稱（第一列）：`jan, name, character, series, type, url, image, batch, stock`
- 也認中文欄名：JAN碼／名稱／角色／系列／類型／官網連結／圖片網址／批次／庫存
- 只有 `jan` 和 `name` 必填；`jan` 缺但 `url` 是 `/products/條碼` 格式時會自動抓條碼
- 多個角色或系列用 `|` 或 `、` 分隔，篩選時會各自歸類
- `batch` 建議用官網的集合名稱，如 `re20261119`（補貨）、`20261127`（發售），會依其中的日期排序
- `stock`：有庫存／售罄（或 1／0），空白代表不更新
- 空白欄位**不會**覆蓋已有的資料；**勾選紀錄永遠不會被匯入動到**
- 匯入前會先顯示「新增幾筆、更新幾筆、略過幾筆」，確認後才寫入

## 目前不做／已知限制
- 沒有刪除商品的按鈕（官網下架的商品留著無妨；真要刪可在 Firebase 主控台刪）
- 圖片只引用官網網址，不另存；官網改檔名時圖片會失效，顯示「圖片無法載入」，重新匯入新網址即可
- 網站有 `noindex`，不會被搜尋引擎收錄，但拿到連結的人都看得到
- 書籤小工具（從官網頁面擷取商品成 CSV）留待下一階段
