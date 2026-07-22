---
title: "Movable-Antenna Position Optimization: A Graph-Based Approach"
method_name: "Graph-Based MA Position Optimization"
authors: [Weidong Mei, Xin Wei, Boyu Ning, Zhi Chen, Rui Zhang]
year: 2024
venue: "IEEE Wireless Communications Letters"
tags: [movable-antenna, discrete-positioning, graph-optimization, low-complexity]
image_source: pdf
arxiv_html: https://arxiv.org/html/2403.16886
created: 2026-07-22
---

# 论文笔记：Graph-Based MA Position Optimization

## 一句话总结

> 将连续移动区域采样成离散点，把带最小阵元间距的多 MA 选点转化为固定跳数最短路径，并给出线性时间近似更新。

## 问题与方法

场景为发射端多 MA 的 MISO 单用户链路，目标最大化接收信号功率：

$$
\max_{\mathcal S}\left|\sum_{m\in\mathcal S}h_m w_m\right|^2,
\quad |\mathcal S|=N,
$$

同时满足离散采样点与最小间距约束。固定位置后用 MRT 消去波束变量，选点目标可表示为点权/边权累积。

论文构造有向无环图：节点对应采样位置，边只连接满足间距的候选，选择 $N$ 个位置等价于固定 hop 数最短路径；定制动态规划获得多项式时间最优解。另给出 sequential update，以线性复杂度获得近最优解。

## 主要实验结论

- 中等采样密度已能明显超过固定阵列与普通 antenna selection。
- 图最优解与穷举一致，顺序更新接近最优但复杂度更低。
- 多径越丰富，位置优化收益越明显；单径下位置移动不改变信道幅度，方案趋同。

## 关键图表（全文 6 图）

- Figure 1：MA-MISO 系统。
- Figure 2：离散采样与图建模。
- Figures 3–6：算法性能、采样数/区域大小/路径数及复杂度比较。

## 与 PG-DFPS 的关键区别

共同点是离散网格和低复杂度集合选择；差异是本文主要最大化通信链路信号功率，而 PG-DFPS 同时使用 desired-favorable、jammer-weak 和集合级 $\rho_{hg}$。因此可安全写：现有离散方法主要面向通信目标，接收端多径抗干扰的物理引导离散选点仍不充分。

## 链接

[arXiv](https://arxiv.org/abs/2403.16886) · PDF：`../PDF/R11_Graph_Based_Position_Optimization.pdf`

