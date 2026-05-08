指令：python PageSpeed_Crawler.py

---

# 📄 PageSpeed Insights 自動化巡檢工具

本工具為基於 Python 與 Selenium 構建的網頁效能自動化稽核腳本。主要用於批量檢測多個政府機關或專案網站的 Google PageSpeed Insights 分數，涵蓋「效能」、「無障礙功能 (A11y)」、「最佳做法」與「SEO」四大指標，並自動導出包含電腦版與行動版數據的 Excel 報表。

## 1. 需準備的輸入資料 (Input Data)

目前腳本採用**內部字典 (Dictionary)** 的方式進行資料配置，無需額外匯入外部 CSV 或 Excel 檔案。每次執行前，請直接於程式碼區塊 `--- 1. 設定輸入資料與對應名稱 ---` 中的 `SITE_DATA` 變數增刪網站清單。

**格式規範範例：**
請確保輸入格式為 `{"目標網址": "網站自訂名稱"}` 的鍵值對 (Key-Value Pair)。

```python
SITE_DATA = {
    # "完整的 HTTPS 網址": "報表上顯示的專案/網站名稱",
    "https://www.cha.gov.tw/mp-1.html": "化學署中文全球資訊網",
    "https://topic.moenv.gov.tw/evsu/": "環境用藥安全使用宣導網"
}

```

* **欄位說明**：
* `Key (網址)`：必須是完整的絕對路徑（包含 `https://`）。這會直接傳遞給 Google 伺服器進行解析。
* `Value (名稱)`：純文字字串。此名稱將直接寫入最終輸出的報表中，作為該網域的易讀標籤。



## 2. 預期的產出結果 (Expected Output)

腳本執行完畢後，會在程式執行的同一個資料夾目錄下，自動生成一份名為 **`PageSpeed_Final_Report.xlsx`** 的 Excel 報表。

**報表欄位結構如下：**

| 欄位名稱 | 資料型態 | 說明 |
| --- | --- | --- |
| **網站名稱** | 字串 | 對應 `SITE_DATA` 中設定的中文網站名稱。 |
| **裝置類型** | 字串 | 區分測試環境，分為「行動版 (Mobile)」與「電腦版 (Desktop)」。 |
| **效能** | 數值 | 網頁載入速度與核心網頁指標 (Core Web Vitals) 評分 (0-100)。 |
| **無障礙功能** | 數值 | 網頁 A11y 標籤與結構的合規性評分 (0-100)。 |
| **最佳做法** | 數值 | 網站安全性、現代網頁標準合規性評分 (0-100)。 |
| **SEO** | 數值 | 搜尋引擎最佳化結構評分 (0-100)。 |
| **目標網址** | 字串 | 實際進行檢測的目標 URL。 |
| **永久報告連結** | 網址 | Google 官方生成的帶 ID 分享連結，可用於後續人工覆核。若發生超時則顯示 `Timeout` 或 `Failed`。 |

> **提示**：報表中的四大分數欄位已在輸出前自動轉為「數值格式」，您可以直接在 Excel 中進行平均值計算、排序或繪製樞紐分析圖表。

## 3. 執行環境與注意事項 (Caveats & Requirements)

### 環境依賴 (Dependencies)

確保您的 Python 環境已安裝以下套件。雖然腳本內未直接 `import openpyxl`，但 Pandas 匯出 Excel 必須依賴該套件：

```bash
pip install pandas selenium webdriver-manager openpyxl

```

### 注意事項

* **網路連線與超時機制**：Google PageSpeed 伺服器運算時間浮動極大。本腳本設有 `120秒` 的長等待上限（等待分數區塊出現）。請確保在網路穩定的環境下執行，若單一網站超過此時間，腳本會紀錄為 `Timeout` 並自動跳至下一個網站，不會導致程式崩潰。
* **畫面過場控制**：腳本在切換「電腦版」分頁時，強制設定了 `time.sleep(3)`。這是為了等待 CSS 動畫與 DOM 結構完全刷新，**請勿輕易縮短此秒數**，否則極易導致 Regex 誤抓到殘留的行動版文字。
* **背景執行 (Headless Mode)**：腳本第 61 行備有隱藏瀏覽器視窗的選項 (`# options.add_argument("--headless")`)。如果您將此工具部署至排程伺服器或不希望跳出視窗干擾工作，可以取消該行的註解。
* **正則表達式 (Regex) 風險**：本工具透過正則表達式從網頁全文解析分數。若未來 Google PageSpeed Insights 更改了網頁的 DOM 結構或中文顯示名稱（例如將「最佳做法」改名為「最佳化實務」），`extract_scores` 函式中的 `patterns` 字典將需要同步維護更新。
