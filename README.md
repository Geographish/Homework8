# ARIA v5.0: The Matai'an Three-Act Auditor (馬太鞍三幕稽核系統)

ARIA (Automated Remote Sensing Impact Auditor) 的第五版，透過Sentinel-2 衛星影像，自動化稽核 2025 年馬太鞍溪巨型堰塞湖由「形成」、「蓄水」至「潰決」的三幕式演變。系統透過計算光譜變遷指標，產出災害遮罩，並與在地關鍵基礎設施進行空間交集運算，最終結合 Groq LLM 生成防災簡報。


## 1. STAC 影像來源 (Reproducible Scene IDs)
* **Pre (災前)**: `S2A_MSIL2A_20250615T023141_R046_T51QUG_20250615T070417`
* **Mid (災中/堰塞湖)**: `S2C_MSIL2A_20250911T022551_R046_T51QUG_20250911T055914`
* **Post (災後/潰決)**: `S2B_MSIL2A_20251016T022559_R046_T51QUG_20251016T042804`



## 2. AI 診斷日誌 
* **問題：土石流遮罩與崩塌遮罩重疊了 — 我是如何決定保留哪一個的？**

* **解決方案**：在空間邏輯上，崩塌源 (Landslide) 位於西側高山陡坡，而土石流 (Debris Flow) 是其向東沖刷至平原的扇狀軌跡。透過設立 X 座標界線（`pre.x < gate_x_debris`），將變遷偵測嚴格劃分：西側高山區才符合 C2 崩塌源的空間要件，而延伸至東側光復平原區的堆積則歸類為 C3 土石流，以此解決遮罩重疊與平地農田被誤判為崩塌源的問題。

## 3. 資料格式

```text
ARIA_v5_Project/
│
├── ARIA_v5_Mataian_Auditor.ipynb    # 核心 Jupyter Notebook 主程式
├── README.md                        # 本專案說明文件
└── output/                          # 分析結果與視覺化輸出資料夾
    ├── mataian_detections.gpkg      # 馬太鞍三幕災害
    ├── change_metrics.png           # 災前/災中/災後光譜變遷指標圖 (NDVI, BSI, NIR Drop)
    ├── final_impact_map.png         # ARIA v5.0 最終影響地圖 (結合災害遮罩與受災設施點位)
    ├── three_masks_tuned.png        # 三大災害遮罩
    └── impact_table.csv             # 現場目擊影響評估表 (Eyewitness Impact Table)