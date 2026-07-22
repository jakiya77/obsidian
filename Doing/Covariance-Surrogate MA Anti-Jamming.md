---
title: "A Covariance-Surrogate Trust-Region Framework for Movable-Antenna Enabled Anti-Jamming with Unknown Jammers"
method_name: "PTRSO"
authors: [Lebin Chen, Ming-Min Zhao, Qingqing Wu, Min-Jian Zhao, Rui Zhang]
year: 2025
venue: arXiv
tags: [anti-jamming, movable-antenna, mvdr, trust-region, sample-covariance]
image_source: pdf
arxiv_html: https://arxiv.org/html/2512.20380
created: 2026-07-22
---

# 论文笔记：Covariance-Surrogate MA Anti-Jamming

## 一句话总结

> 在 jammer 信道未知时，以当前位置的样本协方差构造 MVDR 输出 SINR surrogate，并用投影信赖域迭代更新接收端 MA 位置。

## 系统与 MVDR

接收端 MA 阵列面对多个未知 jammer。固定位置与协方差估计后，MVDR 为

$$
\mathbf w_{\mathrm{MVDR}}=
\frac{\widehat{\mathbf R}_{i+n}^{-1}\mathbf h_s}
{\mathbf h_s^H\widehat{\mathbf R}_{i+n}^{-1}\mathbf h_s}.
$$

相应 surrogate objective 与 $\mathbf h_s^H\widehat{\mathbf R}_{i+n}^{-1}\mathbf h_s$ 成正比。

## 两时间尺度框架

- 每个 snapshot 更新 MVDR beamformer。
- 每个 block 在当前位置 anchor 采样并更新协方差 surrogate。
- 位置只在信赖域内局部改变，以控制“旧位置样本协方差用于新位置”的模型失配。

## 理论与算法

1. 用矩阵集中不等式给出 surrogate 与真实目标误差的局部界。
2. 证明自然的历史平均 surrogate 存在不会随样本数消失的几何偏差。
3. 提出 PTRSO：projected trust-region surrogate optimization；梯度步后投影到位置区域/间距可行集，并限制每轮移动半径。
4. 证明在 anchor 邻域收敛到 surrogate 问题的驻点。

## 主要实验结论

- PTRSO 在不同用户 SNR、接收阵元数、每 block snapshot 数、jammer 数下优于基线。
- snapshot 数增加通常改善协方差估计；jammer 增多使问题更难。
- beampattern 展示优化位置后对 jammer 方向形成更深抑制，同时维持 desired gain。

## 关键图表（全文 10 图）

- Figure 1：MA-MIMO 接收端抗干扰系统。
- Figure 2：两时间尺度框架。
- Figures 3–4：surrogate/真实目标及信赖域几何示例。
- Figures 5–9：SINR、有效性、阵元数、snapshot 数、jammer 数实验。
- Figure 10：不同位置方案的 beampattern。

## 与 PG-DFPS 的关键区别

- R14 解决 jammer CSI 未知与协方差估计；PG-DFPS 假设位置域信道或路径参数可获得。
- R14 是连续、局部、迭代的 surrogate/TR 更新；PG-DFPS 是离散、物理引导的候选预筛与集合构造。
- 二者可被表述为不同信息条件下的互补路线，而非简单性能替代。

## 注意

附件称其为 2025 arXiv 预印本；PDF 注明早期版本已被 IEEE ICC 2026 接收，但当前引用仍应以 arXiv 版本为准，除非正式 proceedings 信息已核实。

## 链接

[arXiv](https://arxiv.org/abs/2512.20380) · PDF：`../PDF/R14_Covariance_Surrogate_Anti_Jamming.pdf`

