# An emotion recognition dataset using millimeter wave radar and physiological reference signals

## 📝 元数据
- **作者**: Jialong Cai, Xinyan Zhang, Yang Pan, Hui Zhou
- **期刊/会议**: Scientific Data (2026)
- **DOI**: 10.1038/s41597-026-07159-6
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

## 💡 启发与思考
- 论文仅使用了经典的 SVM 进行基线测试，特征提取主要依赖传统频段滤波和统计特征。这为引入基于深度学习的轻量化序列模型（如结合点云特征或时序特征的 Mamba/Transformer 架构）留下了极大的提升空间。