 # CICIOT2023 - 隨機森林模型說明與訓練紀要

 ## 概要
- 本專案使用隨機森林（Random Forest）進行多類別分類（8 類），用於辨識網路流量中的惡意/良性類型。
- 模型與訓練報告存放於 `models/` 資料夾。

 ## 主要檔案
- 最新模型檔：[models/complete_model_20260512_131827.joblib](models/complete_model_20260512_131827.joblib)
- 最新訓練報告：[models/model_report_20260512_131827.txt](models/model_report_20260512_131827.txt#L1-L120)
- Jupyter Notebook（含前處理與訓練程式）：[CICIOT2023.ipynb](CICIOT2023.ipynb#L1-L200)

## 資料與特徵摘要
- 訓練集樣本數：32,680,605
- 測試集樣本數：9,337,782
- 驗證集樣本數：4,668,192
- 特徵數量：46
- 類別數量：8（Benign、Brute Force、DDoS、DoS、Mirai、Recon、Spoofing、Web-based）

## 訓練與評估概況
- 演算法：隨機森林（Random Forest），多類別分類。
- 訓練流程：資料清洗 → 特徵工程 → 切分訓練/驗證/測試 → 訓練 & 調參 → 評估 → 匯出模型（.joblib）。

## 評估摘要（節錄自最新訓練報告）
- 測試集整體準確率（accuracy）：0.9962

測試集分類報告（節錄）：

```
              precision    recall  f1-score   support

      Benign       0.91      0.98      0.95    219650
 Brute Force       1.00      0.49      0.66      2613
        DDoS       1.00      1.00      1.00   6797251
         DoS       1.00      1.00      1.00   1618228
       Mirai       1.00      1.00      1.00    526851
       Recon       0.92      0.82      0.86     70917
    Spoofing       0.92      0.85      0.88     97306
   Web-based       1.00      0.40      0.57      4966
```

- 觀察要點：
  - 對於樣本數極大的類別（如 DDoS、DoS、Mirai），模型在 precision/recall 上表現極佳；整體 accuracy 為 0.9962。
  - 少數類別（如 `Brute Force`、`Web-based`）的 recall 仍偏低，建議以資料擴增或類別重抽樣改善偵測率。

## 範例：如何載入模型
在 Python 中載入 `.joblib` 模型：

```python
from joblib import load
model = load('models/complete_model_20260512_131827.joblib')
# 假設 X_new 已為處理好的特徵矩陣
preds = model.predict(X_new)
probs = model.predict_proba(X_new)
```

## 建議與注意事項
- 若要提升少數類別偵測：
  - 增加該類別資料（收集或資料擴增）。
  - 使用 over-/under-sampling 或調整 `class_weight`。
  - 探索二階段分類（先偵測大類，再針對少數類別細分）。
- 部署考量：隨機森林模型在大型樹集合上會增加推論時延與記憶體需求，若要部署於即時系統，請評估延遲、記憶體與可用性，或考慮模型壓縮/樹剪枝或替換為較輕量模型。

## 參考
- 詳細報告請見 [models/model_report_20260512_131827.txt](models/model_report_20260512_131827.txt#L1-L120)。

---

