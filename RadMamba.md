# 论文标题：
RadMamba: Efficient Human Activity Recognition Through a Radar-Based Micro-Doppler-Oriented Mamba State-Space Mode
# 标题翻译
基于雷达微多普勒特征的 Mamba 状态空间模型实现高效人体活动识别

#  **研究痛点 (Motivation):** 
- **传统做法：** 处理雷达信号（微多普勒频谱或点云）通常用 CNN、LSTM 或 Transformer。
- **痛点在哪？** Transformer 虽然牛，但它的自注意力机制计算量是$O(N^2)$。雷达只要连续监测几秒钟，序列$N$就会非常大，导致模型根本无法塞进智能家居的微型雷达芯片里。
# **核心创新 (Method):** 
- 作者将大模型领域最新的 Mamba (状态空间模型 SSM) 引入了雷达领域。
- 他们针对雷达微多普勒（Micro-Doppler, MD）信号的物理特性，定制了专门的 Mamba 块，实现了$O(N)$的线性计算复杂度。也就是：既有 Transformer 的全局视野，又像 RNN 一样轻量快速。
# 输入/输出 (I/O Shape):
 - **Input (输入):** 雷达原始数据经过 FFT 处理后的**微多普勒时频图 (Spectrograms)**。在计算机眼里，这本质上是一个时间步为 $T$ ，特征维度为 $D$ 的时序序列，形状大概是 [ Batch_size, T, D ] 或者二维图 [ Batch_size, Channels, Freq, Time ]。
- **Backbone (骨干):** 一系列的 RadMamba Block。
- **Output (输出):** 动作分类的概率分布（比如：走、跑、跌倒的概率）。

# 阅读过程

![[Pasted image 20260330145122.png]]
mamba要求输入为一维数据，而雷达数据通常是输入一个微多普勒时频图。
舍弃了传统 ViT/Mamba 破坏物理结构的 Linear 线性拉平投影（会丢失一维速度特征），改用 Conv1d (一维卷积) 进行 Token 投影。这不仅保住了多普勒频率的局部连续性，还大大降低了参数量。
在连续 FMCW 雷达数据集上，**仅用 6.7k 个参数**（参数量是同类 SSM 的 1/400 甚至 1/10），准确率反超其他大模型达到 SOTA (最高99.8%)。
# 我的启发
还有在需要连续长时间不间断预测判断，应该具有明显优势
在边缘化设备等算力明显不够的设备上，可以参考部署mamba
 