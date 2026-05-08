指令：python 01_pagespeed_submitter.py  python 02_pagespeed_submitter.py python 03_pagespeed_submitter.py

---

# 📄 PageSpeed Insights 分段式自動化檢測管線

本系統由三支獨立的 Python 腳本組成，構成一條完整的自動化資料管線（Data Pipeline）。系統設計將「耗時的網頁檢測」、「畫面資料抓取」與「文本資料清洗」三個動作完全解耦，大幅降低因 Google 伺服器延遲或結構異動而導致任務全面崩潰的風險。

## 1. 需準備的輸入資料與執行流程 (Input Flow)

本系統需**嚴格按照順序執行**，前一支程式的產出將作為下一支程式的輸入：

### 階段一：`01_pagespeed_submitter.py`

* **輸入方式**：無須外部檔案。請直接修改程式碼內的 `input_data_list` 陣列，填入欲檢測的目標網址。
* **執行動作**：驅動 Chrome 逐一將網址送入 Google PageSpeed，等待分析圓圈出現後，抓取專屬的永久報表連結。

### 階段二：`02_pagespeed_scraper.py`

* **輸入方式**：自動讀取第一階段產生的 `selenium_safe_results.csv`。
* **執行動作**：讀取 CSV 中的永久連結，切換「行動版」與「電腦版」標籤，並將整個網頁的純文字 (`body.text`) 暴力抓取下來。

### 階段三：`03_pagespeed_parser.py`

* **輸入方式**：自動讀取第二階段產生的 `PageSpeed_Full_Data.xlsx`，並透過程式碼內的 `ID_TO_NAME_MAPPING` 字典，將網址自動對應為中文機關名稱。
* **執行動作**：以純資料處理的方式，切出四大分數區塊並整理為最終報表。

## 2. 預期的產出結果 (Expected Output)

隨著三個腳本依序執行，您的資料夾中會依序產生以下三個檔案：

1. **`selenium_safe_results.csv`** (中繼檔一)：
* 包含 `ID` (目標網址) 與 `Link` (Google 產生的永久報告連結)。
* 若該網址發生逾時卡死，`Link` 欄位會標示為 `ERROR`。


2. **`PageSpeed_Full_Data.xlsx`** (中繼檔二)：
* 包含 `URL`、`Type` (Mobile / Desktop) 與 `Content` (極長的未排版網頁純文字)。這是為了保留最原始的資料庫，方便未來重新解析。


3. **`extracted_pagespeed_metrics_selenium.csv`** (🎉 最終完美報表)：
* 具備完整的中文標籤，包含：`識別網站名稱 (中文)`、`裝置類型`、`URL`、`效能 Score`、`無障礙功能 Score`、`最佳做法 Score`、`SEO Score`。
* 所有分數欄位皆已轉為乾淨的「數值格式」，可直接在 Excel 進行樞紐分析或加總。



## 3. 執行環境與注意事項 (Caveats & Requirements)

### 環境依賴 (Dependencies)

請確保您的 Python 環境已安裝以下套件：

```bash
pip install pandas selenium webdriver-manager openpyxl

```

### ⚠️ 核心注意事項

* **容錯與防卡死設計 (Script 1)**：第一支程式設有 120 秒的 `wait_long` 極限等待。由於 Google PageSpeed 在尖峰時段運算極慢，若超過 120 秒仍未出分數，程式會印出警告並**強制抓取當前網址跳至下一筆**，請勿將其視為程式當機。
* **轉場動畫等待 (Script 2)**：第二支程式在點擊「電腦版 (`desktop_tab`)」後，強制寫入了 `time.sleep(3)`。這是為了等待 CSS 過場動畫與 DOM 結構抽換，請勿縮短此秒數，否則極易抓到殘留的行動版數據。
* **文字排版相依性風險 (Script 3)**：第三支程式的解析邏輯依賴於網頁上的實體文字（如尋找 `"診斷效能問題"` 與 `"搜尋引擎最佳化 (SEO)"` 之間的陣列索引 `[0, 2, 4, 6]`）。**若未來 Google PageSpeed 更改了網頁的中文翻譯用詞或上下排版順序**，您必須同步修改 Script 3 中的 `START_STR`、`END_STR` 與陣列索引值。
