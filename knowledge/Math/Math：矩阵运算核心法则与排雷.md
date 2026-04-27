---
tags:
  - 矩阵论
  - 最小二乘法
  - 算法推导
---


---

## 二、 矩阵运算核心法则与排雷

### 1. 默认维度规矩
在工程和数学中，**所有的向量默认都是列向量**。
例如误差向量 $\mathbf{e}$ 包含 3 个元素，则它的维度是 $3 \times 1$。

### 2. 复数域必须用共轭转置 ($H$)
*   实数域（如普通机器学习预测）：用转置 $T$。
*   复数域（数字基带中的 $I+jQ$ 信号）：**必须**用共轭转置 $H$（Hermitian）。求复数向量长度的平方，定义为它自身的共轭转置乘以它自己。

### 3. 误差能量计算：为什么是 $\mathbf{e}^H \mathbf{e}$ 而不是 $\mathbf{e} \mathbf{e}^H$？
要计算误差的“平方和”（能量），结果必须是一个**标量（单一数字）**。

*   **正确做法（内积）：$\mathbf{e}^H \mathbf{e}$**
    *   维度变化：$(1 \times 3) \times (3 \times 1) \rightarrow (1 \times 1)$。
    *   结果是标量 $\sum |e_i|^2$，完美代表总误差能量，可进行求导求极值。
*   **错误做法（外积）：$\mathbf{e} \mathbf{e}^H$**
    *   维度变化：$(3 \times 1) \times (1 \times 3) \rightarrow (3 \times 3)$。
    *   结果是一个协方差矩阵，无法对一个矩阵求“最小值”。

---

## 三、 矩阵求导“三大字典”（类比一元微积分）

将标量求导规则平移到矩阵求导，只需记住以下三条（设 $\mathbf{w}$ 为未知列向量，$\mathbf{c}$ 为常数向量，$\mathbf{R}$ 为常数方阵）：

| 对应项 | 标量求导 (对 $w$) | 矩阵求导 (对 $\mathbf{w}$) | 说明 |
| :--- | :--- | :--- | :--- |
| **常数项** | $c \rightarrow 0$ | $\frac{\partial}{\partial \mathbf{w}} (\mathbf{c}^H \mathbf{c}) = \mathbf{0}$ | 不含变量的项求导直接为零。 |
| **一次项** | $cw \rightarrow c$ | $\frac{\partial}{\partial \mathbf{w}} (\mathbf{c}^H \mathbf{w}) = \mathbf{c}$ | **注：** 这里看似 $\mathbf{c}^H$ 的 $H$ 不见了，是因为结果必须按未知数 $\mathbf{w}$ 的列向量形状重新排列，行变回了列。 |
| **二次项** | $aw^2 \rightarrow 2aw$ | $\frac{\partial}{\partial \mathbf{w}} (\mathbf{w}^H \mathbf{R} \mathbf{w}) = 2\mathbf{R}\mathbf{w}$ | 仅当 $\mathbf{R}$ 是 Hermitian 矩阵（或实对称矩阵）时成立。 |

---

## 四、 最小二乘闭式解推导全过程

由于在数字基带中，权重 $\mathbf{w}$ 没有物理天线硬件（如移相器）的幅度上限约束，这是一个**无约束优化问题**，可以直接通过令导数为零求全局最优解。

设标准方程为 $\mathbf{A}\mathbf{w} = \mathbf{b}$。

**步骤 1：构建目标函数（均方误差）**
$$J(\mathbf{w}) = \|\mathbf{A}\mathbf{w} - \mathbf{b}\|_2^2 = (\mathbf{A}\mathbf{w} - \mathbf{b})^H (\mathbf{A}\mathbf{w} - \mathbf{b})$$

**步骤 2：像多项式一样展开**
利用 $(\mathbf{A}\mathbf{B})^H = \mathbf{B}^H \mathbf{A}^H$ 展开，并将中间两个相等的标量项合并：
$$J(\mathbf{w}) = (\mathbf{w}^H\mathbf{A}^H - \mathbf{b}^H) (\mathbf{A}\mathbf{w} - \mathbf{b})$$
$$J(\mathbf{w}) = \mathbf{w}^H(\mathbf{A}^H\mathbf{A})\mathbf{w} - 2(\mathbf{A}^H\mathbf{b})^H\mathbf{w} + \mathbf{b}^H\mathbf{b}$$
这完全对应高中代数里的 $a^2w^2 - 2abw + b^2$
> [!warning]- 中间两项的处理 
  中间两项 $-\mathbf{w}^T\mathbf{A}^T\mathbf{b}$ 和 $-\mathbf{b}^T\mathbf{A}\mathbf{w}$，它们相乘后的结果其实是一个 $1 \times 1$ 的标量（也就是一个普通的数字）。**一个数字的转置等于它自己**，所以这两项其实是相等的。我们可以把它们合并成一项：
$$J(\mathbf{w}) = \mathbf{w}^T(\mathbf{A}^T\mathbf{A})\mathbf{w} - 2(\mathbf{A}^T\mathbf{b})^T\mathbf{w} + \mathbf{b}^T\mathbf{b}$$

- 二次项：$\mathbf{w}^T(\mathbf{A}^T\mathbf{A})\mathbf{w}$
    
- 一次项：$- 2(\mathbf{A}^T\mathbf{b})^T\mathbf{w}$
    
- 常数项：$\mathbf{b}^T\mathbf{b}$*

**步骤 3：套用字典对 $\mathbf{w}$ 求偏导**
*   二次项 $\mathbf{w}^H(\mathbf{A}^H\mathbf{A})\mathbf{w}$ 导数为 $2(\mathbf{A}^H\mathbf{A})\mathbf{w}$
*   一次项 $- 2(\mathbf{A}^H\mathbf{b})^H\mathbf{w}$ 导数为 $- 2(\mathbf{A}^H\mathbf{b})$
*   常数项 $\mathbf{b}^H\mathbf{b}$ 导数为 $0$

$$\frac{\partial J}{\partial \mathbf{w}} = 2\mathbf{A}^H\mathbf{A}\mathbf{w} - 2\mathbf{A}^H\mathbf{b}$$

**步骤 4：令导数为 0（法线方程）并求解**
$$2\mathbf{A}^H\mathbf{A}\mathbf{w} - 2\mathbf{A}^H\mathbf{b} = 0$$
$$\mathbf{A}^H\mathbf{A}\mathbf{w} = \mathbf{A}^H\mathbf{b}$$
$$\mathbf{w} = (\mathbf{A}^H\mathbf{A})^{-1}\mathbf{A}^H\mathbf{b}$$

**结论：** 
由于矩阵伪逆（Moore-Penrose Pseudoinverse）的数学定义恰好为 $\mathbf{A}^{\dagger} = (\mathbf{A}^H \mathbf{A})^{-1} \mathbf{A}^H$。
因此，最终的最优闭式解为：
$$\mathbf{w} = \mathbf{A}^{\dagger}\mathbf{b}$$