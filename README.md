# ARIA v5.0 — 馬太鞍三幕稽核系統 (The Matai'an Three-Act Auditor)

本專案為 ARIA 系統的第 8 週擴充模組。透過 Microsoft Planetary Computer STAC API 串流 Sentinel-2 L2A 衛星影像，我們針對 2025 年馬太鞍溪堰塞湖事件進行了完整生命週期（災前、災中、災後）的光學鑑識稽核。

本系統成功萃取出三大災害遮罩（堰塞湖、崩塌源頭、下游土石流），並與 W7 瓶頸節點及 W8 光復鄉基礎設施圖層進行空間疊合，產出最終的「現場目擊影響評估表」。

---

## 🛰️ 1. 三幕場景選擇 (Reproducible Scene IDs)

經過嚴格的 TCI 視覺品管，我們挑選出涵蓋事件完整生命週期的三張核心 Sentinel-2 L2A 影像。這三幕完美避開了季風雲層的干擾，確保了馬太鞍河谷上空的清晰度：

* **Act 1: Pre (災前 - 原始森林)**
    * **Date**: 2025-06-15 (Cloud Cover: 8.5%)
    * **Item ID**: `S2A_MSIL2A_20250615T023141_R046_T51QUG_20250615T070417`
* **Act 2: Mid (災中 - 堰塞湖形成，86公頃峰值)**
    * **Date**: 2025-09-11 (Cloud Cover: 13.5%)
    * **Item ID**: `S2C_MSIL2A_20250911T022551_R046_T51QUG_20250911T055914`
* **Act 3: Post (災後 - 潰堤與下游土石流)**
    * **Date**: 2025-10-16 (Cloud Cover: 2.5%)
    * **Item ID**: `S2B_MSIL2A_20251016T022559_R046_T51QUG_20251016T042804`

---

## 🛑 2. 覆蓋範圍落差分析 (Coverage Gap Analysis)

從最終生成的 `impact_table.csv` 可以觀察到一個嚴重的系統盲點：既有的 ARIA 災前預警模型未能發揮預期作用。

我們針對 W7 瓶頸節點與 W8 光復鄉套疊節點進行交集運算後發現，災難衝擊（土石流掩埋）集中於光復鄉及萬榮鄉周邊。若我們將資源與預防性撤離計畫僅侷限於都會區，將導致山區聚落與偏鄉在面臨複合型災害（如崩塌引發堰塞湖再導致下游洪患）時完全失去預警保護。

**建議處置：** 縣政府應立即擴展 ARIA 的建模範圍，將全縣的河系、山區潛勢溪流與下游鄉鎮（特別是光復鄉）全面納入基礎設施資料庫中，並建立以流域為單位的上下游連動預警機制。

---

## 🛠️ 3. AI 診斷日誌 (AI Diagnostic Log)

在建構 ARIA v5.0 的過程中，我們遭遇並排除了以下技術挑戰：

1.  **影像載入防護機制 (`DecompressionBombWarning`)**
    * **問題**：在透過 PIL 擷取 Sentinel-2 TCI 縮圖時，因原始圖磚高達 1.2 億畫素，觸發了 Python 影像處理庫的解壓縮炸彈安全限制，導致無法顯示預覽圖。
    * **解法**：在腳本頂端配置 `Image.MAX_IMAGE_PIXELS = None` 解除限制，並改用 STAC 提供的 `"rendered_preview"` 資產取代完整 `"visual"` 串流，將預覽載入時間從數分鐘縮短至 3 秒內。
2.  **堰塞湖零偵測 (Empty Barrier Lake Mask)**
    * **問題**：初次生成 C1 堰塞湖遮罩時，回傳的像素總數為 0，整張圖表呈現空白。
    * **解法**：重新審視物理參數，確認災中 (Mid) 的堰塞湖水體夾帶大量崩塌泥沙，屬**高濁度水體**。將 `nir_mid` 閾值從常規的 0.15 放寬至 0.18 後，成功捕捉到位於萬榮上游的湖泊範圍。同時，將寫死的 X 座標空間閘道，改為自動抓取影像 `x_min` 與 `x_max` 動態計算邊界比例（35%），消除了人為寫入固定座標帶來的裁切誤差。
3.  **JSON 隱形字元報錯 (`ValueError: Expected object or value`)**
    * **問題**：在讀取 W7 (`top5_bottleneck_coordinates.json`) 時，GeoPandas 與 Pandas 皆拒絕解析，顯示格式錯誤。以純文字模式檢驗後，發現檔案第一行包含了隱藏的 UTF-8 BOM (`\ufeff`) 標記。
    * **解法**：捨棄常規讀取方式，改用原生的 `open(..., encoding="utf-8-sig")` 強制忽略 BOM 字元，再轉換回 Pandas DataFrame 與 GeoDataFrame，並將 CRS 從原生的 EPSG:3826 轉換至目標 EPSG:32651，順利完成空間疊合。

---

## 📂 4. 輸出檔案結構

執行 `ARIA_v5_mataian.ipynb` 後，系統會於 `output/` 目錄生成以下核心檔案：

* `change_metrics.png`: 四項變遷指標之視覺化。
* `three_masks.png` / `three_masks_fixed.png`: 堰塞湖、崩塌源、土石流之二元遮罩結果。
* `mataian_detections.gpkg`: 包含上述三大災害多邊形的 GeoPackage 向量圖層。
* `impact_table.csv`: 空間交集後的最終現場目擊影響評估表。