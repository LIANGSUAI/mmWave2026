# 毕设文献梳理与路线图

## 1. 经典基线模型 (Baseline)
* [[RadHAR]]：采用体素化预处理，结合 3D CNN + Bi-LSTM。奠定了雷达动作识别的基础流程，但计算量大。

## 2. 前沿与轻量化优化 (SOTA)
* [[RadMamba]]：针对 LSTM 和 Transformer 的长序列计算瓶颈，引入 Mamba 状态空间模型，实现 $O(N)$ 复杂度，专攻边缘设备部署。