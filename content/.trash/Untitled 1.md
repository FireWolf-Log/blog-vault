你目前處於規劃階段，我建議先不要決定使用 Astro、n8n 或寫哪種 Python 程式，而是先評估兩個彼此獨立的問題：

1. **社群平台 API：** 能不能免費讀取自己的貼文、發布新貼文？免費額度、權限申請、帳號類型限制是什麼？
    
2. **自動化執行環境：** 能不能免費、定期、自動執行？需不需要自己的電腦保持開機？
    

先講結論：

**若以零成本、長期維護、個人用途為目標，GitHub Actions + Python 是最值得優先評估的執行方案。真正的瓶頸通常不是自動化程式放哪裡，而是社群平台是否提供可用的免費 API。**

## **一、社群 API 免費可用性評估**

截至 2026 年 11 月 14 日，依可查證的官方文件整理。表中「免費」指一般自行開發使用 API 不必另購付費 API 方案，不代表免審核、無配額或所有功能可用。

| 平台／帳號類型                      | 讀取自己的貼文 | 發布貼文  | 免費使用評估          |
| ---------------------------- | ------- | ----- | --------------- |
| Facebook 個人檔案                | 受限      | x     | 不適合官方 API 全自動化  |
| Facebook 粉絲專頁                | o       | o     | 可免費使用，需權限       |
| Instagram 一般個人帳號             | x       | x     | 不支援目前的正式 API    |
| Instagram Creator / Business | o       | o     | 可免費使用，需權限       |
| Threads                      | o       | o     | 有免費官方 API，具配額限制 |
| Bluesky                      | o       | o     | 免費且限制相對寬鬆       |
| Mastodon                     | o       | o     | 免費，依伺服器限制       |
| X / Twitter                  | 依付費額度   | 依付費額度 | 不宜假設能完全免費       |

其中 Facebook 個人檔案的「受限」是指某些經授權的貼文讀取情境存在，但不等於一般開發者能免費、完整備份自己的所有歷史貼文。

### **1. Facebook**

**個人 Profile 與粉絲專頁 Page 必須分開評估。**

Facebook Page Feed API 明確支援讀取及發布，發布通常需要 `pages_manage_posts`、`pages_read_engagement` 等權限。

相反地，Facebook 個人檔案不能透過一般 Graph API 自動發布，而 `user_posts` 等讀取權限也有審核及存取範圍限制。

此外，Page 的 `published_posts` 端點有約每年 600 篇排名後貼文的讀取限制，因此不能直接把這個端點視為完整歷史封存工具。

Facebook Developers+2

**評估結果：** 如果主要使用 Facebook 個人檔案，官方資料匯出比自行開發 Graph API 更適合作為歷史備份來源。

### **2. Instagram**

目前官方 API 主要服務專業帳號（Business / Creator），提供讀取媒體和發布功能；一般個人帳號不在支援範圍。

而且發布有媒體格式、額度及權限限制，並非任意 Markdown 純文字都可以直接發布。

Postman API Network+1

**評估結果：** 如果不想將 Instagram 帳號轉為專業帳號，API 自動化基本上不適用。

### **3. Bluesky 和 Mastodon**

這兩個平台最適合拿來驗證整套自動化系統，因為有相對開放的官方 API，免費讀寫也比較容易實現。

**Bluesky 官方配額**

寫入操作

## **5,000**

點數／小時／帳號

寫入操作

## **35,000**

點數／天／帳號

建立一筆記錄通常消耗 3 點，因此對一般個人部落格的貼文頻率來說相當充裕。

Bluesky Protocol Services

**Mastodon 官方預設配額**

## **300 次**

API 請求／5 分鐘／帳號（伺服器可有不同設定）

上傳媒體等操作另有獨立限制。

Mastodon documentation

### **4. Threads 和 X 的限制**

**Threads：** 官方提供讀取、發布相關 API，使用前需要建立 Meta 開發者應用並取得相應權限。Meta 的官方限流文件顯示，Threads API 有依帳號曝光量計算的請求配額，因此不能視為無限免費使用。

Facebook Developers

**X：** API 的收費與方案已多次調整。這次沒有取得足以確認 2026 年 11 月完整免費讀寫額度的官方價格資料，因此不把 X 列為已確認免費的平台。正式選用前應查詢 [X Developer 官方文件](https://docs.x.com/?utm_source=chatgpt.com) 與開發者控制台中的實際價格。

### **5. 免費不等於一定能投入正式使用**

對 Meta 平台，除了請求配額，還要評估：

- **權限核准：** 某些 API scope 需要 App Review 或進階存取。
    
- **帳號資格：** Facebook Page、Instagram Professional Account 與一般個人帳號不同。
    
- **權杖有效期：** 有些 Token 需要定期更新或重新授權。
    
- **歷史資料完整性：** API 能讀最新貼文，不代表能取回所有歷史貼文、相簿、留言或原始媒體。
    

因此，規劃時我會把平台分成「可以免費測試」、「可免費長期運作但需要審核」、「不支援所需功能」、「費用待確認」四個等級，而不是只有免費／付費兩種。

## **二、自動化程式在哪裡執行？**

你不一定需要自己的電腦，也不一定需要租 VPS。

以下比較以每天備份一次、每週發布數篇、使用 Python 程式為假設。

|執行環境|免費條件|無需自己電腦開機|自動定期執行|適合程度|
|---|---|---|---|---|
|**GitHub Actions**|公開 Repo 的標準 Runner 免費|||最推薦|
|Cloudflare Workers|有免費請求及 CPU 額度|||適合輕量 API 工作|
|Google Apps Script|Google 免費服務配額內|||適合 JavaScript|
|自架 n8n|軟體免費，主機成本另計|視主機而定||適合複雜流程|
|本機 Python + 排程|無雲端執行費||電腦須開機|備用方案|

### **1. GitHub Actions**

這是最契合你目前 GitHub + Obsidian + Markdown 架構的方案。

GitHub Free

適合零成本測試

公開 Repository

## **免費**

標準 GitHub-hosted Runner

私人 Repository

## **2,000 分鐘**

每月包含的傳統標準 Runner 額度

官方計費資訊的呈現已包含 compute usage 等新計量方式，實際帳單以帳號的計費頁面為準。

GitHub Docs+2

假設每天執行兩次備份，每次兩分鐘，一個月約 120 分鐘。即使使用私人 Repo，這種量級通常也遠低於免費額度。

但仍有幾項重要限制：

- 排程最短間隔為 5 分鐘，但不保證準時，壅塞時可能延遲甚至漏執行。
    
- 公開 Repo 連續 60 天無活動，排程可能被自動停用。
    
- 雲端 Runner 是暫時性環境，每次任務結束後不能依賴本機檔案繼續存在。
    
- 若備份內容包含私人貼文，不應直接存入公開 Repository。
    

GitHub 官方：[Scheduled workflows](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows?utm_source=chatgpt.com#schedule)。

GitHub Docs

### **2. Cloudflare Workers**

若你的工作只需要：

讀取 API → 轉換資料 → 呼叫 GitHub API 提交 Markdown

那麼 Workers 也是值得考慮的選項。它能使用 Cron Triggers，且不需要自己的電腦開機。

不過它是比較偏 JavaScript／TypeScript 與 Web API 的執行環境，Python 套件相容性和執行模型需要另外評估。

官方價格：[Cloudflare Workers Pricing](https://developers.cloudflare.com/workers/platform/pricing/?utm_source=chatgpt.com)。

### **3. Google Apps Script**

適合以 JavaScript 發送 HTTP 請求，配合時間觸發器，每天抓取社群資料。

優點是不需要自行維護主機，缺點是單次執行時間、每日配額等限制，而且與你原本偏 Python + GitHub 的工具鏈整合度較低。

官方：[Apps Script Quotas](https://developers.google.com/apps-script/guides/services/quotas?utm_source=chatgpt.com)。

### **4. 自己的電腦**

Windows 工作排程器（Task Scheduler）或 Linux/macOS 的排程服務也可以執行同一套 Python 程式。

這適合需要瀏覽器操作、處理大型媒體或人工登入的平台，但如果電腦關機，就無法按時執行。也可以設計成下次開機補跑。

所以我的建議是：**把 Python 程式設計成既能在 GitHub Actions 執行，也能在本機手動執行**，不必一開始就綁死其中一種環境。

## **三、綜合評估：哪一種組合最值得採用？**

|組合|成本|自動化|我的評估|
|---|---|---|---|
|Markdown + GitHub Actions + Bluesky/Mastodon API|免費|雙向|優先實驗|
|Markdown + GitHub Actions + Threads API|可免費|雙向，受權限限制|值得測試|
|Markdown + GitHub Actions + FB Page API|可免費|雙向，受權限限制|適合有粉專者|
|Markdown + GitHub Actions + IG Professional API|可免費|雙向，受媒體格式限制|有條件採用|
|FB 個人檔案 + 官方資料匯出 + 本機 Python|免費|半自動|適合歷史備份|
|X API + 任一執行環境|待確認|依 API 額度|暫不列為零成本核心|

## **四、規劃階段我會採用的技術架構**

**共用 Python 程式**

讀取／發布／轉換／去重／記錄同步狀態

**GitHub Actions**

主要：雲端定期執行

**本機 Python**

備用：手動或排程

各平台獨立 API Adapter

Facebook / Instagram / Threads / Bluesky / Mastodon

**Markdown Repository**

Obsidian 編輯、Git 保存、Astro 建置網站

這種設計有一個很重要的好處：**日後變更執行平台，不需要重寫各社群 API 的核心邏輯。**

若未來 GitHub Actions 不符合需求，可以把同一套 Python 程式搬到 VPS、Docker、家用伺服器，或本機執行。

還要特別區分兩種資料：公開部落格文章可以放在公開 Git Repo，但私人社群備份、存取權杖和未公開附件應保存在私人儲存空間。GitHub Pages 本身並不是私人備份服務。

### **目前的決策建議**

我的優先順序是：

1. **先確認社群 API 是否真的允許自己的帳號免費讀寫**，尤其 Meta 各種帳號類型及 App Review。
    
2. 執行環境先選 GitHub Actions，因為成本及維護負擔都很低。
    
3. 備份與發布分開實作，不強求每個平台都能雙向同步。
    
4. 對不開放 API 的平台，保留官方資料匯出 + 本機轉檔的途徑。
    

下一步最值得深入查的其實不是怎麼寫程式，而是 **Facebook 個人檔案、Instagram 一般帳號、Threads 這三種 Meta 帳號，在不付費且只供自己使用的情況下，到底可以取得哪些 API 權限，以及哪些必須進行 App Review**。這會最直接影響整套方案的可行性。