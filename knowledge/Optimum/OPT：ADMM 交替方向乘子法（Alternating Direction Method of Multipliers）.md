
---
## 恒模约束波束赋形的 ADMM 优化笔记 admm

### 1. 核心矛盾与直觉推导 (Split-and-Conquer)

我们要解决的问题是：在满足**所有天线功率相等**（恒模约束）的前提下，让**波束指向目标方向**（最大化信号功率）。

- **目标函数**：$\max_{\mathbf{w}} \mathbf{w}^H \mathbf{H} \mathbf{w}$ （其中 $\mathbf{H} = \mathbf{a}\mathbf{a}^H$）
    
- **物理约束**：$|w_n| = 1, \forall n$ （非凸，很难直接求导）[[OPT：非凸函数合集]]
    

**直觉推导：**

既然一个变量 $\mathbf{w}$ 很难同时处理目标函数和硬性约束，我们就引入一个替身变量 $\mathbf{v}$。

1. 让 $\mathbf{w}$ 去负责追逐最大的信号功率。
    
2. 让 $\mathbf{v}$ 去负责死守恒模约束（住在单位圆上）。
    
3. 通过一个**等式约束 $\mathbf{w} = \mathbf{v}$** 把它们强行捆绑在一起。
    

---

### 2. 增广拉格朗日函数 (从软约束到硬惩罚)

为了求解这个捆绑问题，我们需要构造增广拉格朗日函数。相比于传统乘子法，它增加了一个**二次惩罚项**。

$$L_{\rho}(\mathbf{w}, \mathbf{v}, \mathbf{\lambda}) = \underbrace{-\mathbf{w}^H \mathbf{H} \mathbf{w}}_{\text{目标函数}} + \underbrace{\text{Re}\{\mathbf{\lambda}^H(\mathbf{w} - \mathbf{v})\}}_{\text{传统线性约束}} + \underbrace{\frac{\rho}{2} \|\mathbf{w} - \mathbf{v}\|^2}_{\text{二次惩罚项 (电网)}}$$

- **$\rho$ 的物理意义**：它像是一根橡皮筋。$\rho$ 越大，橡皮筋越紧，强制要求 $\mathbf{w}$ 必须等于 $\mathbf{v}$。
    
- **稳定性保证**：二次项让函数表面变得更圆润（强凸性），保证了交替优化时每一步都有稳定的解。
    

---

### 3. 为了简化计算的方案：缩放形式 (Scaled Form)

在写代码时，处理 $\mathbf{\lambda}^H(\mathbf{w} - \mathbf{v})$ 和 $\frac{\rho}{2} \|\mathbf{w} - \mathbf{v}\|^2$ 两项非常繁琐。我们利用**配方法**引入**缩放对偶变量 $\mathbf{u} = \mathbf{\lambda} / \rho$**。

**变换过程：**

通过初中代数的完全平方式原理：

$$\text{一次项} + \text{二次项} \Rightarrow \frac{\rho}{2} \|\mathbf{w} - \mathbf{v} + \mathbf{u}\|^2 - \text{常数}$$

**最终代码所依据的公式：**

$$\mathcal{L}(\mathbf{w}, \mathbf{v}, \mathbf{u}) = -\mathbf{w}^H \mathbf{H} \mathbf{w} + \frac{\rho}{2} \|\mathbf{w} - \mathbf{v} + \mathbf{u}\|^2$$

_注：常数项不影响求导，直接舍弃。_

---

### 4. 方案对比：为什么选缩放形式？

|**维度**|**传统形式 (Unscaled λ)**|**缩放形式 (Scaled u)**|
|---|---|---|
|**公式简洁度**|包含线性项和二次项，较冗长|只有一个紧凑的二次平方项|
|**更新 $\mathbf{w}$ 的难度**|导数包含 $\lambda$ 和 $\rho$，易写错正负号|导数极简，就是典型的最小二乘形式|
|**更新对偶变量**|$\lambda = \lambda + \rho(w - v)$|$u = u + (w - v)$ (直接累加残差)|
|**工程直观性**|抽象的拉格朗日乘子|直观的“历史误差累积”|

---

### 5. ADMM 交替优化三步走 (代码逻辑)

在每一轮迭代中，我们依次执行：

1. **更新 $\mathbf{w}$ (固定 $\mathbf{v}, \mathbf{u}$)**：
    
    求解一个二次型问题。
    
    $$\mathbf{w}^{k+1} = \left( \frac{\rho}{2}\mathbf{I} - \mathbf{H} \right)^{-1} \frac{\rho}{2}(\mathbf{v}^k - \mathbf{u}^k)$$
    
    _关键：$\rho$ 必须足够大（$\rho > 2\lambda_{\max}(\mathbf{H})$），否则矩阵求逆会发散，导致波束对不齐。_
    
2. **更新 $\mathbf{v}$ (固定 $\mathbf{w}, \mathbf{u}$)**：
    
    将 $\mathbf{w} + \mathbf{u}$ 投影到单位圆上。
    
    $$\mathbf{v}^{k+1} = \exp(j \cdot \text{angle}(\mathbf{w}^{k+1} + \mathbf{u}^k))$$
    
3. **更新 $\mathbf{u}$ (累加残差)**：
    
    更新账本，记录这一轮 $\mathbf{w}$ 和 $\mathbf{v}$ 还有多少差距。
    
    $$\mathbf{u}^{k+1} = \mathbf{u}^k + \mathbf{w}^{k+1} - \mathbf{v}^{k+1}$$
    

---

### 总结

ADMM 的精髓在于**“把约束当做惩罚，把复杂交给迭代”**。通过引入 $u$，我们将复杂的波束赋形约束问题变成了两步极其简单的计算：一次**矩阵求逆**（对应 $\mathbf{w}$）和一次**相位提取**（对应 $\mathbf{v}$）。

这份笔记能帮你理清思路吗？如果你想深入了解那个关于 $\rho$ 必须大于特征值的稳定性条件，我可以再为你详细拆解一下那个海森矩阵的判别过程。

## 仿真结果

当设置$\rho = 2$ 的时候会得到如下结果：

| ![[Pasted image 20260304174653.png]] | ![[Pasted image 20260304174707.png]] |
| ------------------------------------ | ------------------------------------ |

出现这个结果的原因是：
在 ADMM 算法中，$\rho$（惩罚参数）不仅决定了收敛速度，更决定了算法在处理非凸问题（如恒模波束赋形）时的**稳定性**。

你之前看到的“对不齐”现象，本质上是由于 $\rho$ 取得太小，导致 $w$ 的子优化问题从“找最小值”变成了“找最大值”，从而导致算法发散。

以下是选择 $\rho$ 的原理与计算方法：

### 1. 核心原理：保证海森矩阵（Hessian）的正定性

在更新 $w$ 的步骤中，我们实际上是在最小化增广拉格朗日函数关于 $w$ 的部分。对于你的目标函数 $-\mathbf{w}^H \mathbf{H} \mathbf{w}$，加上惩罚项后的目标函数为：

$$f(w) = -\mathbf{w}^H \mathbf{H} \mathbf{w} + \frac{\rho}{2} \|\mathbf{w} - \mathbf{v} + \mathbf{u}\|^2$$

要让这个函数有**唯一的最小值**（即抛物面开口向上），它的二阶导数矩阵（海森矩阵）必须是**正定**的。

该函数的海森矩阵计算如下：

$$\nabla^2 f(w) = \rho \mathbf{I} - 2\mathbf{H}$$
##### 第一步：写出仅关于 $\mathbf{w}$ 的目标函数

在 ADMM 更新 $\mathbf{w}$ 的这一步，我们将 $\mathbf{v}$ 和 $\mathbf{u}$ 视为常数。目标函数为：

$$f(\mathbf{w}) = -\mathbf{w}^H \mathbf{H} \mathbf{w} + \frac{\rho}{2} \|\mathbf{w} - \mathbf{v} + \mathbf{u}\|^2$$

为了方便推导，我们令常数项 $\mathbf{c} = \mathbf{v} - \mathbf{u}$，函数可以简写为：

$$f(\mathbf{w}) = -\mathbf{w}^H \mathbf{H} \mathbf{w} + \frac{\rho}{2} \|\mathbf{w} - \mathbf{c}\|^2$$

##### 第二步：展开二范数平方项

向量的二范数平方 $\|\mathbf{x}\|^2$ 等于向量自己的共轭转置乘自己 $\mathbf{x}^H \mathbf{x}$。我们将后面的惩罚项展开：

$$\|\mathbf{w} - \mathbf{c}\|^2 = (\mathbf{w} - \mathbf{c})^H (\mathbf{w} - \mathbf{c})$$

$$= (\mathbf{w}^H - \mathbf{c}^H) (\mathbf{w} - \mathbf{c})$$

$$= \mathbf{w}^H \mathbf{w} - \mathbf{w}^H \mathbf{c} - \mathbf{c}^H \mathbf{w} + \mathbf{c}^H \mathbf{c}$$

为了把它和前面的 $-\mathbf{w}^H \mathbf{H} \mathbf{w}$ 统一起来，我们可以把 $\mathbf{w}^H \mathbf{w}$ 写成带有单位阵 $\mathbf{I}$ 的形式，即 $\mathbf{w}^H \mathbf{I} \mathbf{w}$。

现在，把展开后的结果代回原函数：

$$f(\mathbf{w}) = -\mathbf{w}^H \mathbf{H} \mathbf{w} + \frac{\rho}{2} (\mathbf{w}^H \mathbf{I} \mathbf{w} - \mathbf{w}^H \mathbf{c} - \mathbf{c}^H \mathbf{w} + \mathbf{c}^H \mathbf{c})$$

##### 第三步：合并并提取二次项

在微积分中，决定一个函数凹凸性（曲率）的只有**二次项**。一次项（包含一个 $\mathbf{w}$ 的项）求二次导数会变成 $0$，常数项求一次导数就会变成 $0$。

所以，我们只需要盯着带有 $\mathbf{w}^H \dots \mathbf{w}$ 的项：

$$f_{\text{quad}}(\mathbf{w}) = -\mathbf{w}^H \mathbf{H} \mathbf{w} + \frac{\rho}{2} \mathbf{w}^H \mathbf{I} \mathbf{w}$$

提取公因子 $\mathbf{w}^H$ 和 $\mathbf{w}$：

$$f_{\text{quad}}(\mathbf{w}) = \mathbf{w}^H \left( \frac{\rho}{2} \mathbf{I} - \mathbf{H} \right) \mathbf{w}$$

这就是这个目标函数的“核心曲率矩阵”（我们定义它为 $\mathbf{M}$）：

$$\mathbf{M} = \frac{\rho}{2} \mathbf{I} - \mathbf{H}$$
_(注：在某些复数导数定义下，常数系数略有不同，但核心逻辑一致：$\rho$ 必须压制住 $\mathbf{H}$)_

- **如果 $\rho$ 太小**：$\rho \mathbf{I} - 2\mathbf{H}$ 会出现负特征值。此时优化问题变成了在一个“开口向下”的曲面上找最低点，算法会迅速滑向无穷远，导致结果完全畸变。
    
- **如果 $\rho$ 足够大**：矩阵变为正定，确保每次迭代 $w$ 都能稳步向正确的物理方向（主瓣方向）靠拢。
    

### 2. 如何计算 $\rho$？

理论上的临界值取决于矩阵 $\mathbf{H}$ 的**最大特征值** $\lambda_{\max}(\mathbf{H})$。

#### 步骤 A：分析矩阵特性

在波束赋形中，$\mathbf{H} = \mathbf{a}\mathbf{a}^H$，这是一个秩为 1 的矩阵。

对于这种特殊矩阵，它的最大特征值等于导向矢量 $\mathbf{a}$ 的模长的平方：

$$\lambda_{\max}(\mathbf{H}) = \|\mathbf{a}\|_2^2$$

#### 步骤 B：设定安全阈值

为了保证算法收敛且稳定，通常要求：

$$\rho > 2 \cdot \lambda_{\max}(\mathbf{H})$$

或者更严谨地，在复数域 ADMM 常用形式中：

$$\rho > \text{谱范数}(\mathbf{H})$$

#### 步骤 C：工程经验公式

在实际写程序时，为了留出足够的安全裕量（防止噪声或舍入误差干扰），通常会设置一个系数：

$$\rho = (1.5 \sim 2.5) \times \|\mathbf{a}\|_2^2$$

在之前的代码修正中，我建议使用 `rho = 2.1 * norm(steering_theta)^2`。

- `norm(steering_theta)^2` 计算的是天线阵列的总能量（对于 $N=10$ 的阵列，这个值大约在 $6.4 \sim 10$ 之间）。
    
- 乘以 `2.1` 确保了 $\rho$ 绝对大于特征值，从而使 $w$ 的子问题始终是稳定的凸优化。
    
- [[Math：谱范数（Spectral Norm）和最大特征值（Maximum Eigenvalue）]]
### 3. $\rho$ 选取的利弊权衡

- **$\rho$ 选得接近临界值（较小但合法）**：
    
    - **优点**：算法对目标函数（最大化功率）非常敏感，收敛到高增益点的速度快。
        
    - **缺点**：容易产生振荡，稳定性边缘游走。
        
- **$\rho$ 选得很大（如 100 倍特征值）**：
    
    - **优点**：极其稳定，$w$ 会非常死板地跟着 $v$（恒模约束）走。
        
    - **缺点**：收敛非常缓慢，可能需要成千上万次迭代才能对准主瓣。
        

### 总结

计算一个好的 $\rho$ 的标准步骤是：

1. 计算导向矢量的能量 $P = \text{norm}(\text{steering\_vector})^2$。
    
2. 将 $\rho$ 设置为 $2 \times P$ 左右的数值。
    
3. 如果发现收敛曲线（Value）左右乱跳，说明 $\rho$ 还是小了，需要继续调大。
    

| $\rho = 3P = 25.6323$                                                  | ![[Pasted image 20260304201318.png]] | ![[Pasted image 20260304201305.png]] |
| ---------------------------------------------------------------------- | ------------------------------------ | ------------------------------------ |
| $\rho = 2P = 16.85$   警告: 矩阵接近奇异值，或者缩放不良。结果可能不准确。RCOND = 7.161115e-19。 | ![[Pasted image 20260304201532.png]] | ![[Pasted image 20260304201514.png]] |
| $\rho = 20$                                                            | ![[Pasted image 20260304203434.png]] | ![[Pasted image 20260304203419.png]] |

