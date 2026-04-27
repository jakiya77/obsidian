维纳–霍夫方程（Wiener-Hopf Equation）的推导是统计信号处理中的经典内容。要推导这个最优解，通常有两种方法：**代数展开求导法**和**正交原理（Orthogonality Principle）**。

为了清晰起见，我们以实数域的有限长冲激响应（FIR）滤波器为例进行推导。

### 1. 变量与符号定义

假设维纳滤波器是一个长度为 $M$ 的 FIR 滤波器：

- **输入信号向量**：$\mathbf{x}[n] = [x[n], x[n-1], \dots, x[n-M+1]]^T$
    
- **滤波器抽头系数向量**：$\mathbf{h} = [h_0, h_1, \dots, h_{M-1}]^T$
    
- **滤波器输出**：$y[n] = \mathbf{h}^T \mathbf{x}[n] = \mathbf{x}^T[n] \mathbf{h}$
    
- **目标信号**：$d[n]$
    
- **估计误差**：$e[n] = d[n] - y[n] = d[n] - \mathbf{h}^T \mathbf{x}[n]$
    

---

### 推导方法一：代数求导法（最小化代价函数）

维纳滤波器的目标是寻找一组权重 $\mathbf{h}$，使得均方误差（MSE）最小化。我们将代价函数定义为 $J(\mathbf{h})$。

**第一步：展开代价函数**

$$J(\mathbf{h}) = E\{e^2[n]\} = E\{(d[n] - \mathbf{h}^T \mathbf{x}[n])^2\}$$

将平方项展开：

$$J(\mathbf{h}) = E\{d^2[n] - 2d[n]\mathbf{x}^T[n]\mathbf{h} + \mathbf{h}^T\mathbf{x}[n]\mathbf{x}^T[n]\mathbf{h}\}$$

**第二步：引入期望算子**

因为期望算子 $E\{\cdot\}$ 是线性的，且滤波器系数 $\mathbf{h}$ 是确定性向量（非随机变量），我们可以将期望符号移入括号内：

$$J(\mathbf{h}) = E\{d^2[n]\} - 2E\{d[n]\mathbf{x}^T[n]\}\mathbf{h} + \mathbf{h}^T E\{\mathbf{x}[n]\mathbf{x}^T[n]\} \mathbf{h}$$

**第三步：代入统计矩阵定义**

根据平稳随机信号的统计特性，我们定义：

- 目标信号的方差（功率）：$\sigma_d^2 = E\{d^2[n]\}$
    
- 输入信号的自相关矩阵：$\mathbf{R}_{xx} = E\{\mathbf{x}[n]\mathbf{x}^T[n]\}$
    
- 输入信号与目标信号的互相关向量：$\mathbf{r}_{xd} = E\{\mathbf{x}[n]d[n]\}$ （注意 $E\{d[n]\mathbf{x}^T[n]\} = \mathbf{r}_{xd}^T$）
    

将这些定义代入代价函数，得到一个关于 $\mathbf{h}$ 的二次型函数：

$$J(\mathbf{h}) = \sigma_d^2 - 2\mathbf{r}_{xd}^T\mathbf{h} + \mathbf{h}^T\mathbf{R}_{xx}\mathbf{h}$$

**第四步：对向量求导并令其为零**

为了找到使 $J(\mathbf{h})$ 最小的最优向量 $\mathbf{h}_{opt}$，我们对 $\mathbf{h}$ 求梯度（偏导数），并将其置为零向量 $\mathbf{0}$。

根据矩阵求导法则 $\frac{\partial (\mathbf{a}^T\mathbf{x})}{\partial \mathbf{x}} = \mathbf{a}$ 以及 $\frac{\partial (\mathbf{x}^T\mathbf{A}\mathbf{x})}{\partial \mathbf{x}} = 2\mathbf{A}\mathbf{x}$（当 $\mathbf{A}$ 为对称矩阵时）：

$$\nabla_{\mathbf{h}} J(\mathbf{h}) = -2\mathbf{r}_{xd} + 2\mathbf{R}_{xx}\mathbf{h} = \mathbf{0}$$

化简后即可得到**维纳-霍夫方程**：

$$\mathbf{R}_{xx}\mathbf{h}_{opt} = \mathbf{r}_{xd}$$

如果自相关矩阵 $\mathbf{R}_{xx}$ 是非奇异的（可逆），最优解即为 $\mathbf{h}_{opt} = \mathbf{R}_{xx}^{-1}\mathbf{r}_{xd}$。

---

### 推导方法二：正交原理（更具物理直觉的推导）

在统计估计理论中，**正交原理**指出：**当均方误差达到最小时，最优的估计误差 $e[n]$ 必定与所有用于估计的输入数据 $\mathbf{x}[n]$ 正交。**

在统计学中，“正交”意味着两者的互相关为零。用数学公式表达即为：

$$E\{e[n]\mathbf{x}[n]\} = \mathbf{0}$$

将误差的定义 $e[n] = d[n] - \mathbf{x}^T[n]\mathbf{h}$ 代入上式：

$$E\{(d[n] - \mathbf{x}^T[n]\mathbf{h})\mathbf{x}[n]\} = \mathbf{0}$$

展开期望项：

$$E\{d[n]\mathbf{x}[n]\} - E\{\mathbf{x}[n]\mathbf{x}^T[n]\}\mathbf{h} = \mathbf{0}$$

直接代入 $\mathbf{R}_{xx}$ 和 $\mathbf{r}_{xd}$ 的定义：

$$\mathbf{r}_{xd} - \mathbf{R}_{xx}\mathbf{h} = \mathbf{0} \quad \implies \quad \mathbf{R}_{xx}\mathbf{h} = \mathbf{r}_{xd}$$

通过正交原理，我们用极简的步骤直接得出了同样的维纳-霍夫方程。这种推导方式不仅在数学上更简洁，而且深刻揭示了最优线性估计的几何意义：**最优估计本质上是目标信号在输入信号张成的子空间上的投影**。

