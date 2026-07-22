---
title: "Movable-Antenna Array Enhanced Beamforming: Achieving Full Array Gain with Null Steering"
method_name: "Full Array Gain with Null Steering"
authors: [Lipeng Zhu, Wenyan Ma, Rui Zhang]
year: 2023
venue: "IEEE Communications Letters"
tags: [movable-antenna, beamforming, null-steering, array-gain]
image_source: pdf
arxiv_html: https://arxiv.org/html/2308.08787
created: 2026-07-22
---

# 论文笔记：Full Array Gain with Null Steering

## 一句话总结

> 在特定阵元数与零陷方向条件下，联合设计线性 MA 阵列位置和相位权值，可同时获得期望方向满阵列增益和非期望方向零陷。

## 关键目标

期望方向 steering vector 为 $\mathbf a(\theta_0,\mathbf x)$，干扰方向为 $\theta_k$。设计阵元位置 $\mathbf x$ 与权值 $\mathbf w$，满足

$$
|\mathbf w^H\mathbf a(\theta_0,\mathbf x)|=N,
\qquad
\mathbf w^H\mathbf a(\theta_k,\mathbf x)=0,\;\forall k.
$$

满增益意味着各阵元对期望方向同相叠加；论文证明可通过特殊位置向量正交化 steering vectors，权值只需固定幅度的相位调整。

## 方法与结论

- 构造 steering-vector orthogonality 条件。
- 在可行的阵元数/方向数与孔径条件下给出闭式 APV/AWV。
- MA 相比固定 ULA 能解除“期望方向增益—干扰方向零陷”的部分几何折衷。

## 关键图表

- Figure 1：线性 MA 阵列与角度定义。
- Figure 2：$N=8$ 的最优位置向量示例。
- Figures 3–4：MA 与 FPA beampattern 对比。
- Figure 5：期望方向 beam gain 对比。
- Table I：不同阵元/零陷配置下的可行性或性能汇总。

## 对 PG-DFPS 的直接启发

- 可支撑“改变位置会重塑有效空间响应与零陷能力”。
- 你的场景是 desired/jammer **复合多径信道**，不应把结论简化为“对每条 jammer path 都严格打零陷”。更恰当的指标是集合级归一化相关性

$$
\rho_{hg}=\frac{|\mathbf h^H\mathbf g|}{\|\mathbf h\|\,\|\mathbf g\|},
$$

它衡量两个复合信道向量对接收合并器是否容易分离。

## 链接

[arXiv](https://arxiv.org/abs/2308.08787) · PDF：`../PDF/R7_Full_Array_Gain_Null_Steering.pdf`

