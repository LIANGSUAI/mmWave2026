# 毫米波雷达数据集：mmWave-3DPCHM-1.0

## 基本参数

- **全称:** mmWave-3DPCHM-1.0: A 3D Point Cloud Dataset for Human Actions Using Millimeter-Wave Radar
- **中文名:** 基于毫米波雷达三维点云的人体动作数据集
- **数据集 DOI:** https://doi.org/10.57760/sciencedb.09520
- **版本:** V4
- **发布平台:** Science Data Bank
- **引用日期:** 2026-05-12
- **关联论文:** JIN Biao, SUN Kangsheng, WU Hao, et al. 3D point cloud from millimeter-wave radar for human action recognition: dataset and method. Journal of Radars, 2025, 14(1): 73-89.
- **论文 DOI:** https://doi.org/10.12000/JR24195
- **数据类型:** 毫米波雷达 3D 点云人体动作数据集。

## 采集设备

该数据集使用两类毫米波雷达传感器采集：

- **TI IWR1443-ISK**
- **Vayyar vBlu RF imaging module**

这点很重要：它本身就包含不同雷达设备来源，适合观察设备差异对点云分布和动作识别的影响。

## 采集场景

- **场地:** 会议室。
- **雷达安装:** 雷达安装在墙面，高度约 1.75 m。
- **动作区域:** 志愿者在矩形区域内完成动作。
- **距离:** 动作区域与雷达水平距离约 1.2 m。
- **区域大小:** 约 2 m × 2 m。

## 数据规模

- **志愿者:** 7 名。
- **动作类别:** 12 类。
- **每类动作采集:** 每个动作 3 组数据。
- **每组时长:** 3 分钟。
- **采集方式:** 志愿者在采集过程中连续完成指定动作。

## 动作类别

共 12 类，包括 3 类静态动作和 9 类动态动作：

### 静态动作

- falling / fall，跌倒
- sitting still，静坐
- standing，站立

### 动态动作

- punching，出拳
- jumping，跳跃
- waving left hand，挥左手
- leaning forward to the left，向左前倾
- opening arms，张开双臂
- waving right hand，挥右手
- leaning forward to the right，向右前倾
- squatting，下蹲
- walking，行走

## 数据内容

数据集包含：

- 原始毫米波雷达点云数据。
- 预处理后的点云数据。
- 点云处理代码。
- 预定义动作对应的图像和视频材料。
- 数据以 Excel 文件形式保存。

其中，TI 雷达数据还提供了多帧融合、聚类等预处理结果和代码，方便直接做点云动作识别实验。

## 关联方法

关联论文提出了 PETer（Point EdgeConv and Transformer）网络：

- 对每帧 3D 点云构建局部有向邻域图。
- 用 EdgeConv 提取空间几何特征。
- 用 Transformer 建模多帧点云之间的时间关系。
- 论文报告 PETer 在 TI 数据上达到约 98.77%，在 Vayyar 数据上达到约 99.51%。
- 模型大小约 1.09 MB，强调边缘部署友好。

## 与其他数据集对比

- **相对 [[RadHAR_dataset]]:** 3DPCHM 动作类别更多，且包含 TI 与 Vayyar 两类雷达设备；RadHAR 更经典但规模和类别更小。
- **相对 [[MiliPoint]]:** MiliPoint 更大、任务更多；3DPCHM 的优势是包含双设备点云和配套中文数据论文。
- **相对 [[M4Human]]:** M4Human 面向 mesh-level 人体重建；3DPCHM 更适合点云 HAR 和轻量模型验证。
- **相对 [[DAP-Net]] 的 UniMM-HAR:** 3DPCHM 也有多设备特性，但动作、采集协议、格式和标签体系需要手动对齐后才能作为跨源测试集使用。

## 适合的研究问题

- 毫米波点云动作识别。
- TI 与 Vayyar 两类雷达设备下的跨设备泛化。
- 点云稀疏性、噪声和多帧融合策略分析。
- Doppler / intensity / xyz 特征在动作识别中的作用。
- 轻量模型在边缘设备上的部署可行性。

## 复现与使用注意

- 如果要接入 [[DAP-Net]]，需要把数据转换成 DAP 期望的格式：`[T, P, C] = [32, 64, 5]`，其中 `C = x, y, z, Doppler, intensity`。
- 需要确认 Excel 文件中的字段名、坐标系、Doppler/速度字段、强度字段是否与 DAP-Net 的预处理脚本一致。
- 需要手动做动作标签映射，因为 3DPCHM 的 12 类动作与 RadHAR、mRI、MM-Fi 或 UniMM-HAR 的 33 类动作不完全一致。
- 若使用 RadHAR 训练、3DPCHM 测试，应只保留共有或可映射动作，例如 walking、jumping、squatting、waving、standing/falling 等，具体映射需逐类核验。

## 对我的价值

- 这是一个适合做“跨数据集泛化测试”的外部点云数据集。
- 可以作为 RadHAR 复现之后的下一步：先跑同数据集训练/测试，再尝试 RadHAR -> 3DPCHM 的 cross-dataset transfer。
- 如果配合 DAP-Net，可以测试 Doppler-aware 策略在非 UniMM-HAR 数据源上的泛化能力。

## 资源链接

- **Science Data Bank DOI:** https://doi.org/10.57760/sciencedb.09520
- **数据论文 DOI:** https://doi.org/10.12000/JR24195
- **数据集检索页:** https://www.scidb.cn/detail?dataSetId=4c0cbfe8349e4a0ca19a73dc165be658

