# CICIOT2023 - XGBoost 攻擊分類

此專案基於 `CICIOT2023.ipynb`，使用 CIC-IoT-2023 資料集訓練 XGBoost 多分類模型來進行 IoT 攻擊分類。

**目錄**
- 資料來源與準備
- 執行環境
- Notebook 執行流程
- 模型與超參數說明
- 模型儲存與報告
- 調參建議

---

**資料來源與準備**

- 資料集由 notebook 中透過 `kagglehub.dataset_download("madhavmalhotra/unb-cic-iot-dataset")` 下載。
- 下載後 notebook 會尋找路徑 `wataiData/csv/CICIoT2023` 下的所有 `.csv` 檔並合併。
- 標籤會被映射為 high-level 類別（例如 `DDoS`, `Brute Force`, `Spoofing`, `DoS`, `Recon`, `Web-based`, `Mirai`, `Benign`），未映射之資料會被移除。
- 資料清洗採「溫和策略」：將 inf 替換為 NaN、刪除缺失比例超過 50% 的列、其餘以中位數填補數值欄位。

**資料切割**
- 切法：先以 `train_test_split` 切出 70% 訓練集與 30% 暫存，之後再把暫存分為測試/驗證，最終比例如下：
  - 訓練集：70%
  - 測試集：20%
  - 驗證集：10%

---

**執行環境**

建議建立 virtualenv 或 conda 環境後安裝以下套件：

```bash
pip install pandas numpy scikit-learn xgboost matplotlib seaborn joblib kagglehub
```

Notebook 檔案： `CICIOT2023.ipynb`。

---

**Notebook 執行流程（簡要）**

1. 下載並合併 CICIoT2023 的 CSV 檔。
2. 對原始細分類標籤做 mapping 成 `label_category`（高階類別）。
3. 資料清洗：處理 inf、刪除缺失過多的列、以中位數填補缺失值。
4. 分離特徵與標籤，使用 `LabelEncoder` 編碼標籤。
5. 切分資料為訓練/測試/驗證。
6. 建立 `xgboost.XGBClassifier` 並以驗證集做評估。
7. 輸出混淆矩陣、特徵重要性；將模型與報告儲存至 `models/` 資料夾。

---

**模型超參數（來自 notebook）**

Notebook 中使用的 XGBoost 參數如下（`xgb_params`）：

```python
xgb_params = {
    'max_depth': 10,
    'learning_rate': 0.1,
    'n_estimators': 100,
    'objective': 'multi:softmax',
    'num_class': len(le.classes_),
    'n_jobs': -1,
    'random_state': 42,
    'eval_metric': 'mlogloss',
    'subsample': 0.8,
    'colsample_bytree': 0.8,
    'verbosity': 1
}
```

參數說明與使用建議：

- `max_depth`：決定每棵樹允許的最大深度。較大值能擬合較複雜的關係，但易過擬合。Notebook 使用 `10`，常見範圍為 `3-12`。
- `learning_rate`：梯度下降步長（又稱 eta）。較小的學習率需要更多樹（`n_estimators`）才會收斂。Notebook 使用 `0.1`，建議範圍 `0.01-0.3`。
- `n_estimators`：樹的數量（弱學習器數量）。Notebook 使用 `100`。若降低 `learning_rate`，請增加此值（例如 `200-1000`）。
- `objective`：任務目標；此為多分類 `multi:softmax`（直接輸出類別）。若需要機率分布可改為 `multi:softprob`。
- `num_class`：類別數，須與標籤數一致。
- `n_jobs`：CPU 核心數，`-1` 表示使用所有可用核心。
- `random_state`：隨機種子以利結果重現。
- `eval_metric`：驗證時使用的評估指標，`mlogloss` 為多分類對數損失。
- `subsample`：列取樣比例（row sampling），降低過擬合；Notebook 使用 `0.8`，常見 `0.5-1.0`。
- `colsample_bytree`：每棵樹特徵採樣比例（feature sampling），Notebook 使用 `0.8`，常見 `0.3-1.0`。
- `verbosity`：輸出詳細程度，0=靜默，1=一般。

額外可考慮的正則化參數（Notebook 未設定，但建議在調參時嘗試）：
- `gamma`：分裂節點需要的最小損失減少（控制樹的複雜度）。
- `reg_alpha`（L1）與 `reg_lambda`（L2）：權重正則化，幫助降低過擬合。

如何在 notebook 中更改參數：編輯 `xgb_params` 字典後重新執行訓練區塊（第 6 部分）。

---

**模型儲存與報告**

- 儲存函式：`save_complete_xgboost_model(...)`，會將以下內容打包並以 `joblib` 儲存到 `models/`：模型物件、`LabelEncoder`、特徵名稱、驗證集快照、config、訓練資訊、性能報告、特徵重要性與 timestamp。
- 儲存檔案範例：`models/xgboost_complete_YYYYMMDD_HHMMSS.joblib` 與 `models/xgboost_report_YYYYMMDD_HHMMSS.txt`。

要載入儲存的模型：

```python
import joblib
pkg = joblib.load('models/xgboost_complete_20260512_115134.joblib')
model = pkg['model']
le = pkg['label_encoder']
feature_names = pkg['feature_names']
```

---
