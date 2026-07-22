---
title: "Deep Learning-Assisted Jamming Mitigation with Movable Antenna Array"
method_name: "DL-Assisted MA Jamming Mitigation"
authors: [Xiao Tang, Yudan Jiang, Jinxin Liu, Qinghe Du, Dusit Niyato, Zhu Han]
year: 2025
venue: "IEEE Transactions on Vehicular Technology"
tags: [anti-jamming, movable-antenna, receive-beamforming, deep-learning]
image_source: pdf
arxiv_html: https://arxiv.org/html/2410.20344
created: 2026-07-22
---

# 论文笔记：Deep Learning-Assisted Jamming Mitigation

## 一句话总结

> 将接收波束闭式化为广义 Rayleigh quotient，再训练 MLP 预测接收端 MA 阵元位置，以低在线复杂度逼近最大 SINR。

## 系统与目标

合法链路受到多个 jammer 干扰，接收端配置 MA 阵列。联合位置 $\mathbf T$ 与接收权值 $\mathbf w$：

$$
\max_{\mathbf T,\mathbf w}
\frac{P_s|\mathbf w^H\mathbf h_s(\mathbf T)|^2}
{\mathbf w^H\left(\sum_j P_j\mathbf h_j(\mathbf T)\mathbf h_j^H(\mathbf T)+\sigma^2\mathbf I\right)\mathbf w}.
$$

固定位置时，最优接收器由 generalized Rayleigh quotient 给出：

$$
\mathbf w^\star\propto \mathbf R_{i+n}^{-1}\mathbf h_s.
$$

## 方法

- 用解析接收器消除 $\mathbf w$。
- MLP 输入信道/场景特征，输出满足区域和间距约束的阵元位置。
- 以负 SINR 或等价目标进行 SGD 离线训练，在线仅需一次前向推理。

## 实验结论

- 达到接近数值搜索的抗干扰 SINR。
- 随阵元数和移动区域增大，性能提高；jammer 数增加时优势仍存在。
- 在线推理快，但训练成本和分布外泛化不在“低复杂度”口径中消失。

## 关键图表（全文 6 图）

- Figure 1：接收端 MA 抗干扰系统。
- Figure 2：MLP 学习框架。
- Figures 3–5：SINR 随阵元数、区域大小、jammer 数变化。
- Figure 6：算法运行时间。

## 与 PG-DFPS 的 novelty 边界

不能再声称“首次接收端 MA 抗干扰”。应强调：PG-DFPS 无需离线数据/训练，直接使用可解释的位置域 desired-favorable、jammer-weak 轮廓与集合级信道可分性构造离散位置集。

## 局限

- 依赖训练分布和训练预算。
- 网络输出为何选择某些位置缺乏物理可解释性。
- 环境、路径或 jammer 统计改变时可能需重新训练。

## 链接

[arXiv](https://arxiv.org/abs/2410.20344) · PDF：`../PDF/R13_DL_Jamming_Mitigation.pdf`

