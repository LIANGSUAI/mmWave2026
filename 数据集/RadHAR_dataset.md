# 毫米波雷达数据集：RadHAR (3D点云)

## 基本参数
- **全称**: Radar-based Human Activity Recognition (Point Cloud Dataset)
- **数据格式**: 3D 稀疏点云 (3D sparse point clouds)，脱离了传统的二维微多普勒频谱限制。
- **雷达类型**: 毫米波雷达 (常用 77 GHz 设备提取点云)
- **场景特征**: 基于空间位置的室内宏观运动监控，具有极高的隐私保护特性。

## 数据规模
- 整体包含约 167,000 帧点云数据。
- 录制有效监控时长达 5566 秒。

## 动作类别 (5类健身与宏观运动)
1. Walking (行走)
2. Jumping (跳跃)
3. Jumping Jacks (开合跳)
4. Squats (深蹲)
5. Boxing (拳击)

## 资源链接
- **研究背景**: 此数据集常用于验证 PointNet 等三维深度学习网络直接在雷达空间点云上进行分类的有效性。