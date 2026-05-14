# An emotion recognition dataset using millimeter wave radar and physiological reference signals

## 📝 元数据
- **作者**: Jialong Cai, Xinyan Zhang, Yang Pan, Hui Zhou
- **期刊/会议**: Scientific Data (2026)
- **DOI**: 10.1038/s41597-026-07159-6
- **复现仓库**: https://github.com/LIANGSUAI/mmemo
- **标签**: #毫米波雷达 #情绪识别 #数据集 #FMCW #信号处理

## 🎯 核心贡献
- 发布了首个基于 **毫米波雷达 (mmWave)** 和生理参考信号 (PPG, GSR) 的情绪识别公开数据集。
- 证明了通过非接触式的 FMCW 雷达提取心跳和呼吸所构建的特征，在情绪二分类任务（Valence, Arousal, Dominance）上能够达到与接触式设备（如 DEAP 数据集）相当的基准线准确率 (60%-70%)。

## 🛠️ 毫米波信号处理 Pipeline
1. **原始数据获取**: TI IWR6843ISK-ODS (60-64 GHz), 12个虚拟通道。
2. **Range-FFT**: 提取目标距离。
3. **MTI (Moving Target Indication)**: 使用均值相消法抑制静态杂波。
4. **多距离单元相位融合**: 选取能量最高的5个 range bins 进行加权相位展开 (Phase Unwrapping)，提升鲁棒性。
5. **滤波分离**: 0.1-0.5 Hz (呼吸) / 1.0-1.8 Hz (心跳)。
6. **特征提取**: 时频域特征、Hilbert-Huang 变换 (HHT) 分解、以及 HRV 相关特征。

## 🔁 复现记录

- **复现项目**: [LIANGSUAI/mmemo](https://github.com/LIANGSUAI/mmemo)。
- **复现范围**: 从原始数据到分类结果的完整 pipeline，包括 mmWave、PPG、GSR 三类信号。
- **入口脚本**: `run_pipeline.py`，支持 `process`、`features`、`classify`、`summary` 四个步骤。
- **环境**: Python 3.10+，主要依赖 `numpy`、`scipy`、`scikit-learn`、`pandas`，`PyEMD` 用于 HHT，安装困难时可降级近似。

### 复现流程

| 步骤 | 内容 | 说明 |
|---|---|---|
| 信号处理 | mmWave / PPG / GSR preprocessing | mmWave 最耗时，需要对每个 chirp 做 FFT，建议服务器运行 |
| 特征提取 | 生成 `.mat` feature matrix | mmWave 32 维，PPG 28 维，GSR 24 维 |
| 情绪分类 | SVM RBF | SAM 评分二分类，`>5=High`，`<5=Low`，`=5` 跳过 |
| 结果汇总 | Accuracy / Balanced Accuracy / F1 | 输出到 `results/resultsTable.txt` |

### 复现结果

| Signal | Valence Acc | Arousal Acc | Dominance Acc |
|---|---:|---:|---:|
| PPG | 64.8% | 67.2% | 65.1% |
| mmWave | 59.5% | 67.1% | 66.4% |
| GSR | 64.9% | 67.8% | 67.8% |

与论文 Table 7 的报告结果差异在 ±3% 以内，说明复现 pipeline 基本对齐原论文。mmWave 在 Arousal 和 Dominance 上接近 PPG/GSR，但 Valence 略低，说明非接触雷达生命体征对部分情绪维度有效，但信息量仍受限于心跳/呼吸特征。

### 复现细节

- **mmWave 处理**: Range-FFT → MTI 杂波抑制 → 多 bin 融合 `K=5` → 多通道平均 → 一阶差分 → 呼吸/心跳带通滤波。
- **PPG 处理**: 4 阶 Butterworth 带通滤波 `0.6-5 Hz` → 线性去趋势 → 20 点移动平均平滑。
- **GSR 处理**: 电阻转电导 → 3 阶 Butterworth 低通滤波 `1 Hz`。
- **分类设置**: 每个视频内部前 75% 窗口训练、后 25% 窗口测试；Z-score 标准化；GridSearchCV 搜索 SVM 的 `C` 和 `gamma`。
- **注意点**: 该划分方式沿用复现仓库实现，适合复现 Table 7，但不等价于严格跨被试泛化评估。

## 💡 启发与思考
- 论文仅使用了经典的 SVM 进行基线测试，特征提取主要依赖传统频段滤波和统计特征。这为引入基于深度学习的轻量化序列模型（如结合点云特征或时序特征的 Mamba/Transformer 架构）留下了极大的提升空间。
- 复现结果说明，传统手工特征 + SVM 已经能达到 60%-70% 左右准确率；后续如果改进模型，应该重点比较是否真正提升跨被试、跨片段和跨情绪维度的泛化，而不是只提高同一划分下的 Accuracy。
- mmWave 情绪识别的关键瓶颈不一定是分类器，而可能在生命体征提取质量、运动伪影抑制、个体差异校正和情绪标签噪声。
