# PerFedAvg FedAvg Transformer 消費者電力需求預測系統

## 項目概述

基於 **聯邦學習（Federated Learning）** 的消費者電力需求預測研究項目，使用 **Transformer 模型** 實現的時序預測。本系統採用 **Per-FedAvg（個性化聯邦平均） 算法**，支持快速個性化適應。

### 🌟 核心特色

- **🔒 隱私保護**：所有客戶端數據均保留在本地，僅進行模型參數的共享
- **🤖 個性化學習**：Per-FedAvg算法支持全局模型的快速個性化適應
- **⚡ 高效建模**：基於Transformer注意力機制的時序建模，處理25個電力特徵
- **🛡️ 智能優化**：早停機制與學習率自動調度，防止過擬合
- **📊 可視化分析**：提供模型性能評估及注意力權重的可視化工具
- **⚙️ 靈活配置**：完整的YAML配置系統，支持CUDA/MPS/CPU多設備
- **🏗️ 模塊化設計**：清晰的架構分離，便於擴展和維護


## 快速開始

### 前置需求

- Python 3.8+
- PyTorch 2.0+
- CUDA, MPS（可選，支持 GPU 加速）

### 安裝步驟

1. **克隆項目**
```bash
git clone https://github.com/panhyer36/FedAvgTransformerOPSD.git
cd FedAvgTransformerOPSD
```

2. **安裝依賴**
```bash
pip install -r requirements.txt
```

3. **檢查配置**
```bash
python config.py
```

### 開始訓練

#### 方法一：後台訓練（推薦）
```bash
# 開始訓練
./run.sh

# 監控訓練過程
tail -f logs/train_*.log

# 停止訓練
./stop.sh
```

#### 方法二：直接運行
```bash
# 訓練模型（默認使用 config.yaml）
python train.py

# 訓練模型（指定配置文件）
python train.py --config custom_config.yaml

# 測試和可視化（默認使用 config.yaml）
python test.py

# 測試和可視化（指定配置文件）
python test.py --config custom_config.yaml
```

##  項目架構

### 核心組件

```
src/
├── Server.py          #  聯邦學習協調器（模型聚合、全局訓練）
├── Client.py          #  客戶端實現（本地訓練、驗證評估）
├── Model.py           #  Transformer時序預測模型
├── DataLoader.py      #  時序數據加載器
└── Trainer.py         #  訓練器（早停、學習率調度）
```

### 數據組織

```
data/processed/
├── Consumer_01.csv              # 住宅客戶端數據
├── Consumer_02.csv              # 住宅客戶端數據
├── ...                          # Consumer_03 至 Consumer_50
├── Consumer_50.csv              # 住宅客戶端數據
└── Public_Building.csv          # 公共建築客戶端數據
```

### 📊 數據特徵詳細說明

每個CSV文件包含**25個特徵**，涵蓋電器使用、天氣和總用電量：

#### 電器用電量特徵（15個）
```
AC1, AC2, AC3, AC4           # 4個空調設備用電量
Dish washer                  # 洗碗機用電量
Washing Machine              # 洗衣機用電量  
Dryer                        # 烘乾機用電量
Water heater                 # 熱水器用電量
TV                          # 電視用電量
Microwave                   # 微波爐用電量
Kettle                      # 電熱水壺用電量
Lighting                    # 照明用電量
Refrigerator                # 冰箱用電量
Consumption_Total           # 總消費電量
Generation_Total            # 總發電量（如太陽能）
```

#### 天氣特徵（9個）
```
TemperatureC                # 溫度（攝氏度）
DewpointC                   # 露點溫度（攝氏度）
PressurehPa                 # 氣壓（百帕）
WindSpeedKMH                # 風速（公里/小時）
WindSpeedGustKMH            # 陣風風速（公里/小時）
Humidity                    # 濕度（%）
HourlyPrecipMM              # 小時降雨量（毫米）
dailyrainMM                 # 日降雨量（毫米）
SolarRadiationWatts_m2      # 太陽輻射（瓦特/平方米）
```

#### 預測目標（1個）
```
Power_Demand                # 電力需求（預測目標）
```

### 客戶端分佈
- **住宅客戶端**：50個（Consumer_01 至 Consumer_50）
- **公共建築客戶端**：1個（Public_Building）
- **總計**：51個聯邦學習客戶端

### 輸出目錄

```
📁 checkpoints/         #  模型檢查點
📁 plots/              #  可視化圖表
📁 logs/               #  訓練日誌
```

##  配置說明

### 核心配置文件：`config.yaml`

#### 聯邦學習配置
```yaml
federated_learning:
  algorithm: "per_fedavg"    # 聯邦學習算法：per_fedavg（個性化）或 fedavg（標準）
  global_rounds: 100         # 全局訓練輪數
  client_fraction: 0.1       # 每輪參與客戶端比例（10%）
  num_clients: 51            # 總客戶端數量：50個Consumer + 1個Public Building
  eval_interval: 10          # 每10輪評估一次
  
  # Per-FedAvg 專用配置
  per_fedavg:
    meta_step_size: 0.01     # 元學習步長（α），控制個性化適應速度
    use_second_order: false  # 是否使用二階梯度（HVP）：true=HVP版本，false=First-Order版本  
    hvp_damping: 0.01        # HVP阻尼係數，提高數值穩定性（僅HVP版本生效）
```

#### 本地訓練配置
```yaml
local_training:
  local_epochs: 8            # 本地訓練輪數
  learning_rate: 0.0005      # 學習率
  batch_size: 32             # 批次大小
```

#### 模型架構配置
```yaml
model:
  feature_dim: 25            # 輸入特徵維度（包含電器用電量、天氣、總用電量等）
  d_model: 256               # Transformer 模型維度
  nhead: 8                   # 多頭注意力數量
  num_layers: 4              # Transformer 層數
  output_dim: 1              # 輸出維度（Power_Demand預測）
  max_seq_length: 100        # 最大序列長度
  dropout: 0.1               # Dropout 比率
```

#### 數據配置
```yaml
data:
  data_path: "data/processed"
  input_length: 96           # 輸入序列長度（96個時間點）
  output_length: 1           # 輸出序列長度（預測1個時間點）
  train_ratio: 0.8           # 訓練集比例（80%）
  val_ratio: 0.1             # 驗證集比例（10%）
  test_ratio: 0.1            # 測試集比例（10%）
  target: ["Power_Demand"]   # 預測目標
```

#### 訓練優化配置
```yaml
training:
  early_stopping:
    patience: 15             # 早停耐心值
    min_delta: 0.001         # 最小改善閾值
```

#### 個性化測試配置
```yaml
testing:
  personalization_steps: 3   # Per-FedAvg 個性化適應步數
  support_ratio: 0.2         # Support set 比例（用於個性化適應）
  adaptation_lr: 0.001       # 個性化適應學習率
```

#### 設備配置
```yaml
device:
  type: "auto"               # 自動選擇設備：cuda（NVIDIA GPU）、mps（Apple Silicon）、cpu
```

## 🔧 使用指南

### 訓練命令詳解

| 命令 | 功能 | 說明 |
|------|------|------|
| `./run.sh` | 後台訓練 | 使用 nohup 後台運行，自動創建日誌 |
| `./stop.sh` | 停止訓練 | 停止訓練進程 |
| `python train.py` | 前台訓練 | 直接在終端運行訓練（默認 config.yaml） |
| `python train.py --config my_config.yaml` | 自定義配置訓練 | 使用指定配置文件進行訓練 |
| `python test.py` | 模型測試 | 加載模型進行測試和可視化（默認 config.yaml） |
| `python test.py --config my_config.yaml` | 自定義配置測試 | 使用指定配置文件進行測試 |

### 監控和日誌

```bash
# 實時查看訓練日誌
tail -f logs/train_*.log

# 檢查訓練狀態
ps aux | grep python

# 查看 GPU 使用情況（如果有 CUDA）
nvidia-smi
```

### 💻 設備配置

系統支持多種計算設備的自動選擇和優化：

| 設備類型 | 支持平台 | 性能 | 適用場景 |
|----------|----------|------|-----------|
| **CUDA** | NVIDIA GPU | 🚀 最高 | 大規模訓練（推薦） |
| **MPS** | Apple Silicon (M1/M2/M3) | ⚡ 高 | Mac用戶GPU加速 |
| **CPU** | 所有平台 | 🐌 基礎 | 無GPU環境備用 |

**自動設備選擇邏輯**：
```yaml
device:
  type: "auto"  # 系統自動選擇最佳可用設備
```

**手動指定設備**：
```yaml
device:
  type: "cuda"   # 強制使用NVIDIA GPU
  # type: "mps"   # 強制使用Apple Silicon GPU  
  # type: "cpu"   # 強制使用CPU
```

## 📊 測試和評估

### 🎯 雙重評估模式

系統提供**全局模型評估**和**個性化模型評估**兩種模式：

#### 全局模型評估
```bash
python test.py --config config.yaml
```
直接使用訓練好的全局模型在各客戶端測試集上評估性能。

#### 個性化模型評估（Per-FedAvg專用）
```bash
python test.py --config config.yaml --personalized
```
模擬實際部署場景，執行個性化適應後再評估性能。

### 📈 性能指標

系統會輸出以下詳細評估指標：

#### 回歸性能指標
- **MSE（均方誤差）**：模型預測精度的平方誤差
- **MAE（平均絕對誤差）**：預測偏差的絕對值平均
- **RMSE（均方根誤差）**：預測準確性的標準指標
- **R²（決定係數）**：模型對數據變異的解釋能力（0-1，越接近1越好）

#### Per-FedAvg專用指標
- **個性化改善率**：個性化後相對於全局模型的性能提升
- **適應效率**：每個適應步驟的平均性能改善
- **客戶端異質性分析**：不同客戶端的個性化需求差異

### 🎨 可視化輸出

測試完成後，在 `plots/` 目錄中會自動生成：

#### 基礎評估圖表
1. **預測對比圖**：真實值 vs 預測值散點圖（包含R²值）
2. **時序預測圖**：時間序列預測結果對比
3. **誤差分佈圖**：預測誤差的統計分佈分析
4. **注意力權重圖**：Transformer 注意力機制可視化

#### Per-FedAvg專用圖表
5. **個性化改善圖**：個性化前後性能對比
6. **客戶端異質性圖**：不同客戶端的適應效果分析
7. **適應收斂曲線**：個性化適應過程的損失變化

### 📋 測試報告

系統會生成詳細的測試報告，包含：
- 整體性能統計
- 各客戶端詳細結果
- 個性化vs全局模型對比（如適用）
- 可視化圖表路徑索引

## 🔬 技術細節

### Per-FedAvg 聯邦學習流程

**Per-FedAvg（Personalized Federated Averaging）** 是基於MAML（Model-Agnostic Meta-Learning）的個性化聯邦學習算法：

#### 📋 標準訓練流程
1. **初始化**：Server 創建全局模型並分發給客戶端
2. **元學習訓練**：各客戶端執行Per-FedAvg本地訓練（支持First-Order或HVP版本）
3. **參數聚合**：Server 使用聯邦平均算法聚合參數
4. **模型分發**：更新後的全局模型分發給所有客戶端
5. **迭代訓練**：重複步驟2-4直到收斂

#### 🎯 個性化適應流程
1. **載入全局模型**：客戶端獲得訓練好的全局模型
2. **數據分割**：將本地數據分為Support Set和Query Set
3. **快速適應**：在Support Set上執行少量梯度步驟（通常1-5步）
4. **性能評估**：在Query Set上評估個性化後的模型性能

#### ⚡ Per-FedAvg vs 標準FedAvg

| 特性 | 標準FedAvg | Per-FedAvg |
|------|------------|------------|
| 目標 | 學習單一全局模型 | 學習可快速個性化的模型 |
| 本地訓練 | 標準監督學習 | 模擬個性化適應的元學習 |
| 適應能力 | 無個性化 | 支持快速個性化適應 |
| 計算複雜度 | 低 | 中等 |
| 異質性處理 | 有限 | 優秀 |

### 🏗️ Transformer 模型架構

- **位置編碼**：正弦-餘弦位置編碼，支持時序信息理解
- **多頭注意力**：8個注意力頭並行處理，捕捉複雜時序依賴
- **模型維度**：256維Transformer內部表示空間
- **網絡深度**：4層Transformer編碼器堆疊
- **正則化策略**：0.1 Dropout率防止過擬合
- **序列處理**：支持最長100個時間步的序列

### 📊 數據處理流程

1. **數據載入**：讀取CSV格式的時序電力數據（25個特徵）
2. **時序分割**：按時間順序分割，避免數據洩漏
3. **特徵標準化**：基於訓練集統計進行Z-score標準化
4. **序列生成**：96步輸入→1步輸出的滑動窗口
5. **批次組織**：高效的DataLoader批次處理

### 🔧 個性化測試機制

**Per-FedAvg專用的個性化評估**：

```python
# 個性化適應參數
personalization_steps: 3      # 適應步數
support_ratio: 0.2           # Support set比例  
adaptation_lr: 0.001         # 適應學習率
```

**測試流程**：
1. 載入訓練好的全局模型
2. 為每個客戶端執行個性化適應
3. 在Query set上評估適應後性能
4. 生成個性化vs全局模型對比報告

## 🔍 故障排除

### 常見問題

#### 1. 內存不足
```bash
# 減少批次大小
# 修改 config.yaml
local_training:
  batch_size: 16  # 從 32 減少到 16
```

#### 2. CUDA/MPS 錯誤
```bash
# 強制使用 CPU
# 修改 config.yaml
device:
  type: "cpu"

# 檢查設備可用性
python -c "import torch; print(f'CUDA: {torch.cuda.is_available()}, MPS: {torch.backends.mps.is_available()}')"
```

#### 3. Per-FedAvg 訓練不穩定
```bash
# 調整元學習步長
# 修改 config.yaml
federated_learning:
  per_fedavg:
    meta_step_size: 0.005  # 從 0.01 降低到 0.005
    use_second_order: false  # 使用First-Order版本提高穩定性
```

#### 4. 訓練無法收斂
```bash
# 調整學習率和訓練參數
local_training:
  learning_rate: 0.0001  # 降低學習率
  local_epochs: 4        # 減少本地訓練輪次
```

#### 5. 數據加載錯誤
```bash
# 檢查數據完整性
python -c "import pandas as pd; pd.read_csv('data/processed/Consumer_01.csv').info()"
```


## 📦 項目依賴

### 核心依賴

| 套件 | 版本要求 | 功能 |
|------|----------|------|
| **torch** | ≥2.0.0 | PyTorch深度學習框架 |
| **torchvision** | ≥0.15.0 | 計算機視覺工具（依賴項） |
| **numpy** | ≥1.21.0 | 高性能數值計算 |
| **pandas** | ≥1.3.0 | 數據處理和分析 |
| **scikit-learn** | ≥1.0.0 | 機器學習工具和評估指標 |

### 可視化依賴
| 套件 | 版本要求 | 功能 |
|------|----------|------|
| **matplotlib** | ≥3.5.0 | 基礎圖表繪製 |
| **seaborn** | ≥0.11.0 | 統計圖表和美化 |

### 工具依賴
| 套件 | 版本要求 | 功能 |
|------|----------|------|
| **pyyaml** | ≥6.0 | YAML配置文件解析 |
| **psutil** | ≥5.8.0 | 系統資源監控 |
| **tensorboard** | ≥2.10.0 | 可選：訓練過程可視化 |
| **python-dotenv** | - | 環境變量管理 |
| **requests** | - | HTTP請求支持 |

### 🚀 快速安裝

#### 方法一：使用requirements.txt（推薦）
```bash
pip install -r requirements.txt
```

#### 方法二：手動安裝核心依賴
```bash
# 基礎PyTorch環境
pip install torch>=2.0.0 torchvision>=0.15.0

# 數據處理
pip install numpy>=1.21.0 pandas>=1.3.0 scikit-learn>=1.0.0

# 可視化
pip install matplotlib>=3.5.0 seaborn>=0.11.0

# 配置和工具
pip install pyyaml>=6.0 psutil>=5.8.0
```

### 💡 GPU加速設置

#### NVIDIA CUDA
```bash
# 檢查CUDA可用性
python -c "import torch; print(f'CUDA available: {torch.cuda.is_available()}')"

# 如需CUDA支持，安裝對應版本的PyTorch
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu118
```

#### Apple Silicon (MPS)
```bash
# MacOS用戶自動支持MPS，無需額外安裝
python -c "import torch; print(f'MPS available: {torch.backends.mps.is_available()}')"
```
