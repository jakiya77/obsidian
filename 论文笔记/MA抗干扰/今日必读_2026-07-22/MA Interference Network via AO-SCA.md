---
title: "Movable Antenna Enabled Interference Network: Joint Antenna Position and Beamforming Design"
method_name: "MA AO-SCA Interference Network"
authors: [Honghao Wang, Qingqing Wu, Wen Chen]
year: 2024
venue: "IEEE Wireless Communications Letters"
tags: [movable-antenna, interference-network, alternating-optimization, successive-convex-approximation]
image_source: pdf
arxiv_html: https://arxiv.org/html/2403.13573
created: 2026-07-22
---

# 论文笔记：MA Interference Network via AO-SCA

## 一句话总结

> 在 MISO 干扰网络中联合优化发射端 MA 位置与波束，利用 BCD、SCA 和 SOCP 在满足 SINR 约束时降低总发射功率。

## 优化问题

$$
\min_{\{\mathbf w_k,\mathbf T_k\}}\sum_k\|\mathbf w_k\|^2
\quad\text{s.t.}\quad
\frac{|\mathbf h_{kk}^H(\mathbf T_k)\mathbf w_k|^2}
{\sum_{j\ne k}|\mathbf h_{jk}^H(\mathbf T_j)\mathbf w_j|^2+\sigma_k^2}
\ge \gamma_k.
$$

位置通过 field-response 非线性进入 desired 与 cross-link，且与 beamformer 强耦合。

## 求解框架

1. 固定位置，以 SOCP 更新波束形成。
2. 固定波束，通过辅助变量和 SCA 对非凸位置响应作局部凸近似。
3. 在 BCD 外循环中交替迭代至收敛。

## 主要结论

- MA 通过增强 desired link、削弱 cross-link，允许更多同频小区或更低总功率。
- 示例中 4 根 MA + MRT 可达到约 9 根固定天线 + SOCP 的性能，体现几何自由度对硬件/数字设计的替代价值。

## 关键图表（全文 4 图）

- Figure 1：多小区 MISO 干扰网络。
- Figure 2：算法收敛与性能。
- Figures 3–4：总功率随小区数、阵元数或移动区域的变化。

## 对 PG-DFPS 的作用

这是 AO-SCA-CVX baseline 的直接引用。它也清楚说明复杂度来源：每轮既要构造局部近似，又要重复求解 SOCP/凸子问题。PG-DFPS 的卖点是搜索中不反复求接收机级 SINR/CVX，而使用可解释的点级与集合级代理。

## 链接

[arXiv](https://arxiv.org/abs/2403.13573) · PDF：`../PDF/R9_AO_SCA_Interference_Network.pdf`

