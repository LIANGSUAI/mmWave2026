#毫米波 #行为识别 
### 论文标题：
RadMamba: Efficient Human Activity Recognition Through a Radar-Based Micro-Doppler-Oriented Mamba State-Space Mode
### 标题翻译
基于雷达微多普勒特征的 Mamba 状态空间模型实现高效人体活动识别

_IEEE Transactions on Radar Systems_ (IEEE T-RS)

###  **研究痛点 (Motivation):** 
- **传统做法：** 处理雷达信号（微多普勒频谱或点云）通常用 CNN、LSTM 或 Transformer。（例如经典的基线架构 [[RadHAR]]）或 Transformer。
- **痛点在哪？** Transformer 虽然牛，但它的自注意力机制计算量是$O(N^2)$。雷达只要连续监测几秒钟，序列$N$就会非常大，导致模型根本无法塞进智能家居的微型雷达芯片里。
### **核心创新 (Method):** 
- 作者将大模型领域最新的 Mamba (状态空间模型 SSM) 引入了雷达领域。
- 他们针对雷达微多普勒（Micro-Doppler, MD）信号的物理特性，定制了专门的 Mamba 块，实现了$O(N)$的线性计算复杂度。也就是：既有 Transformer 的全局视野，又像 RNN 一样轻量快速。
### 输入/输出 (I/O Shape):
 - **Input (输入):** 雷达原始数据经过 FFT 处理后的**微多普勒时频图 (Spectrograms)**。在计算机眼里，这本质上是一个时间步为 $T$ ，特征维度为 $D$ 的时序序列，形状大概是 [ Batch_size, T, D ] 或者二维图 [ Batch_size, Channels, Freq, Time ]。
- **Backbone (骨干):** 一系列的 RadMamba Block。
- **Output (输出):** 动作分类的概率分布（比如：走、跑、跌倒的概率）。

### 阅读过程
mamba要求输入为一维数据，而雷达数据通常是输入一个微多普勒时频图。
舍弃了传统 ViT/Mamba 破坏物理结构的 Linear 线性拉平投影（会丢失一维速度特征），改用 Conv1d (一维卷积) 进行 Token 投影。这不仅保住了多普勒频率的局部连续性，还大大降低了参数量。
在连续 FMCW 雷达数据集上，**仅用 6.7k 个参数**（参数量是同类 SSM 的 1/400 甚至 1/10），准确率反超其他大模型达到 SOTA (最高99.8%)。![[RadMamba架构图.png]]
#### 1. 机制一：通道融合与降采样 (Channel Fusion & Downsampling)
现在的雷达通常有多个接收天线（多通道）。如果把所有通道的数据都输入进去，计算量太大。所以作者第一步先做了一个融合，把多通道的冗余信息合并，并且把图中“没有动作的低价值空白区域”缩小，这一步极大地减少了后续计算的负担。
#### 2. 机制二：多普勒对齐切片 (Doppler-Aligned Segmentation)
![[Pasted image 20260330214313.png]]
- 这是解决“怎么把图片切成小块（Token）”的问题。传统的 ViT 像切豆腐一样，横着切一刀竖着切一刀，变成正方形的 Patch（比如 16x16）。但雷达图的纵坐标是“多普勒速度”，如果横向切断了，速度的连续性就被破坏了。
- **作者的做法：** 他们切片的时候，**顺着多普勒的频率方向切（Doppler-aligned）**，保证速度维度的物理意义是完整的。
#### 3. 机制三：一维卷积 Token 投影 (Convolutional Token Projections)
作者把它称为 **CP-Mamba Block**。切好的多普勒切片，在送入真正的 Mamba 之前，先用一个一维卷积滑动扫描一下。这不仅起到了降维的作用，还能让切片和切片之间产生“相邻的连贯性”。实际上是添加了若干个ConV1d
#### 4. Mamba 状态空间骨干网络 (The SSM state-space model（状态空间模型） Backbone)
经过前三步，原本大而笨重的雷达图，已经被压成了一根又细又长、且富含物理意义的（1D Sequence）。这时候，Mamba 登场了。它通过前面提到的“选择性扫描（Selective Scan）”，像一个极其聪明的筛子，把1D Sequence里代表“走路”、“跌倒”的黄金片段记住，把“静止不动”的废料扔掉。最后送入 Linear 层输出概率。

### 我的启发
还有在需要连续长时间不间断预测判断，应该具有明显优势
在边缘化设备等算力明显不够的设备上，可以参考部署mamba
 