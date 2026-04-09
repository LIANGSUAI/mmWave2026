# 毫米波雷达数据集：CI4R

## 基本参数
- **全称**: Computational Intelligence for Radar Dataset
- **雷达类型**: 调频连续波 (FMCW) 雷达 (TI IWR1443 BOOST)
- **工作频率**: 77 GHz
- **带宽**: 750 MHz (论文设定，官方硬件支持更宽)
- **场景特征**: 非连续动作，包含大量的精细步态异常特征。
- **样本时长**: 20 秒

## 数据规模
- 6 名参与者（覆盖不同年龄、身高和体重）。
- 每人每类动作执行 10 次重复，微多普勒特征图分辨率预处理为 224x224。

## 动作类别 (11类)
1. 向雷达行走 / 背对雷达行走
2. 从地上捡物
3. 弯腰
4. 坐在椅子上 / 跪下 / 向雷达爬行
5. 特殊步态：右腿僵硬跛行 / 踮脚走 / 小碎步走 / 剪刀步态

## 资源链接
- **GitHub 仓库**: [https://github.com/ci4r/CI4R-Activity-Recognition-datasets](https://github.com/ci4r/CI4R-Activity-Recognition-datasets)