# 毫米波雷达数据集：mmFall (跌倒与多模态)

## 基本参数
- **全称**: mmFall Multi-Modal Fall Detection Dataset
- **雷达类型**: 调频连续波 (FMCW) 毫米波雷达 (TI IWR1443)
- **天线配置**: TDM-MIMO 模式，3个发射天线，4个接收天线
- **工作频率**: 77 GHz
- **数据模态**: 同步采集雷达原始数据 (.bin)、RD/RA 图、与视觉 RGB/深度图像。

## 数据规模
- 包含超过 60,000 帧样本数据，帧率为 20 FPS，单个样本长度约 450 帧 (22.5秒)。
- 结合 OpenPose 提取了人体骨骼关键点信息。

## 动作类别 (重点针对易混淆动作)
包含超过 24 种动作，主要分为三大类：
1. 真实跌倒 (Falls)
2. 类似跌倒的极易混淆动作 (Similar-to-fall，如快速下蹲、坐下等)
3. 常规日常动作 (Daily activities)

## 资源链接
- **GitHub 仓库**: [https://github.com/iwantlatiao/mmFall](https://github.com/iwantlatiao/mmFall)
- **数据下载托管**: [ModelScope: mmFall](https://www.modelscope.cn/datasets/iwantlatiao/mmFall)