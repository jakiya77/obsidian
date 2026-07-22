---
type: concept
aliases: [最小方差无失真响应, Minimum Variance Distortionless Response, Capon beamformer]
---

# MVDR

## 定义

在保持期望信号方向/信道无失真的约束下，使输出干扰加噪声功率最小的自适应接收波束形成器。

## 数学形式

$$
\min_{\mathbf w}\;\mathbf w^H\mathbf R_{i+n}\mathbf w
\quad\text{s.t.}\quad
\mathbf w^H\mathbf h=1,
$$

其闭式解为

$$
\mathbf w_{\mathrm{MVDR}}=
\frac{\mathbf R_{i+n}^{-1}\mathbf h}
{\mathbf h^H\mathbf R_{i+n}^{-1}\mathbf h}.
$$

## 核心要点

1. $\mathbf h$ 是期望信道/steering vector，$\mathbf R_{i+n}$ 是干扰加噪声协方差。
2. 输出 SINR 与 $\mathbf h^H\mathbf R_{i+n}^{-1}\mathbf h$ 成正比。
3. 对 MA 接收阵列，天线位置会同时改变 $\mathbf h$ 与 $\mathbf R_{i+n}$，从而使位置优化高度非凸。

## 代表工作

- [[Covariance-Surrogate MA Anti-Jamming]]：未知 jammer 下的样本协方差与信赖域位置优化。
- [[Deep Learning-Assisted Jamming Mitigation]]：以 Rayleigh quotient 消去接收波束变量。

## 相关概念

- [[Full Array Gain with Null Steering]]
- [[Field-Response Model for Movable Antennas]]

