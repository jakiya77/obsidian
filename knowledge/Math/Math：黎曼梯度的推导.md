---
codex_dataview: true
type: knowledge
project: 数学与优化基础
area: knowledge
category: Math
parent: 数学基础
status: reference
priority: medium
created: '2026-05-11'
updated: '2026-05-11'
summary: 核心原理：把算出来的普通梯度，强行按在流形的表面上，把垂直于表面的分量直接切掉。
tags:
- knowledge
- Math
aliases:
- 黎曼梯度的推导
---

>[!hint]+ 核心原理：把算出来的普通梯度，强行按在流形的表面上，把垂直于表面的分量直接切掉。

---

### 第一步：流形定义

在智能反射面（IRS）这类问题中，变量 $q$ 的每一个元素 $q_i$ 都是一个相移器，它只能改变信号的相位，不能改变幅度。

这就意味着，每个 $q_i$ 的模必须为 1。在复数域中，这可以写成：

$$|q_i|^2 = q_i q_i^* = 1$$

这就构成了一个**复圆（Complex Circle）**。当有很多个 $q_i$ 拼成一个向量 $q$ 时，它们构成了一个高维的流形，叫做**复斜流形（Complex Oblique Manifold）**。

### 第二步：定义切空间

假设我们现在站在圆上的某一点 $q_i$，想要沿着某个方向 $z$ 走一小步，同时**保证自己还在圆上**。这个方向 $z$ 就叫作**切向量（Tangent Vector）**。

如何用数学表达这个切方向？

我们对约束条件 $q_i q_i^* = 1$ 求导（假设 $q_i$ 随时间 $t$ 变化，求 $\frac{d}{dt}$）：

$$\dot{q_i} q_i^* + q_i \dot{q_i}^* = 0$$

仔细看这个式子，左边其实是一个复数加上它的共轭，即 $z + z^* = 2\Re\{z\}$。所以上面的式子等价于：

$$2\Re\{\dot{q_i} q_i^*\} = 0 \implies \Re\{\dot{q_i} q_i^*\} = 0$$

**结论 1**：任何一个合格的切方向 $z$，必须满足 **$\Re\{z q_i^*\} = 0$**。也就是说，$z q_i^*$ 必须是一个纯虚数。

### 第三步：定义法向量

如果切向量是沿着圆周切线的，那么法向量（Normal Vector）就是垂直于圆周、指向圆心或圆外的方向。

在单位圆上，点 $q_i$ 本身的位置向量，就是它的法向量方向。

因此，在 $q_i$ 这一点的任何法向量 $v_{\text{norm}}$，都可以表示为 $q_i$ 的实数倍：

$$v_{\text{norm}} = \alpha q_i \quad (\text{其中 } \alpha \text{ 是实数})$$

### 第四步：推导黎曼梯度公式（核心正交分解）

现在，我们计算出了目标函数在普通欧式空间下的梯度 $\nabla f(q_i)$。这个普通梯度通常是乱指的，既有沿着圆周的分量，也有想飞出圆周的分量。

我们要做的，就是把普通梯度 $\nabla f(q_i)$ 拆解为两部分：

$$\nabla f(q_i) = \text{切向分量 (黎曼梯度)} + \text{法向分量}$$

即：

$$\text{grad} f(q_i) = \nabla f(q_i) - \text{法向分量}$$

设法向分量为 $\alpha q_i$（$\alpha$ 待求）。那么黎曼梯度为：

$$\text{grad} f(q_i) = \nabla f(q_i) - \alpha q_i$$

根据**结论 1**，黎曼梯度既然在切空间里，就必须满足 $\Re\{\text{grad} f(q_i) \cdot q_i^*\} = 0$。我们把它代入：

$$\Re\{ (\nabla f(q_i) - \alpha q_i) q_i^* \} = 0$$

展开：

$$\Re\{ \nabla f(q_i) q_i^* \} - \Re\{ \alpha q_i q_i^* \} = 0$$

因为 $q_i q_i^* = 1$，且 $\alpha$ 是实数（实数的实部就是它自己），所以上式变成：

$$\Re\{ \nabla f(q_i) q_i^* \} - \alpha = 0$$

$$\implies \alpha = \Re\{ \nabla f(q_i) q_i^* \}$$

找到了 $\alpha$，我们把它代回黎曼梯度的公式：

$$\text{grad} f(q_i) = \nabla f(q_i) - \Re\{ \nabla f(q_i) q_i^* \} q_i$$

---

### 总结：推广到向量 $\odot$

上面是对**单个元素** $q_i$ 的推导。在你的代码中，$q$ 是一个列向量。因为约束是分别作用在每一个元素上的（每个反射阵子各自满足恒模约束），所以我们需要对向量的**每一个位置**都独立做上述的投影。

在数学符号和 MATLAB 代码中，逐元素相乘（Element-wise multiplication）就是用哈达玛积（Hadamard Product）符号 $\odot$ 表示的（对应 MATLAB 里的 `.*`）。

所以整个向量形式的公式就变成了你看到的样子：

$$\text{grad} f(q) = \underbrace{\nabla f(q)}_{\text{普通梯度}} - \underbrace{\Re\{\nabla f(q) \odot q^*\} \odot q}_{\text{需要剔除的、破坏恒模约束的法向分量}}$$