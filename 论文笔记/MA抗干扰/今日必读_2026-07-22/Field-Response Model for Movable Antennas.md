---
title: "Modeling and Performance Analysis for Movable Antenna Enabled Wireless Communications"
method_name: "Field-Response Model"
authors: [Lipeng Zhu, Wenyan Ma, Rui Zhang]
year: 2024
venue: "IEEE Transactions on Wireless Communications"
tags: [movable-antenna, field-response, multipath-channel, spatial-diversity]
image_source: pdf
arxiv_html: https://arxiv.org/html/2210.05325
created: 2026-07-22
---

# 论文笔记：Field-Response Model for Movable Antennas

## 一句话总结

> 用路径幅相与 AoA/AoD 建立位置相关 field-response 模型，并证明多径越丰富，MA 越可能从位置域起伏中获益。

## 核心模型

对一维接收位置 $p$，你的模型可写为

$$
h(p)=\sum_{\ell=1}^{L}\alpha_{\ell}
\exp\!\left(-j\frac{2\pi}{\lambda}p\sin\theta_{\ell}\right).
$$

其中 $\alpha_\ell$ 是第 $\ell$ 条路径的复增益，$\theta_\ell$ 为到达角，$\lambda$ 为波长。位置改变会改变各路径相位，从而改变相干叠加结果。

论文的一般 3D 模型把发射/接收位置映射为 field-response vector，并以路径响应矩阵连接两端；远场条件下角度不随局部移动显著变化。

## 主要结论

1. 确定性多径下，信道增益在空间中呈周期/准周期起伏。
2. 随机信道下，推导无限大接收区域内最大信道增益上界的期望及近似 CDF。
3. 路径数增加会强化空间小尺度起伏，因此 MA 相对 FPA/AS 的潜在增益增大。
4. 单 MA 在合适区域内可获得接近多天线数字波束形成的 SNR，但只使用一条 RF 链。

## 关键图表（全文 13 图）

- Figures 1–2：系统、坐标和空间角定义。
- Figures 3–6：两径/多径信道随位置的周期结构与二维场分布。
- Figures 7–9：MA、FPA、AS、数字波束形成的 SNR/CDF/路径数对比。
- Figures 10–13：有限区域、随机信道与理论近似的补充验证。

## 对 PG-DFPS 的直接启发

- 是 $H(p)=|h(p)|^2$ 与 jammer 轮廓 $G(p)=|g(p)|^2$ 的直接理论来源。
- 单点 desired-favorable / jammer-weak 筛选利用的是各自位置域包络。
- 多阵元集合仍需考虑向量级 $\mathbf h(\mathbf p)$ 与 $\mathbf g(\mathbf p)$ 的可分性；这一步超出了单 MA 最大信道增益分析。

## 局限

主要研究单链路最大信道增益，不处理接收端 jammer、MVDR 或离散多阵元位置集构造。

## 链接

[arXiv](https://arxiv.org/abs/2210.05325) · PDF：`../PDF/R4_Field_Response_Model.pdf`

