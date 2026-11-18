# 罐罐支持：跨專案規格書

狀態：草稿 v0.2（2026-11-18）
擁有者：tommy771004
適用專案（6 個）：Taiwanrail、TransitRail、PikminTool、TW_veggieprice-、Productivity、-TWReal_Estate_Price_Explorer_google

---

## 1. 目的

讓使用者在每個專案的側邊選單看到「罐罐支持」，點了之後到同一個靜態支持頁，再從那裡用台灣可用的支付平台（第一階段為綠界）小額支持開發者。

設計原則：

- **只有一個支持頁**。所有支付連結只存在支持頁，各專案只放一個入口。換平台、加平台、調金額，都只改支持頁，不必重新部署 6 個專案。
- **不打擾**。入口只出現在選單裡，不主動彈出、不加紅點或徽章、不在操作流程中插入提示。
- **不追蹤**。支持頁不載入任何第三方腳本，不設 cookie。

## 2. 架構

```
各專案側邊選單「罐罐支持」
  └─(新分頁)→ SUPPORT_URL?from=<專案代號>&lang=<zh|en>
                 └─ 支付平台連結（綠界收款連結…）
```

### 2.1 SUPPORT_URL（唯一常數）

```
SUPPORT_URL = https://tommy771004.github.io/support/
```

各專案都只引用這個常數一次（放在專案自己的設定檔或常數檔），不得在多處重複寫死。

### 2.2 專案代號（`from` 參數）

| 專案 | from |
|---|---|
| Taiwanrail | `taiwanrail` |
| TransitRail | `transitrail` |
| PikminTool | `pikmintool` |
| TW_veggieprice- | `veggieprice` |
| Productivity | `productivity` |
| -TWReal_Estate_Price_Explorer_google | `realestate` |

`from` 的用途只有兩個：支持頁顯示「感謝你從〇〇來」的那一句，以及日後有需要時供支付平台備註欄參考。**它不是統計工具**。GitHub Pages 沒有存取紀錄可看，支持頁也不放分析腳本，所以「哪個專案帶來多少點擊」要靠來源專案自己既有的分析工具記錄（§3.6）。

## 3. 側邊選單入口規格（6 個專案共用）

### 3.1 文字

| | 中文 | English（提案） |
|---|---|---|
| 標題 | 罐罐支持 | Buy the dev a treat |
| 說明（灰字） | 給開發者一點鼓勵 | A little encouragement for the developer |

- 依專案目前的介面語言顯示；沒有英文介面的專案只用中文。
- 說明使用該專案「次要文字」的既有色彩 token（例如 Taiwanrail 的 `text-slate-500`），不得自訂新灰色。
- 文案不得出現「斗內」「贊助我們」等與上表不同的字樣，6 個專案用字一致。

### 3.2 圖示

- 使用支持頁 repo 提供的共用圖示 `assets/can.svg`（貓罐頭的線稿，24×24、`stroke="currentColor"`、圓角線端），6 個專案外觀一致。
- 線寬改成該專案現有圖示的線寬（lucide 預設 2；Material Symbols 專案則對齊其 weight）。
- 圖示直接放在列上，不加底色方塊或圓形底。
- 共用圖示完成前，暫用各專案圖示庫中最接近的圖示（lucide `Cat`；Material Symbols `pets`）。

### 3.3 位置

- 放在側邊選單的「應用資訊」區塊：在「意見回饋」之後、「功能更新記錄／關於」之前。沒有這些項目的專案，就放在選單最末段、版本或版權資訊之前。
- 獨立一列，**不放進「更多工具服務」清單**。那份清單是開發者的其他網站，「罐罐支持」是支持開發者本人，性質不同。
- 列的高度、內距、點擊範圍沿用該選單既有的列樣式（觸控目標至少 44px）。

### 3.4 連結行為

```html
<a href="{SUPPORT_URL}?from={專案代號}&lang={zh|en}"
   target="_blank" rel="noopener noreferrer">
```

- 新分頁開啟，不離開目前頁面。
- 不加 `sponsored`／`nofollow`：這是開發者自己的頁面，不是付費版位。
- 選單項目右側可放該選單現有的「外部連結」箭頭；沒有慣例的專案不加。
- 點擊後選單是否關閉，依該專案其他外部連結的既有行為。

### 3.5 Electron（Productivity）

Productivity 的主程序已用 `setWindowOpenHandler` 把 `window.open` 和 `target="_blank"` 交給 `shell.openExternal`（`electron/main.ts:438`），所以 §3.4 的連結在桌面版會用系統瀏覽器開啟。實作後需在桌面版實測一次，確認不是在 app 視窗內開啟。

### 3.6 點擊記錄（選用）

- 有分析工具的專案（GA4 等），點擊時送一個事件：名稱 `support_click`，參數 `{ from: <專案代號> }`。
- 沒有分析工具的專案不記錄，**不得**為此新增追蹤。
- 不送任何個人資料。

### 3.7 不做的事

- 不在首頁、結果頁、通知或任何彈窗中提示「罐罐支持」。
- 不加紅點、徽章、閃爍或計數。
- 不在專案內嵌入支付按鈕、支付 SDK 或 iframe。
- 不需要修改 CSP：在新分頁開啟連結不受 `connect-src` 等規則限制。

### 3.8 功能更新記錄

有使用者可見更新記錄的專案（Taiwanrail `src/lib/changelog.ts`、TransitRail `src/data/changelog.ts`），同一個 commit 加一筆：

- zh：側邊選單新增「罐罐支持」，可以請開發者吃個罐罐。
- en：Added "Buy the dev a treat" to the side menu.

## 4. 支持頁規格

### 4.1 Repo 與發佈

- 獨立的 **public** repo：`tommy771004/support`（GitHub 免費方案只能從 public repo 發佈 Pages）。公開 repo 的防護見 §9。
- 目錄：

  ```
  SPEC.md           ← 本規格書（不發佈）
  site/             ← 只有這個資料夾會發佈
    index.html
    payments.js     ← 支付連結的唯一來源
    assets/can.svg  ← 共用圖示（各專案複製這份）
  .github/workflows/pages.yml
  ```

- 用 GitHub Actions 只部署 `site/`，不可以用「從分支根目錄發佈」，否則 SPEC.md 也會被公開。
- 這個 repo 是公開的：不得放入任何密鑰、帳號資訊、支付平台後台資料或個人資料。收款連結本身就是公開網址，可以放。

### 4.2 技術限制

- 單一 HTML 檔加一個資料檔，不使用框架、不需要建置步驟。
- 不載入第三方腳本、字型 CDN 或追蹤碼（安全要求見 §9.2）。
- 不設 cookie，不使用 localStorage。
- 首頁小於 50 KB（不含 QR code 圖片）。
- 不需要搜尋引擎收錄：加 `<meta name="robots" content="noindex">`。

### 4.3 內容

1. 一句感謝。有 `from` 時加上專案名稱（例如「謝謝你從台鐵高鐵時刻查詢過來」）；`from` 不在 §2.2 名單內時忽略。
2. 支付方式列表，資料全部來自 `payments.js`。每一項包含：平台名稱、可用的付款方式（信用卡、ATM、超商…）、金額說明、前往付款的按鈕。桌面版另外顯示 QR code，方便用手機付款。
3. **金額：自訂金額，最低 NT$30（約 US$1）**。金額由付款人在支付平台頁面輸入，最低金額由支付平台的收款連結設定強制執行；支持頁只顯示「自訂金額，最低 NT$30」，不自行收集或驗證金額。英文頁面寫「Any amount from NT$30 (about US$1)」。
4. 說明文字：
   - 這是自願支持，不會換得任何商品或服務，也不影響任何功能。
   - 付款在支付平台完成，本頁不收集任何個人資料。
   - 退款與付款問題的聯絡方式。
5. 中英文：`lang=en` 時顯示英文，否則中文；頁面上提供語言切換。

### 4.4 `payments.js` 格式

```js
export const PAYMENTS = [
  {
    id: 'ecpay',
    name: { zh: '綠界科技', en: 'ECPay' },
    methods: { zh: '信用卡・ATM・超商', en: 'Card, ATM, convenience store' },
    url: 'https://…',   // 綠界收款連結（付款人自訂金額）
    minAmount: 30,        // 只用於顯示；實際下限由綠界收款連結設定
    enabled: true,
  },
];
```

- `enabled: false` 就是下架方式，頁面不顯示該項，不必刪除資料。
- 網址必須是 `https://` 且主機在 §9.2 的允許清單內；不符合時頁面略過該項。

### 4.5 視覺

- 支持頁有自己的視覺設計，不套用任何一個專案的品牌色。
- 實作前依 `anti-slop-design` 準則決定設計方向，並用它逐項檢查成品。
- 手機優先：主要操作在單手拇指範圍內，不需要橫向捲動。

## 5. 支付平台

### 5.1 第一階段：綠界科技（ECPay）

選擇理由：個人會員就能收款；PayPal 不支援台灣付款給台灣收款。

**以下都須以綠界官網的現行說明為準，上線前逐項確認並把結果填回本節：**

- [ ] 個人會員的申請資格、審核流程與時間
- [ ] 「收款連結」能否讓付款人自訂金額，並設定最低 NT$30
  - 若不支援自訂金額：改為一個固定 NT$30 的連結（「一罐」），並在本節記錄，由使用者決定是否加其他固定金額
- [ ] 各付款方式的手續費與撥款週期
- [ ] 收款頁可以帶入的備註或參數（能否傳入 `from`）
- [ ] 是否須開立發票或收據；個人收到支持款的所得申報方式

### 5.2 候選平台（待查證，未決定）

| 平台 | 待確認 |
|---|---|
| Portaly | 是否支援台灣個人收款、抽成、提領方式 |
| Oen 應援 | 個人開發者是否適用、抽成 |
| 歐付寶 | 個人收款連結、手續費 |

加入新平台只需要在 `payments.js` 加一筆，各專案不用改。

## 6. 各專案套用清單

| 專案 | 技術 | 入口位置 | 圖示庫 | 備註 |
|---|---|---|---|---|
| Taiwanrail | React 19 + Vite | 側邊選單元件 | lucide | 放在「意見回饋」與「功能更新記錄」之間；加 changelog 一筆 |
| TransitRail | React + Vite | 選單寫在主元件內 | lucide | 加 changelog 一筆 |
| PikminTool | React + Vite | 側邊選單元件 | lucide | 「其他工具」清單之後，「關於」連結之前 |
| TW_veggieprice- | Next.js 15 | 側邊選單元件（2026-11 新增） | Material Symbols | 放在「意見回饋」那一段、「意見回饋」下方；點擊事件用專案既有的 `trackEvent` |
| Productivity | React + Vite + Electron | 工作區側邊欄，「系統設定」按鈕附近 | lucide | 依 §3.5 在桌面版實測 |
| -TWReal_Estate_Price_Explorer_google | React | 側邊選單元件（**未驗證**：本機沒有這個 repo） | lucide | 套用前先 clone 確認選單結構 |

實際檔案位置在套用時於各專案內確認；本 repo 是公開的，不記錄私有專案的內部結構。

### 6.1 每個專案的驗收

- [ ] 選單中出現「罐罐支持」與灰字說明，位置符合 §3.3
- [ ] 中英文介面文字都正確（有英文介面的專案）
- [ ] 點擊後在新分頁開啟 `SUPPORT_URL?from=<代號>&lang=<語言>`，原頁面不變
- [ ] 鍵盤可以聚焦並開啟，螢幕閱讀器念出標題與說明
- [ ] 深色模式下文字對比足夠
- [ ] 沒有新增任何彈窗、徽章或首頁提示
- [ ] 有分析工具的專案能看到 `support_click` 事件
- [ ] 專案既有的檢查（lint、typecheck、測試）通過

## 7. 決定紀錄與待決定事項

已決定（2026-11-18）：

- 託管：本 repo 改為 public，用 GitHub Pages 發佈 `site/`。原本的 private repo 無法啟用 Pages（GitHub 回 422「Your current plan does not support GitHub Pages for this repository」）。
- TW_veggieprice- 已有側邊選單，依 §6 放置。
- 金額：自訂金額，最低 NT$30（約 US$1）。

待決定：

1. **英文用字**：§3.1 的英文是提案。
2. **共用圖示**：`assets/can.svg` 的最終樣式。

## 8. 實作順序

1. 完成 §5.1 綠界查證，取得收款連結
2. 建立支持頁 `site/` 與部署 workflow，取得最終 SUPPORT_URL
3. 以 Taiwanrail 為第一個套用的專案，驗收通過
4. 依 §6 套用其餘 5 個專案（每個專案各自一個 commit 或 PR）

## 9. 資訊安全

### 9.1 Repo 防護（GitHub 設定）

| 項目 | 設定 |
|---|---|
| Secret scanning | 開啟 |
| Push protection | 開啟：推送含密鑰的 commit 會被擋下 |
| Dependabot alerts／安全更新 | 開啟；`.github/dependabot.yml` 每週檢查 GitHub Actions 版本 |
| 私下回報漏洞 | 開啟（Private vulnerability reporting），並提供 `SECURITY.md` |
| `main` 分支 ruleset | 禁止刪除、禁止 force push |
| Actions 權限 | 只允許 GitHub 官方 actions；`GITHUB_TOKEN` 預設唯讀；Actions 不可核准 PR |
| 外部貢獻者的 workflow | 所有外部貢獻者的 PR 都要手動核准才會跑 |
| Pages 部署環境 | `github-pages` 環境只接受 `main` 分支部署 |
| Wiki／Projects | 關閉 |
| CODEOWNERS | 全部檔案由 `@tommy771004` 負責 |

### 9.2 支持頁本身

GitHub Pages 不能自訂回應標頭，所以以 `<meta>` 設定：

- **CSP**：`default-src 'none'; script-src 'self'; style-src 'self'; img-src 'self' data:; base-uri 'none'; form-action 'none'`。不使用 inline script 或 inline style。
- **Referrer**：`<meta name="referrer" content="no-referrer">`，不把來源網址帶到支付平台。
- **防止被嵌入**：GitHub Pages 無法送 `X-Frame-Options`，`frame-ancestors` 在 meta 中無效。頁面上沒有任何可被誤點的敏感操作（付款都在支付平台完成），因此可接受；若頁面用 JS，加上 `if (window.top !== window.self)` 時改以新視窗開啟連結。
- **參數處理**：`from` 與 `lang` 只拿來比對 §2.2 的固定名單與 `zh`／`en`，**不得**把網址參數原樣放進 DOM（不用 `innerHTML`，只用對照表裡的文字）。
- **支付連結允許清單**：`payments.js` 的網址主機必須在清單內，目前為綠界的收款網域（上線時依實際連結填入），其他一律略過。
- **外部連結**：一律 `rel="noopener noreferrer"`。
- **Workflow**：只部署 `site/`；使用的 action 固定到 commit SHA；權限只給 `contents: read`、`pages: write`、`id-token: write`。
