#数据集
# 毫米波雷达数据集：DIAT-μRadHAR

## 基本参数
- **全称**: Defense Institute of Advanced Technology μRadHAR Dataset
- **雷达类型**: 连续波 (CW) 雷达
- **工作频率**: 10 GHz (X-band)
- **场景特征**: 非连续 (Non-continuous) 动作，主要针对安防与可疑行为监控。
- **样本时长**: 3 秒

## 数据规模
- 共 30 名受试者，在 10m 到 0.5km 范围内采集。
- 包含 3780 个微多普勒频谱样本 (预处理形状为 3x224x224)。

## 动作类别 (6类)
1. Army crawling (匍匐前进)
2. Boxing (拳击)
3. Jumping while holding a gun (持枪跳跃)
4. Army jogging (武装慢跑)
5. Army marching (武装行军)
6. Stone-pelting/grenade-throwing (投掷石块/手榴弹)

## 资源链接
- **GitHub 仓库**: [https://github.com/mainak15/DIAT--RadHAR-Dataset](https://github.com/mainak15/DIAT--RadHAR-Dataset)
- **关联论文**: *DIAT-RadHARNet: A Lightweight DCNN for Radar based Classification of Human Suspicious Activities*