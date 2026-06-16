---
codex_dataview: true
type: knowledge
project: 数学与优化基础
area: knowledge
category: Optimum
parent: 优化算法
status: reference
priority: medium
created: '2026-05-06'
updated: '2026-05-06'
summary: Optimization
tags:
- knowledge
- Optimum
- MVDR
- BCD
- SDR
- AO
- Beamforming
- Anti-Jamming
- Interference
aliases:
- SDR核心机制剖析
---
---
tags:
  - Optimization
  - SDR
  - Manifold-Optimization
  - Beamforming
  - Algorithm
date: {{date}}
---

> [!abstract] 笔记背景
> 本笔记整理自关于抗干扰波束赋形、动态阵元相位联合优化（MINLP 问题）中非凸优化算法的讨论。核心聚焦于**流形优化 (MO)** 与 **半正定松弛 (SDR)** 的对比，以及 SDR 底层数学逻辑（升维、迹、秩、高斯随机化）的物理意义。

---

## 一、 核心算法框架：流形优化 (MO) vs. 半正定松弛 (SDR)

> [!faq] 提问 1：什么是SDR 

* **核心思想：** 通过升维，将非凸的二次型问题转化为凸的半正定规划 (SDP) 问题，代价是舍弃极度非凸的秩一约束。
* **优势：** 能提供严格的理论性能上界，结合高斯随机化能得到极高质量的次优解。
* **劣势：** 变量升维至 $\mathcal{O}(N^2)$ 矩阵级别，计算复杂度通常高达 $\mathcal{O}(N^{4.5})$，不适合大规模阵列。

### 具体算法步骤
#### 第一步：变量提升 (Lifting)

我们不再直接求解向量 $\mathbf{w}$，而是定义一个新的矩阵变量 $\mathbf{X}$：

$$\mathbf{X} = \mathbf{w}\mathbf{w}^H$$

这样做的好处是，原本目标函数里的二次项 $\mathbf{w}^H \mathbf{H} \mathbf{w}$ 变成了线性项 $\text{Tr}(\mathbf{H}\mathbf{X})$。线性函数是天生凸的，好算得多。

#### 第二步：识别“祸根”

虽然目标函数变凸了，但新变量 $\mathbf{X}$ 必须满足两个硬性条件：

1. **半正定性**：$\mathbf{X} \succeq 0$（这个是凸约束，好办）。
    
2. **秩为1**：$\text{rank}(\mathbf{X}) = 1$（**这就是非凸的祸根**，因为它要求矩阵只能由一个向量生成，不能是多个向量的叠加）。
    

    

#### 第三步：松弛 (Relaxation)

**这是最关键的一步：** 我们假装看不见 $\text{rank}(\mathbf{X}) = 1$ 这个约束，直接把它**扔掉**！

剩下的问题就变成了：


$$\min_{\mathbf{X}} \text{Tr}(\mathbf{H}\mathbf{X}), \quad \text{s.t.} \quad \mathbf{X} \succeq 0, \quad \text{其他线性约束}$$

这就是一个标准的**半正定规划 (SDP)** 问题。它是凸的，可以用成熟的工具箱（如 CVX）在多项式时间内算出全局最优解。

---

## 二、 复杂系统中的算法选型策略 

> [!faq] 提问 2：面对连续变量 $\mathbf{w}$ 和离散变量 $\mathbf{\Phi}$ 深度耦合的优化问题（复杂度 $\mathcal{O}(K^N)$），用什么算法更好？

对于此类**混合整数非线性规划 (MINLP)** 问题，基础框架通常是 **交替优化 (AO / BCD)**：利用 MVDR 准则求 $\mathbf{w}$ 的闭式解，交替求解离散的 $\mathbf{\Phi}$。

针对 $\mathbf{\Phi}$ 的求解，有三大流派（根据核心诉求选择）：
1.  **理论分析基准 (Benchmark)：SDR / MO + 最近邻量化。**
    * 先求连续次优解，再强制投影到离散集合。逻辑严密，适合发高水平 Paper 提供性能对比。
2.  **避免强制量化的平滑过渡：连续凸近似与惩罚函数法 (SCA + Penalty)。**
    * 在目标函数中加入惩罚项，随着迭代强迫连续变量向离散值靠拢。数学上更具说服力。
3.  **工程落地与极低复杂度：逐元素坐标下降法 (Element-wise Coordinate Descent)。**
    * 每次只优化一个阵元，遍历其 $K$ 种可能状态。复杂度降至 $\mathcal{O}(KN)$，极度适合 6G 实时系统硬件实现。

---

## 三、 为什么 SDR 是 Benchmark 而不是穷举法？

> [!faq] 提问 3：为什么这里的 Benchmark 不可以使用穷举法得到？为什么要用 SDR？

1.  **不可计算的维度灾难：** 穷举法的复杂度是 $\mathcal{O}(K^N)$。在 $N$ 稍大（如 64）时，计算量达到 $10^{38}$ 量级，物理意义上无法算完，失去了作为 Benchmark 的可复现性。$K$ 代表的是：单个天线阵元（或超表面单元）可以选择的离散物理状态的总数。
2.  **SDR 提供严格的“理论上界 (Upper Bound)”：** SDR 丢弃了秩一约束，相当于扩大了搜索空间。在更大空间找到的最大值必定 $\ge$ 原问题的最大值。这为低复杂度算法提供了一面绝对的镜子。
3.  **性能逼近：** SDR 配合高斯随机化得到的传统解，距离全局最优解非常近。

---

## 四、 SDR 的核心机制与物理意义深度剖析

### 1. 升维的本质：二次型 $\rightarrow$ 一次型

> [!faq] 提问 4：为什么把向量升维成矩阵，就可以把非凸问题变成凸问题？直接用二次型不行吗（二次函数也是凸函数）？

* **二次型的非凸性：** 多维二次型 $\mathbf{x}^H \mathbf{A} \mathbf{x}$ 的凸凹性取决于 $\mathbf{A}$ 的特征值。在阵列处理中，$\mathbf{A}$ 常为不定矩阵（特征值有正有负），形状如马鞍，到处都是局部极值。
* **最大化凸函数的矛盾：** 即使二次型是完美的凸函数，由于我们的目标通常是**最大化 (Max)** 性能，在凸函数里找最大值也是 NP-Hard 的非凸问题。
* **降维打击：** 引入 $\mathbf{X} = \mathbf{x}\mathbf{x}^H$ 后，$\mathbf{x}^H \mathbf{A} \mathbf{x} = \text{Tr}(\mathbf{A} \mathbf{X})$。无论 $\mathbf{A}$ 内部结构多复杂，$\text{Tr}(\mathbf{A} \mathbf{X})$ 作为关于 $\mathbf{X}$ 的一次函数（仿射函数），**永远是一块绝对平坦的倾斜面**，从而彻底消除了目标函数的非凸性。

### 2. 内积与外积的物理意义

> [!faq] 提问 5：向量内积外积的物理意义

* **内积 $\mathbf{x}^H \mathbf{x}$ (标量)：** 算的是**总能量/总功率**。它抹平了阵列的微观结构。
* **外积 $\mathbf{X} = \mathbf{x}\mathbf{x}^H$ (方阵)：** 算的是**瞬时空间相关矩阵**。
    * 对角线元素：各阵元的独立功率。
    * 非对角线元素：阵元间的互相关（包含了至关重要的**相位差**信息）。它将阵元间的协同关系直接展开在了二维矩阵明面上。

### 3. Trace (迹) 与 Rank (秩) 的左右手互搏

> [!faq] 提问 6：使用 $\text{Tr}$ 和 $\text{Rank}$ 的区别和意义是什么

* **换位置的原因：** 矩阵迹的循环移位性质 $\text{Tr}(\mathbf{A}\mathbf{B}\mathbf{C}) = \text{Tr}(\mathbf{C}\mathbf{A}\mathbf{B})$。借此将 $\mathbf{x}^H$ 和 $\mathbf{x}$ 拼装成外积矩阵 $\mathbf{X}$。
* **$\text{Tr}(\mathbf{A}\mathbf{X})$ 的物理意义：** 计算空间结构的内积。如 $\mathbf{A}$ 为干扰协方差，则它代表当前波束状态 $\mathbf{X}$ 下的**总接收干扰功率**。
* **分工明确：**
    * **Trace** 负责把**目标函数**拉平，变成凸的。
    * **Rank** 负责把守**物理规律的底线**。

### 4. 为什么会有 Rank-1 的约束？

> [!faq] 提问 7：为什么会有这个 $\text{rank}(\mathbf{X})=1$ 的约束？

由 $\mathbf{X} = \mathbf{x}\mathbf{x}^H$ 展开可知，矩阵 $\mathbf{X}$ 的每一列都是基础向量 $\mathbf{x}$ 的标量倍数，即所有列向量完全线性相关。因此，只要 $\mathbf{X}$ 是由一维向量外积生成的，它的秩必定为 1。
这是**升维与降维等价的锁死条件**。SDR 的松弛就是强行砸掉这个非凸的枷锁，换取问题的可解性。
#### 例子
##### 1. 拆解矩阵 $\mathbf{X}$ 的内部结构

假设你有一个 $N \times 1$ 的列向量 $\mathbf{x}$（例如代表 $N$ 个阵元的相位）：

$$\mathbf{x} = \begin{bmatrix} x_1 \\ x_2 \\ \vdots \\ x_N \end{bmatrix}$$

它的共轭转置 $\mathbf{x}^H$ 是一个 $1 \times N$ 的行向量：

$$\mathbf{x}^H = \begin{bmatrix} x_1^* & x_2^* & \dots & x_N^* \end{bmatrix}$$

当我们把它们乘在一起得到矩阵 $\mathbf{X}$ 时，根据矩阵乘法法则：

$$\mathbf{X} = \mathbf{x}\mathbf{x}^H = \begin{bmatrix} x_1 \\ x_2 \\ \vdots \\ x_N \end{bmatrix} \begin{bmatrix} x_1^* & x_2^* & \dots & x_N^* \end{bmatrix}$$

展开后，你会得到一个 $N \times N$ 的方阵：

$$\mathbf{X} = \begin{bmatrix} x_1 x_1^* & x_1 x_2^* & \dots & x_1 x_N^* \\ x_2 x_1^* & x_2 x_2^* & \dots & x_2 x_N^* \\ \vdots & \vdots & \ddots & \vdots \\ x_N x_1^* & x_N x_2^* & \dots & x_N x_N^* \end{bmatrix}$$

##### 2. 为什么它的秩是 1？

在线性代数中，**矩阵的秩 (Rank) 等于它所包含的“线性无关”的列向量（或行向量）的最大数量。**

现在，我们仔细观察上面展开的方阵 $\mathbf{X}$ 的每一列：

- **第 1 列** 是：$\begin{bmatrix} x_1 x_1^* \\ x_2 x_1^* \\ \vdots \\ x_N x_1^* \end{bmatrix} = x_1^* \begin{bmatrix} x_1 \\ x_2 \\ \vdots \\ x_N \end{bmatrix} = x_1^* \mathbf{x}$
    
- **第 2 列** 是：$\begin{bmatrix} x_1 x_2^* \\ x_2 x_2^* \\ \vdots \\ x_N x_2^* \end{bmatrix} = x_2^* \begin{bmatrix} x_1 \\ x_2 \\ \vdots \\ x_N \end{bmatrix} = x_2^* \mathbf{x}$
    
- ...
    
- **第 $N$ 列** 是：$x_N^* \mathbf{x}$
    

**破案了！** 矩阵 $\mathbf{X}$ 的每一列，实际上**全部都是同一个基础向量 $\mathbf{x}$ 的标量倍数**。

它们在几何空间中指向完全相同的方向（或者反方向），只是长度不同而已。它们之间存在极强的线性相关性。

因为这里面只有**唯一一个**独立的方向（即 $\mathbf{x}$ 的方向），所以这个矩阵的线性无关列数只有 1，即 **$\text{rank}(\mathbf{X}) = 1$**（假设 $\mathbf{x}$ 不是全零向量）。

### 5. 高斯随机化 (Gaussian Randomization)

> [!faq] 提问 8：高斯随机化的意义

由于 SDR 丢弃了 Rank-1 约束，算出的最优矩阵 $\mathbf{X}^*$ 通常秩 $> 1$，无法直接还原为单个向量解。
**高斯随机化相当于“盲狙 + 强制归位”：**
1.  **视作模具：** 将 $\mathbf{X}^*$ 视为多元复高斯分布的协方差矩阵。
2.  **批量生成：** 根据该分布生成大量（如 $M=1000$ 个）连续的随机高斯向量样本。
3.  **投影量化：** 提取这些样本的相位，强制“吸附”到最近的合法离散集合 $\mathcal{K}$ 中。
4.  **择优录取：** 将所有合法的样本代入原目标函数比对，选出性能最高的一个作为最终解。
5. 