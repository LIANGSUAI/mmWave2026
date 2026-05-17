#毫米波 #点云 #HAR #Doppler #跨源泛化 #异构雷达 #UniMM-HAR #arXiv2026 #重点论文

### 📇 元数据 (Metadata)

- **论文全称：** DAP: Doppler-aware Point Network for Heterogeneous mmWave Action Recognition
- **标题翻译：** DAP：面向异构毫米波动作识别的 Doppler 感知点云网络
- **作者：** Jiaying Lin, Shiman Wu, Jinfu Liu, Can Wang, Mengyuan Liu
- **机构：** Peking University, Huazhong University of Science and Technology, DJI Technology Company Ltd., Christian-Albrechts-Universität zu Kiel
- **期刊/会议：** arXiv preprint
- **发表时间：** 2026-05-10
- **arXiv：** 2605.09604
- **代码/数据集：** https://github.com/jolin830/DAP-Net
- **阅读状态：** 初读笔记，值得后续结合 [[Star Graph + DDGNN]]、[[MiliPoint]] 精读方法和数据集部分

### 一句话总结

这篇论文提出 UniMM-HAR 异构多源毫米波点云 HAR 数据集，并设计 DAP-Net，用 Doppler 作为跨设备相对稳定的运动先验，通过几何增密、特征重校准和文本语义对齐提升不同雷达设备/频段之间的动作识别泛化能力。

### 研究痛点与动机

现有毫米波点云 HAR 数据集大多是 homogeneous single-source setting，即同一个设备、同一个频段、同一套采集流程。这样的模型容易学到数据源特有的统计模式，而不是动作本身。

现实部署中常见异构来源：

- 不同雷达型号：TI IWR1443 vs TI IWR6843。
- 不同频段：76-81 GHz vs 60-64 GHz。
- 不同帧率：10 Hz vs 30 Hz。
- 不同安装高度、距离和采集场景。

这些差异会导致：

- 点云密度不同。
- 空间覆盖范围不同。
- Doppler 分布不同。
- intensity / noise / outlier 统计不同。

所以问题不再只是“如何识别动作”，而是“如何识别跨设备、跨频段、跨数据源仍然稳定的动作语义”。

### UniMM-HAR 数据集

UniMM-HAR 是本文的重要贡献之一。它不是从零采集，而是统一整理三个公开毫米波点云数据集：

- [[RadHAR]] / [[RadHAR_dataset]]：TI IWR1443，76-81 GHz，5 类动作，2 名受试者。
- mRI：TI IWR1443，76-81 GHz，12 类动作，20 名受试者。
- MM-Fi：TI IWR6843，60-64 GHz，27 类动作，40 名受试者。

统一后规模：

- **动作类别：** 33 类，包括 21 类日常动作和 12 类康复动作。
- **受试者：** 62 subjects。
- **序列数：** 40,494 sequences。
- **帧数：** 约 1.29M frames。
- **输入标准化：** `[T, P, C] = [32, 64, 5]`。
- **通道：** `x, y, z, Doppler, intensity`。

### 数据集处理与协议

**Action Alignment**

- 将语义相同的动作合并，例如 RadHAR 的 squatting、mRI 的 squat、MM-Fi 的 Squat 统一为 `squat`。
- 对细粒度差异明显的动作保留区分，例如 `extend both limbs` 和 `extend both upper limbs` 不强行合并。

**Dataset-Aware Preprocessing**

- RadHAR：sliding window，window size 60，stride 10。
- mRI：sliding window，window size 32，stride 16。
- MM-Fi：根据 segmentation files 切成 action clips。

**Representation Standardization**

- 时间维度超过 32 帧则 uniform downsampling，不足则 zero-padding。
- 每帧超过 64 点则 FPS，不足则 repeat sampling。
- 提供 CSV 和 NPZ 两种格式：CSV 保留溯源信息，NPZ 方便模型直接输入。

**Evaluation Protocols**

- Random Split：所有 source 按 60:40 划分。
- Cross-Subject (C-Sub)：mRI + MM-Fi，按 subject 5:5 划分。
- Cross-Set (C-Set)：RadHAR + mRI + MM-Fi，按 scene 4:2 划分。

### 方法概述：DAP-Net

DAP-Net 由三个关键部分组成：

1. **D2R：Dual-space Doppler Reparameterization**
2. **Point Cloud Backbone**
3. **TAM：Text Alignment Module**

整体流程：

```text
mmWave point cloud sequence
  -> D2R: Doppler-guided densification + feature recalibration
  -> point cloud backbone, e.g. PointMLP
  -> TAM: text semantic alignment
  -> action classification
```

### 核心模块 1：D2R

D2R 的目标是用 Doppler 作为 motion prior，减少异构雷达源带来的 source-specific bias。

D2R 包含两个子模块：

- **DGR：Doppler-guided Geometry Reparameterization**
- **MFR：Motion-aware Feature Recalibration**

#### DGR：Doppler 引导的几何重参数化

毫米波点云稀疏且噪声多。直接重复采样会把噪声也放大，固定窗口聚合又对帧率和点云密度敏感。DGR 的做法是：

- 根据每帧点的 Doppler magnitude 学习一个相对运动阈值。
- 把点分成 fast / slow / raw 三个分支。
- 对 motion-salient 的 fast points 做重复增密。
- slow points 保留原始结构。
- raw branch 随机补齐，保证最终点数达到 `Pgoal`。

关键设计：

- **DSQ：Doppler-sorted Soft Quantile**
  - 不用固定绝对阈值，而是学习相对分位点。
  - 更适合不同雷达设备 Doppler 绝对尺度不一致的情况。
- **TMPD：Tri-branch Motion-Aware Point Densification**
  - fast branch 强化运动关键区域。
  - slow branch 保留低速/准静态区域。
  - raw branch 保持原始分布补充。

#### MFR：运动感知特征重校准

MFR 使用 fast points 的特征作为 motion summary vector，生成通道级 scale 和 shift：

```text
F = gamma(c) * F_out + beta(c)
```

它类似 FiLM，用运动显著点指导整体特征重校准，让网络更关注和动作相关的动态模式。

### 核心模块 2：TAM

TAM 使用文本语义作为稳定锚点，把毫米波全局特征对齐到预训练文本空间。

- 文本 prompt 示例：`a mmWave point cloud of a person [CLS]`
- 文本编码器：论文实验用 frozen CLIP，也测试了 SBERT。
- 模型计算 mmWave feature 与 action text embedding 的相似度，再与 backbone classifier logits 融合。

我的理解：

- TAM 的动机是让模型不要只依赖某个 radar source 的统计分布，而是向动作语义靠拢。
- 但这个模块需要重点看消融，因为它有一点“借 CLIP/text alignment 做语义锚点”的流行味道。论文结果显示提升约 0.7%，主要增益还是 D2R。

### 实验结果

**SOTA 对比**

| Model | C-Sub | C-Set |
|---|---:|---:|
| PointNet | 59.27 | 70.96 |
| DGCNN | 71.70 | 52.18 |
| RadHAR | 42.49 | 48.47 |
| PointMLP | 71.06 | 78.13 |
| PST-Transformer | 63.13 | 79.70 |
| FastHAR | 53.72 | 61.16 |
| 3DInAction | 73.43 | 57.81 |
| UST-SSM | 71.50 | 48.40 |
| **DAP-Net** | **80.72** | **81.82** |

DAP-Net 在 C-Sub 和 C-Set 都最高，尤其说明它在异构跨源场景下比普通点云 backbone 和已有毫米波 HAR 方法更稳。

### 消融结果

**模块消融**

| Backbone | Module | Acc |
|---|---|---:|
| PointMLP | baseline | 71.06 |
| PointMLP | +D2R | 80.00 |
| PointMLP | +D2R+TAM | 80.72 |
| PST-Transformer | baseline | 63.13 |
| PST-Transformer | +D2R | 76.73 |
| PST-Transformer | +D2R+TAM | 78.21 |

主要结论：

- D2R 是核心增益来源。
- TAM 有增益，但幅度较小。
- D2R 对不同 backbone 都有效，说明它是 plug-and-play。

**D2R 子模块**

- DGR 和 MFR 单独都有提升。
- DGR + MFR 组合最好。
- DGR 参数量几乎为 0，MFR 只增加约 0.06M 参数。

**超参数**

- fast branch 的重复增密因子 `r=5` 最好。
- `Pgoal=1024` 最好；512 不够，2048 会引入冗余/噪声。

### 论文里对 Doppler 的核心判断

Doppler 只测量雷达视线方向的径向速度，不等于完整 3D 速度。但它仍然有两个价值：

- **动作区分性：** 不同动作有不同速度幅度、周期性和方向变化。
- **跨源一致性：** 不同设备的 Doppler 绝对值可能不同，但同一动作的相对时序变化模式更稳定。

所以 Doppler 可以作为动作语义的运动锚点，帮助模型从设备差异中抽离出动作本身。

### 与已有笔记的关联

- [[Star Graph + DDGNN]]：Star Graph 解决稀疏点云和变长输入，DAP 进一步解决异构雷达源和 Doppler 利用问题。
- [[MiliPoint]]：MiliPoint 是单源点云 benchmark；UniMM-HAR 更强调多源异构、跨设备/跨频段泛化。
- [[RadHAR]]：UniMM-HAR 整合了 RadHAR，并把它放入更复杂的多源 benchmark。
- [[RadMamba]]：RadMamba 走微多普勒时频图和高效序列建模路线；DAP 走点云 + Doppler 运动先验路线。

### 对我现在阶段的价值

- 如果继续看毫米波点云 HAR，DAP 是很好的下一篇：它不是简单追求某个数据集准确率，而是引入了真实部署中的 cross-source generalization 问题。
- 它能帮助我从“模型结构”转向“数据分布和设备差异”的视角。
- UniMM-HAR 值得单独关注，后续如果做点云/图网络/密度分析相关实验，可以作为更现实的 benchmark。

### 批判性思考

- 论文目前是 arXiv preprint，尚未看到正式会议/期刊接收信息。
- UniMM-HAR 是整合已有数据集，不是全新统一采集；不同 source 的动作定义、采样方式、场景标注仍可能带来对齐误差。
- TAM 的贡献相对较小，且依赖文本 prompt 和预训练文本空间，需要警惕它是否只是轻量加分项。
- DGR 使用重复增密而非真实生成新点，物理上更保守，但是否会带来 duplicated-point bias 需要看更多可视化和下游验证。
- C-Set 协议中 source/scene/subject/action 的耦合关系需要仔细看，避免把某种 split 下的提升过度解读为完全跨设备泛化。

### 后续精读问题

- UniMM-HAR 的 action alignment 是否会引入标签语义不一致？
- DSQ 的 learnable quantile 在不同动作/不同 source 下学到的阈值是否可解释？
- DGR 增密后点云空间结构是否真的更接近动作语义区域？
- TAM 使用 CLIP 文本空间是否有必要，是否可被简单 label embedding 替代？
- DAP-Net 在完全未见雷达设备上的泛化是否足够强？
- 能否把 D2R 模块接到 [[Star Graph + DDGNN]] 或其他图网络上？

### 可引用观点

- 毫米波点云 HAR 的真实难点不只是稀疏和噪声，还包括不同雷达源带来的结构性分布偏移。
- Doppler 是毫米波雷达点云区别于普通 3D 点云的重要物理线索，应显式参与运动建模。
- 跨源 benchmark 比单源随机划分更能检验模型是否学到真正的动作语义。

### 信息来源

- arXiv: arXiv:2605.09604
- GitHub: https://github.com/jolin830/DAP-Net

