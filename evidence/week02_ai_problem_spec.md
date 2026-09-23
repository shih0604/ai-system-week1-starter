# Week 2 - AI Problem Spec v0.2

## 1. Business / System Problem：
現行氣象署採用的雷達回波外延方法只能有效預測 0~1 小時的短時預報,但對於實際的防救災決策而言,1~3 小時的預測才足夠。

## 2. Decision Point：
氣象預報人員發布短延時強降水警戒、決定是否啟動防災應變前的預報判讀時刻。

## 3. Unit of Analysis：
單一時間點的雷達回波序列(以五分山雷達站為例)。

## 4. Input：
t 時刻前 1 小時雷達回波觀測序列(10 幀,每 6 分鐘一幀),含極座標多仰角或 CAPPI 多高度層資料。Train/val/test 依年份切分(2020~2024 訓練、2025 測試),避免同一事件同時出現在訓練與測試集。

## 5. Target：
Ground truth 為 CAPPI 組合C(低中高分層最大合成回波,dBZ),對齊學姊論文最佳配置。時間點為 t+6 到 t+180 分鐘(不含 t+0)。

## 6. Output：
未來 3 小時、每 6 分鐘一幀的預測回波圖(共 30 幀)。

## 7. Baseline：

氣象署現行 NSSL 外延方法
SimVP 確定性模型(重新訓練為 T_out=30,輸出 3 小時版本,確保與新模型公平比較)
Persistence(單純延續目前回波不變)作為基本 sanity check

## 8. Success Criteria：

Technical：CSI、FSS(20/40 dBZ 門檻)、RMSE 於 1~3 小時區間平均優於 baseline 至少 5%,不要求每個 lead time 都達標。
System：模型可穩定產出 30 幀預測,並整合進準作業化 Pipeline 運行。
Decision / Value：延長有效預警時效至 3 小時,提供防救災決策更充裕的應變時間。

## 9. Failure Criteria：

資料缺測時未依設計跳過、強行輸出錯誤結果
40 dBZ 以上強回波核心被模型漏失
1~3 小時預測誤差成長速度不優於現行方法,或長時效預測嚴重模糊化

## 10. HITL / Fallback：
預報人員可對照現行 NSSL 外延結果進行人工判讀與最終決策;資料缺失或模型異常時。

## 11. 最大 Assumption：
單一雷達站(五分山)之觀測資料,經延長訓練與模型調整後,仍能有效捕捉對流系統跨 3 小時的生成、增強與消散特性。



# AI Review Record

## Finding 1
AI Reviewer：Target（ground truth）定義不夠明確，包含合成方式、時間戳定義、CSI/FSS threshold 都沒講清楚。
Decision：Accept
Reason：Target 沒定義清楚，後面所有指標比較都會失去基準,這是最基本、必須先解決的問題。
Change：
Ground truth 定義為 CAPPI 組合C（低中高分層最大合成,對齊學姊論文最佳配置）
評估時間點為 t+6 到 t+180 分鐘,不含 t+0
CSI/FSS 門檻採 20 dBZ 與 40 dBZ 兩組(分別看「有沒有下雨」跟「強不強」)

## Finding 2
AI Reviewer：Input 可能有 temporal/event leakage,尤其雷達序列高度重疊,隨機切分 train/test 會讓同一事件前後幀同時出現在兩邊。
Decision：Accept
Reason：這是方法論上最嚴重的風險,如果沒處理,模型評估結果會失真,即使指標好看也無法採信。
Change：Train/val/test 依「年份」切分(沿用學姊原本 2020~2024 訓練、2025 測試的方式),不隨機切 frame,避免同一降雨事件同時出現在訓練與測試集。

## Finding 3
AI Reviewer：「優於 baseline」沒有定義多少才算優於;「誤差成長趨緩」也不是可驗收的定義。
Decision：Modify
Reason：完全量化所有細節(信賴區間、統計顯著性)對目前階段太複雜,但至少要有一個明確數字門檻才能驗收。
Change：CSI/FSS 在 1~3 小時區間平均優於 baseline 至少 5%,不要求每個 lead time 都達標;誤差成長的細節判準留待後續版本補充。

## Finding 4
AI Reviewer：SimVP baseline 如果只用原本 1 小時設定去跟 3 小時新模型比較,並不公平;且只有兩個 baseline 可能不足以支持「AI 方法有效」的結論。
Decision：Accept
Reason：這點確實成立,不公平的 baseline 會讓比較結果沒有意義。
Change：SimVP baseline 需重新訓練成同樣輸出 3 小時(T_out=30)的版本才能比較;另外加入 persistence(單純延續目前回波不變)作為最基本的 sanity check baseline。

## Finding 5
AI Reviewer：Failure Criteria 目前只涵蓋長時效模糊化,還缺資料缺測、極端事件失敗、空間偏移、時間偏移、系統運作失敗等多種 failure case。
Decision：Modify
Reason：全部類型對目前階段範圍太大,先聚焦最重要、最可能實際發生的兩類,其餘留待後續版本。
Change：新增兩類 failure:①資料缺測時 Pipeline 自動跳過該時間點,不強制輸出(沿用學姊 operation 設計原則);②40 dBZ 以上強回波核心被漏失,視為 failure。空間偏移、時間偏移等其餘類型暫不在本版定義範圍內。