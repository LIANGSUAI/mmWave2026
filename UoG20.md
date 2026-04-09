# 毫米波雷达数据集：UoG20 (连续动作)

## 基本参数
- **全称**: University of Glasgow 2020 Radar Dataset
- **雷达类型**: 调频连续波 (FMCW) 雷达
- **工作频率**: 5.8 GHz
- **带宽**: 400 MHz
- **场景特征**: **连续 (Continuous) 动作序列**。参与者自主决定动作间的切换和轨迹，动作之间没有明显的中断，极具挑战性。
- **样本时长**: 35 秒

## 数据规模
- 15 名参与者（14名男性，1名女性，年龄 21-35 岁）。
- 每帧特征图分辨率约为 240x1471。

## 动作类别 (6类连续触发)
1. Pick (捡物)
2. Drink (喝水)
3. Walk (行走)
4. Sit (坐下)
5. Stand (站立)
6. Fall (跌倒)

## 资源链接
- **相关论文**: *Continuous human activity classification from FMCW radar with bi-LSTM networks* (IEEE Sensors J., 2020)