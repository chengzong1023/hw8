# ARIA v5.0 — Matai'an Three-Act Auditor

## Event
**2025 馬太鞍溪堰塞湖事件** (Matai'an Creek Barrier Lake, Hualien County)

| Date | Event |
|------|-------|
| Jul 21, 2025 | Typhoon Wipha triggers landslide → barrier lake forms (~200 m deep) |
| Sep 11, 2025 | Lake reaches peak area ~86 ha |
| Sep 23, 2025 14:50 | Lake breaches → 15.4M m³ released in 30 min → 18+ fatalities |

---

## Reproducible STAC Item IDs

```python
PRE_ITEM_ID  = "S2A_MSIL2A_20250615T023141_R046_T51QUG_20250615T070417"
MID_ITEM_ID  = "S2C_MSIL2A_20250911T022551_R046_T51QUG_20250911T055914"
POST_ITEM_ID = "S2B_MSIL2A_20251016T022559_R046_T51QUG_20251016T042804"
```

| Act | Item ID | Date | Cloud Cover | Key Feature |
|-----|---------|------|-------------|-------------|
| PRE  | `S2A_MSIL2A_20250615T023141_R04...` | 2025/06/15 | 8.5%  | Original forest, no anomaly |
| MID  | `S2C_MSIL2A_20250911T022551_R04...` | 2025/09/11 | 13.5% | Barrier lake peak ~86 ha |
| POST | `S2B_MSIL2A_20251016T022559_R04...` | 2025/10/16 | 2.5%  | Lake gone; Guangfu sediment |

---

## Detection Results

| Mask | Area (km²) | Method |
|------|-----------|--------|
| C1 Barrier Lake | 1.046 | Turbid water spectral rule (NIR 0.10–0.18) + spatial gate |
| C2 Landslide Source | 19.818 | NIR Drop + SWIR, F1-tuned (best F1=0.533) |
| C3 Debris Flow (Guangfu) | 9.498 | NDVI Drop + BSI Change, lon > 121.35°E gate |

---

## Coverage Gap Discussion

W3 (花蓮市避難所) 和 W7 (花蓮市道路瓶頸) 的涵蓋範圍以花蓮市為中心，
距離光復鄉約 **30 公里**。空間連接結果顯示 W3/W7 零命中——
這不是系統錯誤，而是 SOP 圖層的地理缺口。

**三個層面的缺口**:
1. **地理缺口**: W3/W7 BBOX 根本未涵蓋光復鄉座標
2. **風險模型缺口**: W4 未將萬榮上游堰塞湖列為高風險
3. **網路拓撲缺口**: W7 不含台9線馬太鞍溪橋節點

→ W8 Guangfu overlay 的建立正是為了填補此缺口。

---

## AI Diagnostic Log

### 問題：Mid 事件窗口（8–9 月）幾乎全是雲，如何找到可用場景？

**遭遇**：
直接查詢 `cloud_cover < 20%` 在 2025/08/01–09/20 窗口回傳 0 筆。
「C2 landslide mask 面積達 19.8 km²，F1 僅 0.533，研判閾值過寬導致假陽性（河床砂洲、收割農地）混入。」
**解法**：
1. 放寬閾值至 `cloud_cover < 40%`，回傳 7 筆候選場景
2. 用 `show_top3_tci()` 串流 TCI 縮圖（`overview_level=3`），逐一人眼確認馬太鞍溪谷上空
3. 發現 `S2C_MSIL2A_20250911` 雖全幅雲量 13.5%，但雲層集中在東側海岸，
   谷地（興趣區域）清晰可見
4. 選定此場景，同時確認日期（9/11）與 NCDR 湖面峰值紀錄吻合
5. 若有 W4 坡度資料（slope > 15°）可進一步過濾，預期面積可縮小至 3–5 km²。

**教訓**：tile-level `eo:cloud_cover` 是整張衛星影像的統計值，
不代表 AOI 範圍的雲況。TCI Quick-QA 才是最後決策依據。


---

## Output Files

```
ARIA_v5_mataian.ipynb          # Main analysis notebook
mataian_detections.gpkg        # 3 detection layers (barrier_lake / landslide_source / debris_flow)
impact_table.csv               # Eyewitness Impact Table
output/
  three_act_tci_panel.png      # Three-act TCI side-by-side
  tci_candidates_pre/mid/post.png
  change_metrics_pre_mid.png
  change_metrics_pre_post.png
  nir_drop_pre_mid/post.png
  swir_post_mid/post.png
  bsi_change_pre_mid/post.png
  ndvi_change_pre_mid/post.png
  mask_c1_barrier_lake.png
  mask_c2_landslide_source.png
  mask_c3_debris_flow.png
  final_impact_map.png
  ai_advisor_brief.txt         # AI Advisor operational brief (if API key set)
```
