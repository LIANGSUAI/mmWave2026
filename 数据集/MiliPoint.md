# 毫米波雷达数据集：MiliPoint

## 基本参数

- **全称:** [[文献笔记/MiliPoint|MiliPoint: A Point Cloud Dataset for mmWave Radar]]
- **数据集定位:** 大规模开放式毫米波雷达点云数据集，用于 human activity recognition 相关任务。
- **论文来源:** NeurIPS 2023 Datasets and Benchmarks Track。
- **作者:** Han Cui, Shu Zhong, Jiacheng Wu, Zichao Shen, Naim Dahnoun, Yiren Zhao。
- **发表时间:** 2023-09-23 arXiv；NeurIPS 2023 正式收录。
- **arXiv:** 2309.13425。
- **DOI:** https://doi.org/10.48550/arXiv.2309.13425；Proceedings DOI: https://doi.org/10.52202/075280-2739。
- **硬件:** 文献索引中提到使用 TI IWR1843 mmWave radar。
- **数据格式:** mmWave radar point cloud，通常包含 3D 坐标、Doppler velocity、intensity/SNR 等点级属性，具体字段需以数据发布说明为准。

## 数据规模

- **总帧数:** 约 545K frames。
- **受试者:** 11 subjects。
- **动作类别:** 49 activities。
- **任务覆盖:** identification、action classification、keypoint estimation 三类任务。

MiliPoint 的重要性在于：它不是只给一个动作分类标签，而是把“身份识别、动作分类、关键点估计”三个层次放在同一个 radar point cloud benchmark 中，方便比较不同点云网络到底学到了什么。

## 数据任务

### 1. Identification

识别不同受试者身份。这个任务可以测试毫米波点云中是否包含稳定的个体运动模式或体态特征。

### 2. Action Classification

对 49 类人体动作进行分类，是最直接的 HAR benchmark 任务。

### 3. Keypoint Estimation

从稀疏雷达点云估计人体关键点，比动作分类更细粒度，也更接近 pose estimation / skeleton reconstruction。

## Baseline 方法

论文建立了一组 point-based deep neural network baseline，包括：

- DGCNN
- PointNet++
- PointTransformer

这些 baseline 很适合和 [[Star Graph + DDGNN]]、[[RadHAR]]、后续 Mamba/GNN 方案进行横向比较。

## 适合的研究问题

- 稀疏、噪声大的毫米波点云是否能直接用点云网络建模？
- 传统视觉点云方法迁移到 radar point cloud 时，瓶颈在哪里？
- 增加时间堆叠帧数是否能提升 keypoint estimation 和 action classification？
- 基于图结构的方法是否比 PointNet/PointTransformer 更适合雷达稀疏点云？

## 可用性与限制

- **优势:** NeurIPS Datasets and Benchmarks 收录，公开性和基准价值较高。
- **限制:** 相比 [[M4Human]]，MiliPoint 更偏 point cloud 与 HAR，不直接提供 mesh-level 标注。
- **需要核验:** 数据下载入口、具体字段格式、训练/验证/测试划分和 license。

## 与其他数据集对比

- **相对 [[RadHAR_dataset]]:** MiliPoint 规模更大、动作更丰富，并支持关键点估计。
- **相对 [[M4Human]]:** MiliPoint 更早、更偏点云和 HAR；M4Human 更新、更偏多模态 HMR 与 mesh-level 标注。
- **相对 [[mmRehab]]:** MiliPoint 是通用 HAR/关键点 benchmark；mmRehab 更强调康复姿态、深度图对齐和几何先验。

## 对我的课题的价值

- 如果研究“密度分析”，MiliPoint 的稀疏点云非常适合作为密度分布、点云稀疏性、异常点过滤和图结构构建的案例。
- 如果做可解释模型，可以比较不同动作在点云密度、Doppler 分布、空间扩展范围上的差异。
- 可以作为 [[Star Graph + DDGNN]] 或后续 Graph-Mamba 类方法的候选评测数据集。

## 资源链接

- **NeurIPS 页面:** https://papers.nips.cc/paper_files/paper/2023/hash/c60468eca9cd0b0083f0ff9d0aeb171a-Abstract-Datasets_and_Benchmarks.html
- **OpenReview:** https://openreview.net/forum?id=FpK2aQfbyo
- **arXiv:** https://arxiv.org/abs/2309.13425
- **UCL Discovery:** https://discovery.ucl.ac.uk/id/eprint/10192698
- **Papers with Code:** https://paperswithcode.com/paper/milipoint-a-point-cloud-dataset-for-mmwave

