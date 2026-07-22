---
title: "Joint Beamforming and Antenna Position Optimization for Movable Antenna-Assisted Spectrum Sharing"
method_name: "MA-Assisted Spectrum Sharing AO"
authors: [Xin Wei, Weidong Mei, Dong Wang, Boyu Ning, Zhi Chen]
year: 2024
venue: "IEEE Wireless Communications Letters"
tags: [movable-antenna, spectrum-sharing, interference-constraint, alternating-optimization]
image_source: pdf
arxiv_html: https://arxiv.org/html/2406.19590
created: 2026-07-22
---

# 论文笔记：MA-Assisted Spectrum Sharing

## 一句话总结

> 在认知无线电中联合优化次用户发射波束与 MA 位置，在提高 SR 接收功率的同时限制对多个 PR 的共信道干扰。

## 问题形式

$$
\max_{\mathbf w,\mathbf T}\;|\mathbf h_s^H(\mathbf T)\mathbf w|^2
\quad\text{s.t.}\quad
|\mathbf h_k^H(\mathbf T)\mathbf w|^2\le \Gamma_k,\;\forall k,
\quad \|\mathbf w\|^2\le P.
$$

desired 与 interference powers 都是 MA 位置的高度非线性函数。

## 方法

- 理论分析说明：移动区域足够大时，MA 有机会在维持对 SR 的 MRT 增益时减小对多个 PR 的泄漏。
- 用 AO 交替优化 beamformer 与位置，获得高质量次优解。
- 与 PSO、ZF、FPA 等 baseline 对比。

## 关键发现

- AO 接近 PSO，但通常以更结构化的方式获得更好或相当性能。
- 干扰阈值较严时，单纯 MRT 难以满足约束，位置/波束联合设计更重要。
- 路径数增加会带来更强空间多样性，MA 与 FPA 的性能差距扩大。

## 关键图表（全文 2 图）

- Figure 1：次级发射机、次级接收机与多个主用户接收机的系统模型。
- Figure 2(a–c)：SR SNR 随移动区域、干扰阈值和路径数的变化。

## 对 PG-DFPS 的作用

它证明 desired gain 与 interference suppression 应联合考虑，但目标仍通过 AO 直接迭代优化。你的单点双轮廓 $H(p),G(p)$ 是更轻量的物理代理，集合级相关性则补足“逐点低干扰不等于阵列级易分离”。

## 链接

[arXiv](https://arxiv.org/abs/2406.19590) · PDF：`../PDF/R10_Spectrum_Sharing_AO.pdf`

