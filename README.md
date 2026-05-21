# 個人作品集網站

> 蘇品寧 (Zoe Su) — 後端 / 全端 / 雲端工程師作品集

純靜態網頁,適合部署到 AWS S3 + CloudFront + Route 53。

## 專案結構

```
portfolio/
├── index.html      # 整個網站(單頁,含 inline CSS)
├── assets/         # 圖片放這裡
│   ├── crm.png
│   ├── ev.png
│   ├── hotel.png
│   └── (其他截圖)
└── README.md       # 你正在看
```

## 本機預覽

不需要任何工具,直接用瀏覽器打開 `index.html` 就好。

或用 Python 起一個簡單 server(看起來更接近正式部署):

```bash
cd portfolio
python -m http.server 8000
# 瀏覽器開 http://localhost:8000
```

## 你需要做的事(部署前)

### 1. 換掉專案截圖

`assets/` 目錄裡放你的截圖,然後在 `index.html` 裡把 placeholder 換成 `<img>`:

```html
<!-- 找到這段 -->
<div class="project-visual">
  <div class="placeholder">CRM 客戶管理後台<br>(截圖待補)</div>
</div>

<!-- 換成 -->
<div class="project-visual">
  <img src="./assets/crm.png" alt="CRM 客戶管理後台截圖">
</div>
```

四個專案都要換。建議截圖規格:
- 比例:4 : 3 (其他比例也行,會自動 cover)
- 解析度:至少 1200x900,以免 retina 螢幕看起來模糊
- 格式:PNG 或 WebP(WebP 檔案小很多)

### 2. 補上 GitHub / LinkedIn 連結

`index.html` 裡所有 `href="#"` 都要換成真實網址:

- 每個專案的 GitHub 連結
- CRM 的 Live Demo (Render 網址)
- 最下面 Contact 區塊的 GitHub、LinkedIn

### 3. (可選) 換掉 favicon

`<head>` 區塊可以加:

```html
<link rel="icon" type="image/svg+xml" href="./assets/favicon.svg">
```

---

## 部署到 AWS

### Step 1 · 註冊網域

到 Route 53 或第三方註冊商買網域。Route 53 比較貴一點但整合最方便。

推薦網域候選:
- `zoesu.dev` — 簡短專業,.dev 是 Google 管理的 TLD,所有 .dev 強制 HTTPS
- `zoesu.tw` — 在地化
- `pinningsu.com` — 全名

預算:**約 NT$300–700 / 年**(視 TLD 而定)

### Step 2 · S3 Bucket 建立

1. AWS Console → S3 → Create bucket
2. Bucket name:**用你的網域名**,例如 `zoesu.dev`(這很重要,後面靜態網站 host 要對應)
3. Region:選 `ap-northeast-1`(東京,離台灣最近)
4. **取消勾選** "Block all public access"(因為要公開)
5. 建立後進入 bucket → Properties → Static website hosting → Enable
   - Index document: `index.html`
6. Permissions → Bucket policy,貼上:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::zoesu.dev/*"
    }
  ]
}
```

(把 `zoesu.dev` 換成你的 bucket 名)

### Step 3 · 上傳檔案

```bash
# 安裝 AWS CLI 並 aws configure 後
cd portfolio
aws s3 sync . s3://zoesu.dev/ --exclude "README.md" --exclude ".*"
```

或直接在 Console 拖拉上傳 `index.html` 和 `assets/`。

### Step 4 · CloudFront CDN

1. CloudFront → Create distribution
2. Origin domain:**不要選 bucket 的 S3 API endpoint**,要選**靜態網站 endpoint**(類似 `zoesu.dev.s3-website-ap-northeast-1.amazonaws.com`)
3. Viewer protocol policy: **Redirect HTTP to HTTPS**
4. Alternate domain names (CNAMEs):填你的網域 `zoesu.dev`、`www.zoesu.dev`
5. SSL certificate:Request certificate in ACM(必須在 `us-east-1` 區域申請給 CloudFront 用)
6. Default root object:`index.html`

### Step 5 · Route 53 設定

1. Route 53 → Hosted zones → 你的網域
2. 建立兩筆 A record(都用 Alias):
   - `zoesu.dev` → 你的 CloudFront distribution
   - `www.zoesu.dev` → 你的 CloudFront distribution

DNS 傳遞需要時間,等個 10–30 分鐘。

### Step 6 · 驗證

瀏覽器打開 `https://你的網域`,看到網站就成功。

可順便檢查:
- `http://你的網域` 會自動轉 HTTPS ✓
- `www.你的網域` 也通 ✓
- DevTools 的 Network 看到 `X-Cache: Hit from cloudfront` ✓

---

## 費用估算

| 項目 | 月費 | 備註 |
|---|---|---|
| Route 53 hosted zone | ~NT$15 | 一定要付,沒有 free tier |
| 網域 | ~NT$30 | 平均下來,看 TLD |
| S3 儲存 + 流量 | ~NT$0 | 個人作品集流量極低,在 free tier 內 |
| CloudFront 流量 | ~NT$0 | 每月 1TB 免費(個人網站根本用不到) |
| ACM SSL 憑證 | NT$0 | CloudFront 用永遠免費 |

**第一年(含網域):約 NT$700 / 年(NT$60 / 月)**

---

## 之後更新內容

改完 `index.html` 或圖片之後:

```bash
# 重新同步
aws s3 sync . s3://zoesu.dev/ --exclude "README.md" --exclude ".*" --delete

# 清 CloudFront 快取(讓改動立即生效)
aws cloudfront create-invalidation \
  --distribution-id YOUR_DISTRIBUTION_ID \
  --paths "/*"
```

每月 1000 次 invalidation 免費。

---

## 設計說明(面試可講)

如果面試官問起這個網站,可以提到:

- **單檔架構**:整個網站就是一個 HTML 檔,CSS inline。沒有 build 流程、沒有 framework,純靜態。優點是檔案數量少、cache 友善、零依賴。
- **字體選擇**:Fraunces (serif display) + Manrope (sans body) + JetBrains Mono (accent),三種字體分工明確。中文用 Noto Serif TC 配對。
- **配色**:暖米白 (`#FAF6F0`) + 深酒紅 (`#8B2635`),避開科技業常見的「深藍/紫漸層」套版感。
- **效能**:沒有 JS framework、沒有第三方追蹤、沒有大圖。在 CloudFront 加持下首屏載入 < 0.5 秒。
- **CDN 策略**:CloudFront 全球邊緣節點,離使用者最近的節點服務內容。日本/台灣使用者會打到東京節點。
