# Dataset: mmWave-Physio Emotion Recognition

## 🗂️ 数据集概览
- **来源文献**: [[mmEmotion]]
- **开源地址**: Zenodo (DOI: 10.5281/zenodo.15825931)
- **受试人群**: 15名健康大学生（含年龄、性别、利手性等元数据）
- **采集模态**: 
  - 非接触式：毫米波雷达 (mmWave)
  - 接触式参考：光电容积脉搏波 (PPG)、皮电反应 (GSR)
- **标注方式 (Ground Truth)**: 采用 SAM 量表的主观打分（包含 Valence效价, Arousal唤醒度, Dominance支配度，分值1-9分）

## 📁 目录结构与数据规格
整个数据集由三个核心压缩包组成，满足从原始信号研究到直接训练机器学习模型的不同需求：

### 1. 01_raw_data.zip (原始信号数据)
保留了最底层的传感器输出，适合做底层信号处理和点云生成算法的研究。
- `mmwave/`: 存储雷达原始 ADC 数据（`.mat` 格式）。其中的 `adcData` 矩阵包含12行（对应TDM复用形成的12个虚拟天线通道）。数据按 chirp 顺序排列。
- `ppg_and_gsr/`: 200Hz 采样率的生理信号（`.csv` 格式）。第一列为 PPG，第二列为 GSR。
- `self_assessment/`: 包含受试者元数据 (`Participants_metadata.csv`) 和所有视频片段的 SAM 主观打分结果 (`SAM_Pxx.csv`)。

### 2. 02_processed_data.zip (预处理数据)
作者提供的 Baseline 清洗数据，已剔除每次试验前 10 秒的注视十字准备期。
- `mmwave/`: 提供提取出的雷达生命体征。文件包含三列：融合后的原始相位、呼吸信号（0.1-0.5 Hz 滤波）、心跳信号（1.0-1.8 Hz 滤波）。
- `ppg/`: 经过 0.6-5 Hz 巴特沃斯滤波、线性去基线漂移和 0.1 秒移动平均平滑处理后的信号。
- `gsr/`: 原始电阻数据已转换为电导率，并经过 1 Hz 低通滤波。

### 3. 03_features.zip (特征数据 - 适合直接输入分类器)
作者提取好的手工特征集，直接可用于模型训练。
- **滑动窗口策略**: 考虑到情绪的演变，仅选取每个视频片段的**最后 60 秒**。使用 **5 秒无重叠滑动窗口**，因此每个片段切分为 12 个样本。
- **特征维度明细** (存储在 `featureMatrix` 中)：
  - **mmWave (32维)**: 包括生命体征的均值/方差、一阶/二阶差分、PSD频段能量、HHT分解分量，以及基于动态规划模板匹配提取的 HRV 特征（如 RMSSD, pNN50 等）。
  - **PPG (28维)**: 提取逻辑与雷达信号类似。
  - **GSR (24维)**: 提取了中位数、均值、极值、归一化范围、一阶/二阶导数以及 0-2Hz 的 PSD 特征。

## 💻 开源与复现代码
- **论文原始代码/数据相关仓库**: GitHub (`CJ20Project/mmWaveEmoRecDataset`，待进一步核验具体可用性)
- **我的复现仓库**: https://github.com/LIANGSUAI/mmemo
- **支持语言**: Python 为主，复现仓库将原论文中 MATLAB/Python 信号处理与特征提取逻辑整理为一键 pipeline。

### 复现仓库结构

```text
mmemo/
├── run_pipeline.py           # 主入口，一键运行或分步运行
├── requirements.txt          # Python 依赖
├── DEPLOY.md                 # 部署与复现说明
├── src/
│   ├── config.py             # 路径、雷达参数、窗口参数
│   ├── mmwave_processor.py   # mmWave 信号处理
│   ├── ppg_gsr_processor.py  # PPG/GSR 信号处理
│   ├── feature_utils.py      # 特征提取工具
│   └── extract_features.py   # 特征提取主脚本
└── results/                  # 本地分类结果
```

### 一键运行方式

```bash
pip install -r requirements.txt
# 修改 src/config.py 中 RAW_DATA_DIR 为 01_raw_data 路径
python -u run_pipeline.py
```

也可以分步运行：

```bash
python run_pipeline.py --step process
python run_pipeline.py --step features
python run_pipeline.py --step classify
python run_pipeline.py --step summary
```

### 复现参数

- **雷达参数**: 3 Tx、4 Rx、256 ADC samples、ADC 采样率 5.12 MHz、frame periodicity 10 ms、起始频率 60 GHz、chirp slope 66.590 THz/s。
- **窗口策略**: 取每段视频最后 60 秒；5 秒窗口；无重叠。
- **参与者与片段**: `P01-P15`，clip `00-18`，其中 `00` 为 baseline。
- **标签规则**: SAM 分数 `>5` 为 High，`<5` 为 Low，`=5` 不参与二分类。
- **分类器**: SVM RBF，GridSearchCV 调参，输出 Accuracy、Balanced Accuracy、F1。

### 复现结果

| Signal | Valence Acc | Arousal Acc | Dominance Acc |
|---|---:|---:|---:|
| PPG | 64.8% | 67.2% | 65.1% |
| mmWave | 59.5% | 67.1% | 66.4% |
| GSR | 64.9% | 67.8% | 67.8% |

与论文 Table 7 差异在 ±3% 以内，说明数据预处理、特征提取和 SVM 分类流程基本复现成功。

### 使用注意

- mmWave 原始信号处理最耗时，仓库说明约 30-60 分钟，且单个文件可能需要数 GB 内存，建议至少 16 GB 内存。
- HHT 特征依赖 `PyEMD`；若安装困难，复现代码会自动降级为带通滤波近似。
- 当前复现主要对齐论文 Table 7，不代表已经完成跨被试或跨会话的严格泛化验证。
