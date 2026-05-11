
---
aliases: [相干信号的秩亏缺, 多径相干性, DARISAA抗干扰]
tags: [文献笔记, 阵列信号处理, DoA估计, 抗干扰, 多径效应]


> [!abstract] 来源文献
> [cite_start]**标题**：Combating Suppressive Jamming with Dynamic Agile Reconfigurable Intelligent Surfaces Antenna Array (DARISAA) [cite: 1, 2]
> [cite_start]**核心议题**：在强压制干扰下，利用动态敏捷可重构智能表面天线阵列（DARISAA）在模拟域消除干扰，并进行目标信号与相干干扰的 DoA 估计 [cite: 7, 8, 9, 10]。

## 1. 核心问题：相干干扰与秩亏缺 (Rank-Deficient)

> [!quote] 论文原文片段
> [cite_start]"Since multipaths with different DoAs are coming from the same jamming source, this is a DoA estimation of coherent signals. It is well known that subspace-based DoA estimation algorithms such as the MUSIC algorithm and ESPRIT, can be employed to estimate the DoA with super-resolution, but they fail with coherent signals because the constructed covariance matrix is rank-deficient." [cite: 233, 234]

### 现象与底层逻辑
[cite_start]在存在多径干扰的场景中，$K$ 个不同方向到达的干扰信号本质上来源于同一个干扰机 [cite: 232, 233]。构建的协方差矩阵 $R_z$ 中代表干扰的部分为：
[cite_start]$$R_J = (1-\alpha)^2 P_J V^H A_J \beta_J \beta_J^H A_J^H V$$ [cite: 239]
* [cite_start]$A_J$ 是阵列流形矩阵（包含 $K$ 个方向信息，维度 $N \times K$） [cite: 163]。
* [cite_start]$\beta_J$ 是多径衰减系数列向量（维度 $K \times 1$） [cite: 164]。

[cite_start]由于 $A_J \beta_J$ 矩阵乘法会产生一个单一的**列向量**，其外积矩阵必然导致秩 (Rank) 为 1。因此，无论存在多少个 ($K$) 相干干扰信号，信号子空间的有效维度仅为 1（即发生秩亏缺），这直接导致了 MUSIC 和 ESPRIT 等依赖信号子空间秩数等于信号源数的传统算法失效 [cite: 234, 243]。

### 论文的解决方案
[cite_start]传统的前向空间平滑（FSS）方法虽然能恢复秩，但会损失阵列孔径并降低估计精度 [cite: 235][cite_start]。本文提出利用 DARISA 的**动态敏捷调整**特性，采用**时间平滑滤波（TSS）**方法：在一个符号周期内，快速变换天线响应模式进行 $T_p$ 次观测 [cite: 236, 245][cite_start]。通过对这多个观测协方差矩阵求平均，在不损失物理阵列孔径的前提下恢复了相干信号矩阵的秩 [cite: 246, 282]。

---

## 2. 个人疑问与概念辨析

> [!question] 疑问 1：既然多径到达的幅度和相位都不一样了，为什么还叫“时间上完全相干”？相干性体现在哪？

**核心解答：线性相关性决定相干性**
信号处理中的“相干（Coherence）”指的是**完全的线性相关**，而不是指两路信号在特定时刻的绝对值相等。
* **物理本质**：无论是路径 1 ($x_1(t) = \beta_1 s_J(t)$) 还是路径 2 ($x_2(t) = \beta_2 s_J(t)$)，它们底层的波形起伏（基带包络）是死死绑定在一起的。
* **数学关系**：两路信号之间永远存在一个恒定的复数比例常数，即 $x_2(t) = \frac{\beta_2}{\beta_1} x_1(t)$。
* **对秩的影响**：在构建阵列协方差矩阵时，代表不同多径的各列向量虽然具体数值不同，但全都是第一列的倍数（极度线性相关）。它们无法为矩阵提供新的有效数学维度，因此整个矩阵表现为 Rank 1。

> [!question] 疑问 2：如果是多径传输导致波形发生了改变，比如该是波峰的地方变成了波谷，这还会是完全相干的吗？

**核心解答：波峰变波谷（相位反转）依然是完全相干的，且符合窄带假设。**
* **线性相关的包容性**：波峰变波谷（$180^\circ$ 或 $\pi$ 的相位差）在数学上等价于乘上常数 $-1$（即复平面上的相位旋转 $e^{j\pi}$）。由于 $x_2(t) = -1 \cdot x_1(t)$ 依然满足常数比例关系，所以它们保持着完美的线性相关。
* **窄带假设的视角（关键点）**：
    * **高频载波**：波长极短，多径造成微小的空间距离差（如几厘米）就足以引起剧烈的相位偏移，导致你所想的“波峰变波谷”。
    * **低频基带包络**：变化极其缓慢，携带了真正的信号宏观形状。多径造成的纳秒级时间延迟相对于基带符号周期而言微乎其微。
* **结论**：高频载波的“波峰变波谷”在基带等效模型中仅仅表现为一个复衰减系数 $\beta$ 的相角旋转。宏观的信号包络并未发生错位，数学上依然是乘以一个复常数，因此仍然是 Rank 1 的完全相干信号。