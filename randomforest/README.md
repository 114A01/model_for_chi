# CICIOT2023 - 模型說明與訓練紀錄

## 概要
- 本專案使用隨機森林（Random Forest）對網路流量/攻擊類別進行多類別分類（8 類）。
- 模型與報告位於 `models/` 資料夾。

## 檔案一覽
- 模型檔案：[models/complete_model_20260506_130625.joblib](models/complete_model_20260506_130625.joblib)
- 模型檔案：[models/complete_model_20260508_210127.joblib](models/complete_model_20260508_210127.joblib)
- 訓練報告：[models/model_report_20260506_130625.txt](models/model_report_20260506_130625.txt#L1-L40)
- 訓練報告：[models/model_report_20260508_210127.txt](models/model_report_20260508_210127.txt#L1-L40)
- Jupyter Notebook（含前處理示例）：[CICIOT2023.ipynb](CICIOT2023.ipynb#L84-L110)

## 資料與前處理（摘要）
- 原始資料共使用大量樣本：訓練集約 29,846,653 筆，測試集 8,528,041 筆；驗證集（若有）約 4,263,382 筆。
- 特徵數量：46。
- 類別：8 類（Benign、Brute Force、DDoS、DoS、Mirai、Recon、Spoofing、Web-based）。
- 前處理策略（Notebook 範例）:
  - 將 inf 值替換為 NaN，計算每列缺失比例。
  - 只刪除缺失率超過 50% 的行，其餘缺失值採用中位數填補（數值欄位）。
  - 保留較完整資料以避免過度丟棄樣本（參考 [CICIOT2023.ipynb](CICIOT2023.ipynb#L84-L110)）。

## 訓練方式與過程
- 演算法：隨機森林（Random Forest），以完整特徵集訓練多類別分類器。
- 訓練流程概述：資料清洗 → 特徵工程（欄位選擇/標準化視需要）→ 切分訓練/驗證/測試集 → 訓練隨機森林 → 評估（測試集、驗證集）→ 匯出模型（.joblib）。
- 訓練重點：樣本不均衡為主要挑戰（例如 DDoS 類別樣本量極大），以整體加權評估與各類別指標觀察模型表現。

## 模型細節
- 輸入特徵：46 個欄位（數值特徵為主）。
- 輸出：8 類多分類機率/標籤。
- 模型輸出檔案：見上方 `models/` 連結。

## 評估指標與結果（摘要）
- 測試集整體準確率（accuracy）：0.9958（兩份訓練報告一致）。
- 測試集分類細項（節錄自訓練報告）：

```
類別       precision   recall   f1-score   support
Benign       0.91      0.98      0.94    219,650
Brute Force  1.00      0.47      0.64      2,613
DDoS         1.00      1.00      1.00  5,988,154
DoS          1.00      1.00      1.00  1,618,228
Mirai        1.00      1.00      1.00    526,851
Recon        0.91      0.82      0.86     70,917
Spoofing     0.92      0.85      0.89     97,306
Web-based    1.00      0.32      0.49      4,322
```

- 觀察重點：
  - DDoS、DoS、Mirai 等大類別取得極高 recall/precision；整體 accuracy 接近 0.996。
  - 少數類別（例如 `Brute Force` 與 `Web-based`）的 recall 明顯偏低，代表這些類別的偵測仍有改進空間（可能因樣本數少或特徵不足）。

## 如何載入與使用模型（範例）
在 Python 中載入 `.joblib` 模型：

```python
from joblib import load
model = load('models/complete_model_20260508_210127.joblib')
# 假設 X_new 已為處理好的特徵矩陣
preds = model.predict(X_new)
probs = model.predict_proba(X_new)
```

## 注意事項與建議（後續改進）
- 針對少數類別（Brute Force、Web-based）採取下列策略：
  - 增加資料量（資料擴增或收集更多該類樣本）。
  - 針對該類別進行類別重抽樣（over/under-sampling）或調整 class_weight。
  - 嘗試更強的特徵工程或使用專門的二分類器做二階段分類（先粗分類再細分類）。
- 若要部署於實時檢測環境，建議：
  - 評估模型推論延遲與記憶體佔用（隨機森林對規模較大模型推論成本較高）。
  - 考慮以輕量化模型或透過模型壓縮/樹剪枝降低推論成本。

## 參考
- 詳細訓練報告請見 [models/model_report_20260508_210127.txt](models/model_report_20260508_210127.txt#L1-L40) 與 [models/model_report_20260506_130625.txt](models/model_report_20260506_130625.txt#L1-L40)。

---

