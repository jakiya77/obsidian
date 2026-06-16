---
codex_dataview: true
type: knowledge
project: 数学与优化基础
area: knowledge
category: Optimum
parent: 优化算法
status: reference
priority: medium
created: '2026-05-11'
updated: '2026-05-16'
summary: 核心基础：流形几何概念
tags:
- knowledge
- Optimum
aliases:
- manifold optimization 流型约束
---

### 核心基础：流形几何概念

在讲解具体算法前，这两个算法都依赖以下三个流形优化的核心操作：

1. **欧式梯度（Euclidean Gradient）**：目标函数 $f(q) = q^H A q$ 在无约束欧式空间下的梯度。代码中表现为：
    
    $$\nabla f(q) = 2Aq$$
    
2. **黎曼梯度（Riemannian Gradient）**：将欧式梯度投影到当前点 $q$ 的切空间（Tangent Space）上，以保证搜索方向与流形相切。在复斜流形上，投影公式为：
    
   $$\text{grad} f(q) = \underbrace{\nabla f(q)}_{\text{普通梯度}} - \underbrace{\Re\{\nabla f(q) \odot q^*\} \odot q}_{\text{需要剔除的、破坏恒模约束的法向分量}}$$
    
    _(对应代码：`Rieman_gradient = gradient - real(gradient.*conj(q)).*q`)_
    [[Math：黎曼梯度的推导]]
1. **收回操作（Retraction）**：沿着切空间的切向量（搜索方向）移动后，点会离开流形，需要将其拉回到流形上（即归一化操作）。
    
    $$R_q(d) = \frac{q + d}{|q + d|}$$
    
    _(对应代码中的 `q(j) = q(j)/norm(q(j))`)_
    

---

### 算法一：一阶黎曼共轭梯度法 (First-Order Riemannian Conjugate Gradient)

这段代码的 `while(1)%%%%%%%start oblique manifold` 部分实现的是一阶黎曼共轭梯度法（RCG）。

#### 原理与逻辑：

一阶方法仅利用梯度（一阶导数）信息。最简单的梯度下降法容易出现Z字形震荡，而共轭梯度法通过结合前一次的搜索方向来修正当前方向，从而加速收敛。

#### 核心步骤与公式：

1. **计算搜索方向与向量传输（Vector Transport）**：
    
    由于流形是弯曲的，上一个迭代点 $q_0$ 处的搜索方向 $d_0$ 不能直接和当前点 $q_1$ 处的梯度相加。必须先用向量传输 $\mathcal{T}_{q_0 \to q_1}(d_0)$ 将 $d_0$ 映射到 $q_1$ 的切空间：
    
    $$\mathcal{T}(d_0) = d_0 - \Re\{d_0 \odot q_1^*\} \odot q_1$$
    
    _(对应代码：`T = d_0-(real(d_0.*conj(q_1))).*q_1`)_
    
2. **计算 Polak-Ribière 参数 $\beta$**：
    
    用于决定历史搜索方向的权重：
    
    $$\beta = \frac{\text{grad} f(q_1)^T (\text{grad} f(q_1) - \text{grad} f(q_0))}{\text{grad} f(q_0)^T \text{grad} f(q_0)}$$
    
    _(对应代码：计算 `beta` 的那行)_
    
3. **更新搜索方向（共轭方向）**：
    
    $$d_1 = -\text{grad} f(q_1) + \beta \mathcal{T}(d_0)$$
    
    _(对应代码：`d_1 = -1*Rieman_gradient_1 + beta*T`)_
    
4. **沿方向更新并收回（步长为 $\mu$）**：
    
    $$q_1 = R_{q_0}(\mu d_0)$$
    

---

### 算法二：二阶正则化黎曼牛顿法 (Second-Order Riemannian Newton Method)

代码后半部分（初始化 `s_0` 开始）实现的是二阶流形优化算法。

#### 原理与逻辑：

一阶方法在接近极值点时收敛会变得极其缓慢（呈线性收敛）。二阶算法（牛顿法）则额外利用了黎曼海塞矩阵（Riemannian Hessian，二阶导数信息）来捕捉流形的曲率，能够指引出更精确的下降方向，从而实现二次收敛（Quadratic Convergence）。

因为直接对复数矩阵求二阶导非常复杂，代码的逻辑是将 **$n$ 维复数向量拆解并映射为 $2n$ 维的实数向量，在实数域构建等价的海塞矩阵，最后再还原为复数步长。

#### 核心步骤与公式：

1. **实数化梯度（Real Representation of Gradient）**：
    
    将复数黎曼梯度拆为实部和虚部，拼接成 $2n \times 1$ 的实向量 $\bar{g}$：
    
    $$\bar{g} = \begin{bmatrix} \Re(\text{grad} f(s_0)) \\ \Im(\text{grad} f(s_0)) \end{bmatrix}$$
    
    _(对应代码：`g_bar = [(real( Rieman_gradient_0))', (imag( Rieman_gradient_0))']'`)_
    
2. **构建实数海塞矩阵（Constructing Real Hessian $\bar{H}_{ess}$）**：
    
    代码通过定义 $V_{\text{bar}}, R_{\text{bar}}, Q_{\text{bar}}$ 巧妙地组合出了欧式海塞矩阵在切空间投影后的等价实数矩阵 `HessgR`。这部分是复杂的 Wirtinger 微积分（复变量求导）的实数实现。
    
3. **正则化牛顿步长（Regularized Newton Step / Levenberg-Marquardt）**：
    
    标准的牛顿步长是求海塞矩阵的逆 $\Delta = -H^{-1}g$。但在非凸区域，海塞矩阵可能不是正定的。因此，代码引入了**阻尼因子 $\eta$**（代码中 `eta = 200`），求解正则化方程：
    
    $$\epsilon_R = -\left(\bar{H}_{ess} + \frac{\eta}{2} I_{2n}\right)^{-1} \bar{g}$$
    
    _(对应代码：`epsilon_R = -inv(HessgR+eta/2*eye(n*2))*g_bar`)_
    
4. **还原复数步长并收回**：
    
    将 $2n$ 维实向量 $\epsilon_R$ 重新组合为 $n$ 维复向量 $\epsilon$，并执行收回操作：
    
    $$s_1 = R_{s_0}(\epsilon) = \frac{s_0 + \epsilon}{|s_0 + \epsilon|}$$
    

---

### 二阶与一阶的优越性对比总结

正如你在代码注释中所强调的 `%% 二阶可以代入问题和一阶做对比 强调优越性`，这两种算法在性能上有非常鲜明的对比（对应代码最后生成的两张图）：

- **一阶（红线 `-ro`）**：单次迭代计算极其轻量（只有矩阵向量乘法）。但在接近最优解时，由于缺乏曲率信息，会出现“拉锯”现象，需要**数百到数千次迭代**才能使黎曼梯度的范数降到 $10^{-3}$ 以下。
    
- **二阶（蓝线 `-b*`）**：因为使用了海塞矩阵包含了曲率信息，算法可以直接“看到”最优点的方向。它的下降曲线（在对数图 `semilogy` 中）会呈现“断崖式”的急速下降。通常只需**十几到几十次迭代**就能达到极高的精度。
    

**代价**：二阶方法的优越性是用单次迭代的计算复杂度换来的（需要计算海塞矩阵并对其求逆，复杂度为 $\mathcal{O}(N^3)$）。但在维度 $n$（如100）不是极端巨大的情况下，二阶方法总体的计算时间和收敛效果通常远好于一阶方法。