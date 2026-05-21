# 個人作品集網站 — AWS 雲端部署

> 蘇品寧 Zoe Su 的個人作品集,部署於 AWS 雲端服務。
> 🔗 線上瀏覽:**[https://www.zoesu.dev](https://www.zoesu.dev)**

一個純靜態的單頁式個人作品集網站,從零完成「**建置 → 雲端部署 → 自訂網域 → HTTPS 加密**」的完整流程,展現靜態網站托管於 AWS 的實務能力。

---

## 🏗️ 部署架構

```
使用者瀏覽器
   │  HTTPS
   ▼
Route / 自訂網域 (zoesu.dev)
   │  DNS 解析
   ▼
CloudFront (CDN)  ←── ACM (SSL 憑證, us-east-1)
   │  HTTP (內網)
   ▼
S3 Bucket (Static Website Hosting, ap-northeast-1)
   └── index.html + favicon + assets/
```

| 層級 | 服務 | 用途 |
|---|---|---|
| 儲存 | **AWS S3** | 靜態網站托管,設定 Bucket Policy 控制公開讀取 |
| CDN | **AWS CloudFront** | 全球邊緣節點加速,降低延遲 |
| 憑證 | **AWS Certificate Manager (ACM)** | 簽發 SSL 憑證,啟用 HTTPS |
| DNS | **自訂網域** | 設定 CNAME / 轉址指向 CloudFront |

---

## 🔧 部署流程

### 1. S3 靜態網站托管
- 建立 S3 Bucket(命名與網域一致)
- 啟用 Static Website Hosting,設定 `index.html` 為首頁
- 設定 Bucket Policy 開放公開讀取(`s3:GetObject`)

### 2. ACM SSL 憑證
- 於 **us-east-1**(CloudFront 指定區域)申請 public certificate
- 透過 DNS-01 驗證(在網域 DNS 加入 CNAME)證明網域擁有權
- 同時涵蓋 apex 與 www 子網域

### 3. CloudFront CDN
- 建立 distribution,Origin 指向 S3 Website Endpoint(HTTP only)
- Viewer Protocol Policy 設為 Redirect HTTP to HTTPS
- 綁定自訂網域(Alternate Domain Names)與 ACM 憑證
- 設定 Default Root Object 為 `index.html`

### 4. 自訂網域 DNS
- www 子網域:CNAME 指向 CloudFront distribution
- apex 主網域:轉址(301)至 www

### 5. 內容更新流程
```bash
# 更新檔案後
1. 上傳至 S3(覆蓋)
2. CloudFront 建立 Invalidation (/*) 清除快取
3. 等待邊緣節點同步,變更生效
```

---

## 📂 專案結構

```
.
├── index.html              # 整個網站(單檔,含 inline CSS)
├── favicon.svg             # 網站圖示(向量)
├── favicon.png             # 網站圖示(PNG fallback)
├── apple-touch-icon.png    # iOS 主畫面圖示
└── assets/                 # 專案截圖等圖片資源
```

---

## 🎨 技術特色

- **零依賴單檔架構**:整個網站是一個 HTML 檔,CSS inline,無 build 流程、無 framework,cache 友善、載入快速
- **響應式設計 (RWD)**:桌機與行動裝置皆適配
- **編輯雜誌風格**:暖米白 + 酒紅配色,serif + sans 字體分工
- **效能優化**:CloudFront 全球加速,首屏載入快速

---

## 🛠️ 使用技術

`AWS S3` `AWS CloudFront` `AWS Certificate Manager` `HTML5` `CSS3`

---

## 📝 開發者

**蘇品寧 Zoe Su** — 後端 / 全端 / 雲端工程師
- 🌐 [www.zoesu.dev](https://www.zoesu.dev)
- 💼 [LinkedIn](https://www.linkedin.com/in/%E5%93%81%E5%AF%A7-%E8%98%87-8326303b3/)
- 🐙 [GitHub](https://github.com/zoesu4522)
