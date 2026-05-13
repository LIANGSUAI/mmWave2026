#毫米波 #点云 #数据集 #GNN #HAR

### 📇 元数据 (Metadata)

- **论文全称：** MiliPoint: A Point Cloud Dataset for mmWave Radar
- **标题翻译：** MiliPoint：面向毫米波雷达的点云数据集
- **作者：** 待核验
- **期刊/会议：** 待核验（arXiv:2307.07225）
- **发表年份：** 2023
- **DOI/ArXiv：** 2307.07225
- **相关代码/数据集：** 待核验（预计开源数据集）

### 一句话总结

发布了一个大规模毫米波雷达点云数据集，并提供了包括 GNN 在内的多种 baseline 方法 benchmark，为雷达点云动作识别研究提供了标准化评测平台。

### 研究痛点与动机

- **现有研究的局限：** 雷达点云 HAR 领域缺乏大规模、标准化的公开数据集。现有数据集（如 [[RadHAR_dataset]]）规模较小、动作类别有限，无法充分训练深度学习模型，也无法公平对比不同方法。
- **痛点：** 各研究团队使用私有数据集，方法间无法横向比较；数据规模不足导致模型泛化能力差。
- **切入点：** 构建大规模、多样化的雷达点云数据集，配套多种 baseline（包括 PointNet、GNN 等），推动该领域的标准化研究。

### 核心创新与方法论

- **大规模数据集：** 覆盖更多受试者、更多动作类别、更多场景的毫米波雷达点云数据。（具体规模待核验）
- **标准化评测：** 提供统一的数据划分、评测协议和 baseline 模型。
- **GNN Baseline：** 提供基于图神经网络的 baseline 方法，将雷达点云建模为图结构进行分类。
  - 与 [[Star Graph + DDGNN]] 的关系：MiliPoint 可能作为 Star Graph 的评测数据集之一，或者 Star Graph 是在 MiliPoint 上的一个竞争方法。

### 输入/输出 (I/O)

- **Input：** 毫米波雷达 3D 点云序列（每帧包含 x, y, z, 速度, 强度等特征）。
- **Baseline 方法：** PointNet, GNN (GCN/GAT), 3D CNN 等。
- **Output：** 动作类别分类。

### 实验与结果

- **待核验：** 具体的 baseline 结果对比需查阅原文。

### 对我的课题的帮助

- **数据集选择：** MiliPoint 可以作为毕设的补充数据集，用于验证模型的泛化能力。与 [[DIAT]], [[CI4R]], [[UoG20]] 等已有数据集形成互补。
- **GNN Baseline 对比：** 提供的 GNN baseline 结果可以直接与 [[Star Graph + DDGNN]] 的星型图方法进行对比分析。
- **标准化评测：** 使用统一的评测协议有助于与其他方法公平比较。

### 与已有笔记的关联

- [[Star Graph + DDGNN]]：直接相关的图网络方法，MiliPoint 可能是其评测数据集
- [[RadHAR]]：同为点云 HAR，但 MiliPoint 规模更大
- [[RadHAR_dataset]]：MiliPoint 是其扩展/替代版本

### 后续要读的论文

- 原文的数据集统计信息和 baseline 结果
- 在 MiliPoint 上评测的其他 SOTA 方法
