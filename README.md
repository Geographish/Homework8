# ARIA v5.0: The Matai'an Three-Act Auditor (馬太鞍三幕稽核系統)

## 1. STAC 影像來源 (Reproducible Scene IDs)
* **Pre (災前)**: `S2A_MSIL2A_20250615T023141_R046_T51QUG_20250615T070417`
* **Mid (災中/堰塞湖)**: `S2C_MSIL2A_20250911T022551_R046_T51QUG_20250911T055914`
* **Post (災後/潰決)**: `S2B_MSIL2A_20251016T022559_R046_T51QUG_20251016T042804`

---

## 2. AI 顧問營運簡報 (Generated via Groq API LLaMA-3.3)
**營運簡報**

**一、 三幕時間軸確認：衛星影像證明了什麼**
根據 ARIA v5.0 的三幕時間軸分析，衛星影像證明了 Matai'an Creek barrier lake 事件的前、中、後三個階段。Act 1 顯示了森林茂盛的 Matai'an 谷地，沒有湖泊的存在。Act 2 顯示了 barrier lake 的形成，而 Act 3 顯示了湖泊的排空和山體滑坡的源頭驗證。

**二、 預警窗口期 (Jul 21 to Sep 23, 64 days)：如果 ARIA 當時有上線，能提供什麼警告**
如果 ARIA v5.0 當時已經上線，則可以在 64 天的預警窗口期內提供警告，幫助我們提前預防和應對災害。這個預警窗口期可以讓我們進行搶險和疏散工作，減少災害的影響。

**三、 覆蓋範圍落差 (Coverage Gap)：為什麼 W3/W7 錯失了光復鄉？我們該如何改進？**
根據 ARIA CHAIN SUMMARY，W3 和 W7 分析了 42 個花蓮縣的避難所和 5 個瓶頸，但卻錯失了光復鄉。這個覆蓋範圍落差可能是由於資料不足或分析方法的局限性引起的。為了改進，我們需要收集更多的資料和優化分析方法，確保所有地區都能受到有效的覆蓋。

**四、 未來 24 小時命令：包含優先搶通、收容所補給、無人機(UAV)任務**
根據分析結果，我們需要優先搶通受災地區的交通路線，補給收容所的物資，並使用無人機進行搜索和救援工作。這些任務需要在未來 24 小時內完成，以確保受災民眾的安全和福祉。

**五、 模型改進建議：提出一個具體建議來擴展 ARIA 系統**
為了擴展 ARIA 系統，我們提出以下建議：增加更多的資料來源，例如社交媒體和感測器資料，來提高分析的準確性和覆蓋範圍。同時，需要優化分析方法，例如使用機器學習算法，來提高預警的準確性和時效性。這些改進可以幫助我們更好地預防和應對災害，保護民眾的生命和財產。

---

## 3. AI 診斷日誌 (AI Diagnostic Log)
* **問題 1：STAC API 伺服器連線逾時**
  * **狀況**：在執行 `catalog.search` 檢索影像時，頻繁遭遇 `APIError: The request exceeded the maximum allowed time`。
  * **解法**：我實作了「指數退避重試機制 (Exponential Backoff Retry)」，透過 `try-except` 捕獲錯誤並搭配時間延遲重試，成功解決 STAC 伺服器不穩定的問題。
* **問題 2：LLM Token 溢位與偽陽性雜訊處理**
  * **狀況**：最初模型將光復鄉平地的裸露稻田全數誤判為崩塌源，導致上千個雜訊節點，且未過濾的 `impact_table` 直接導致 Groq API 發生 `BadRequestError` (Token 超載)。
  * **解法**：首先，我引入了**空間閘道 (Spatial Gate)** (`pre.x < gate_x_debris`) 將崩塌偵測嚴格限制於西側高山區域。接著，在傳遞給 LLM 時，使用 Pandas 過濾邏輯 (`impact_table[impact_table["Notes"] != "outside event area"]`) 僅擷取受災設施名單，成功完成輕量化的 AI 報告自動生成。
