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
- **数据获取状态：** GitHub 未直接提供 processed UniMM-HAR 下载；README 写明 processed dataset 仅限学术研究邮件申请，联系人 `jylin25@stu.pku.edu.cn`。若不申请处理后数据，需要自行下载 RadHAR、mRI、MM-Fi 并运行预处理脚本。
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

![[DAP-Net架构图.png]]

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

#### Table 3：UniMM-HAR 上与已有方法对比

说明：Original PC Type 表示原方法面向的点云类型。MPC = mmWave radar point clouds，SPC = static point clouds，TPC = temporal point clouds。

| Model | Source | Original PC Type | C-Sub (%) | C-Set (%) |
|---|---|---|---:|---:|
| PointNet | CVPR'17 | SPC | 59.27 | 70.96 |
| DGCNN | NN'18 | SPC | 71.70 | 52.18 |
| RadHAR | mmNSS'19 | MPC | 42.49 | 48.47 |
| PointMLP | ICLR'22 | SPC | 71.06 | 78.13 |
| PST-Transformer | TPAMI'22 | TPC | 63.13 | 79.70 |
| PointCLIP | CVPR'22 | SPC | 67.82 | 69.82 |
| Clip2point | ICCV'23 | SPC | 64.17 | 59.56 |
| FastHAR | CIKM'24 | MPC | 53.72 | 61.16 |
| 3DInAction | CVPR'24 | TPC | 73.43 | 57.81 |
| UST-SSM | ICCV'25 | TPC | 71.50 | 48.40 |
| **DAP-Net** | - | MPC | **80.72** | **81.82** |

DAP-Net 在 C-Sub 和 C-Set 都最高，尤其说明它在异构跨源场景下比普通点云 backbone 和已有毫米波 HAR 方法更稳。

#### Table 4：D2R 与 TAM 对不同 backbone 的影响

| Backbone | Module | Acc (%) |
|---|---|---:|
| PointMLP | - | 71.06 |
| PointMLP | +D2R | 80.00 ↑8.94 |
| PointMLP | +D2R+TAM | 80.72 ↑9.66 |
| UST-SSM | - | 71.50 |
| UST-SSM | +D2R | 74.00 ↑2.50 |
| UST-SSM | +D2R+TAM | 74.94 ↑3.44 |
| PST-Transformer | - | 63.13 |
| PST-Transformer | +D2R | 76.73 ↑13.60 |
| PST-Transformer | +D2R+TAM | 78.21 ↑15.08 |

主要结论：D2R 是核心增益来源，TAM 有稳定但较小的增益；D2R 对 PointMLP、UST-SSM、PST-Transformer 都有效，说明它更像一个可插拔的 Doppler-aware 前端模块。

### 消融结果

#### Table 5：D2R 子模块消融

| Backbone | DGR | MFR | Acc (%) |
|---|---:|---:|---:|
| PointMLP | ✗ | ✗ | 71.06 |
| PointMLP | ✓ | ✗ | 76.97 ↑5.91 |
| PointMLP | ✗ | ✓ | 78.76 ↑7.70 |
| PointMLP | ✓ | ✓ | 80.00 ↑8.94 |
| UST-SSM | ✗ | ✗ | 71.50 |
| UST-SSM | ✓ | ✗ | 73.69 ↑2.19 |
| UST-SSM | ✓ | ✓ | 74.00 ↑2.50 |
| PST-Transformer | ✗ | ✗ | 63.13 |
| PST-Transformer | ✓ | ✗ | 72.31 ↑9.18 |
| PST-Transformer | ✓ | ✓ | 76.73 ↑13.60 |

结论：DGR 和 MFR 都能带来提升，组合后最好。MFR 在 PointMLP 上单独提升更明显，说明运动显著点对通道特征重校准很有效。

#### Table 6：DSQ 与 TMPD 设计消融

| Backbone | Points Split | Fast Branch Densification |     Acc (%) |
| -------- | ------------ | ------------------------- | ----------: |
| PointMLP | -            | -                         |       71.06 |
| PointMLP | 0.2 quantile | MLP densification         | 71.00 ↓0.06 |
| PointMLP | 0.2 quantile | r-fold duplication        | 76.46 ↑5.40 |
| PointMLP | DSQ          | r-fold duplication        | 76.97 ↑5.91 |
| UST-SSM  | -            | -                         |       71.50 |
| UST-SSM  | 0.2 quantile | MLP densification         | 71.40 ↓0.10 |
| UST-SSM  | 0.2 quantile | r-fold duplication        | 73.40 ↑1.90 |
| UST-SSM  | DSQ          | r-fold duplication        | 74.00 ↑2.50 |

结论：论文选择重复增密而不是 MLP 生成新点，是一个很关键的设计。MLP densification 可能会引入伪结构或噪声；基于 Doppler 的 fast points 重复采样更保守，也更稳定。

#### Table 7：固定分位数与 DSQ 对比

| Quantile | Split Type | Acc (%) |
|---|---|---:|
| - | - | 71.06 |
| 0.2 | Fixed | 76.46 ↑5.40 |
| 0.3 | Fixed | 76.55 ↑5.40 |
| 0.5 | Fixed | 76.14 ↑5.08 |
| 0.8 | Fixed | 76.12 ↑5.06 |
| DSQ | Learnable | 76.97 ↑5.91 |

结论：固定阈值已经说明 Doppler fast points 有用，但 DSQ 的 learnable quantile 最好，更符合“不同设备 Doppler 尺度不一致”的问题设定。注意：原表中 `0.3 Fixed` 的提升写作 ↑5.40，按 76.55 - 71.06 计算应约为 ↑5.49，这里保留原表数值并标记为论文原文。

#### Table 8：几何增密方法对比

| Densification | Acc (%) |
|---|---:|
| Repeat Sampling | 71.06 |
| Super-frame Fusion | 59.72 ↓11.34 |
| MLP | 43.54 ↓27.52 |
| DGR | 76.97 ↑5.91 |

结论：普通 repeat sampling 只是补点，不会突出动作关键区域；super-frame fusion 和 MLP 反而明显下降，说明在毫米波稀疏点云里“生成更多点”不一定有用，关键是围绕 motion-salient points 做有约束的重参数化。

#### Table 9：TAM 的 prompt 与文本编码器消融

| Method | Action Text Prompt | Text Encoder | Acc (%) |
|---|---|---|---:|
| Baseline | - | - | 80.00 |
| w/ TAM | `[CLS]` | CLIP | 80.71 ↑0.71 |
| w/ TAM | `A person performing [CLS]` | CLIP | 80.65 ↑0.65 |
| w/ TAM | `a mmWave point cloud of a person [CLS]` | SBERT | 80.67 ↑0.67 |
| w/ TAM | `a mmWave point cloud of a person [CLS]` | CLIP | 80.72 ↑0.72 |

结论：TAM 的提升大约 0.7%，属于小但稳定的增益。prompt 里显式加入 mmWave point cloud 语境略好；CLIP 略优于 SBERT。精读时可以把 TAM 当作补充模块，重点仍应放在 D2R。

### 附录实验结果

#### Table 16：r-fold 重复增密因子

| Method | r-fold | Acc (%) |
|---|---:|---:|
| DAP-Net (w/o TAM) | 1 | 78.76 |
| DAP-Net (w/o TAM) | 5 | 79.99 ↑1.23 |
| DAP-Net (w/o TAM) | 10 | 79.19 ↑0.43 |

结论：`r=5` 最好。重复太少不能充分强调 fast points，重复太多可能放大冗余和噪声。

#### Table 17：Pgoal 点数设置

| Method | Pgoal | Acc (%) |
|---|---:|---:|
| DAP-Net (w/o TAM) | 512 | 79.61 |
| DAP-Net (w/o TAM) | 1024 | 79.99 |
| DAP-Net (w/o TAM) | 2048 | 77.55 |

结论：`Pgoal=1024` 最好。512 信息量偏少，2048 可能引入更多重复点和噪声。

#### Table 18：不同 backbone 接入 D2R / TAM 后的参数量与 FLOPs

| Backbone | Params (M) | FLOPs (G) |
|---|---:|---:|
| PointMLP | 13.24 | 15.75 |
| +D2R | 13.30 ↑0.06 | 15.75 ↑0.00 |
| +D2R+TAM | 14.43 ↑1.19 | 15.84 ↑0.09 |
| UST-SSM | 9.19 | 2.79 |
| +D2R | 9.27 ↑0.08 | 2.79 ↑0.00 |
| +D2R+TAM | 10.03 ↑0.84 | 2.79 ↑0.00 |

结论：D2R 带来的计算和参数开销很小；TAM 主要增加参数量，但 FLOPs 增量也很低。

#### Table 19：DGR 与 MFR 的模块级开销

| Backbone / Module | Params (M) | Params (%) | FLOPs (M) | FLOPs (%) |
|---|---:|---:|---:|---:|
| PointMLP / DGR | 0 | 0.000% | 0 | 0.000% |
| PointMLP / MFR | 0.06 | 0.505% | 2.16 | 0.014% |
| UST-SSM / DGR | 0 | 0.000% | 0 | 0.000% |
| UST-SSM / MFR | 0.06 | 0.650% | 0.05 | 0.002% |

结论：DGR 几乎没有额外参数和 FLOPs，因为它主要是基于 Doppler 的分组和重复增密；MFR 的额外开销也很小，因此 D2R 的性价比很高。

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

- 它不是简单追求某个数据集准确率，而是引入了真实部署中的 cross-source generalization 问题。
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
