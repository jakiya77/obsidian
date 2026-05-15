## 1. 物理架构
![[png：Dynamic Metasurface Antennas for  6G Extreme Massive MIMO Communications.png]]横向一排波导上嵌有若干个element，每一个波导连接着一个RF chain，设计每一个波导由电机驱动可以左右移动，即 $x_{n,m} = u_n + d_m$。
![[png：Modeling and Performance Analysis for Movable Antenna Enabled Wireless Communications.png]]
![[png：Movable-Element RIS-Aided Wireless Communications ：An Element-Wise Position Optimization Approach.png|452]]
上图分别是传统的可移动天线架构和可移动RIS架构。可以观察到早期全数字 MA 架构需要再每一个element上连接RF chain，不但面临高额的RF chain开销，还会存在每个天线独立 2D 运动所需的复杂机械解耦。而新型的ME-RIS架构不需要逐元素连接RF chain，只连接控制线，只能改变信号的传播路径，无法在本地进行任何信号剥离或自适应滤波。于是引入M-DMA架构，波导管作为信号传输与合成的物理媒介，实现对信号的波域处理和数字域处理。
 
## 2. 移动性的引入和过载零陷分析
$$K \le \frac{N_{\text{RF}}(N_M + 3)}{2} - 1$$

## 3. 优化问题 基于 AO 与流形优化的 M-DMA 联合抗干扰算法

### 1. 算法核心思想 (AO Framework)

总体目标是最大化接收端的 SINR（或等效为最小化残留干扰功率）。由于目标函数关于三个变量 $\mathbf{w}$ (数字权重), $\mathbf{v}$ (超表面相位向量, $v_m = e^{j\phi_m}$), 和 $\mathbf{x}$ (物理位置) 是高度非凸且深度耦合的，我们采用交替优化（AO）将原问题解耦为三个子问题。

**AO 迭代逻辑：**

- **步骤 1**: 固定 $\mathbf{v}$ 和 $\mathbf{x}$，求解最优的 $\mathbf{w}$（存在闭式解）。
    
- **步骤 2**: 固定 $\mathbf{w}$ 和 $\mathbf{x}$，使用**流形优化 (MO)** 求解最优的相位 $\mathbf{v}$。
    
- **步骤 3**: 固定 $\mathbf{w}$ 和 $\mathbf{v}$，求解最优的位置 $\mathbf{x}$。
    
- 不断循环以上三步，直到目标函数收敛。
    

---

### 2. 子问题拆解与求解策略

#### 子问题 1：优化数字权重 $\mathbf{w}$ (MVDR 闭式解)

当 DMA 阵元的相位 $\mathbf{v}$ 和波导管位置 $\mathbf{x}$ 固定时，等效的基带干扰信道矩阵 $\mathbf{H}_J$ 和目标信道 $\mathbf{h}_d$ 都是已知的常量。

这个问题退化为经典的 MVDR 波束赋形问题。

- **优化目标**: 最小化干扰加噪声功率 $\mathbf{w}^H \mathbf{R}_{J+N} \mathbf{w}$，受限于无失真响应约束 $\mathbf{w}^H \mathbf{h}_d = 1$。
    
- **最优解 (闭式解)**:
    
    $$\mathbf{w}_{opt} = \frac{\mathbf{R}_{J+N}^{-1} \mathbf{h}_d}{\mathbf{h}_d^H \mathbf{R}_{J+N}^{-1} \mathbf{h}_d}$$
    
    _(注：这一步计算复杂度主要在协方差矩阵的求逆上。)_
    

---

#### 子问题 2：优化表面相位 $\mathbf{v}$ (核心：流形优化)

固定 $\mathbf{w}$ 和 $\mathbf{x}$ 后，我们需要优化相位。我们将 $N_{\mathrm{RF}} \times N_M$ 个相位组合成一个复数向量 $\mathbf{v}$。

此时的优化目标可以转化为一个二次型问题：

$$\min_{\mathbf{v}} \quad \mathbf{v}^H \mathbf{\Phi} \mathbf{v} - 2\Re\{\mathbf{v}^H \mathbf{u}\}$$

$$\text{s.t.} \quad |v_i| = 1, \quad \forall i$$

由于 $|v_i| = 1$ 定义了一个**复圆流形 (Complex Circle Manifold, CCM)** $\mathcal{M} = \{ \mathbf{v} \in \mathbb{C}^N : |v_i| = 1 \}$，这不再是一个欧式空间，传统梯度下降会破坏约束。我们引入流形优化。

> [!important] 流形优化 (Riemannian Manifold Optimization) 的标准三步曲
> 
> 1. **欧氏梯度 (Euclidean Gradient)**：
>     
>     先计算目标函数在无约束欧氏空间下的梯度：$\nabla_{\mathbf{v}} f(\mathbf{v}) = 2\mathbf{\Phi}\mathbf{v} - 2\mathbf{u}$
>     
> 2. **黎曼梯度 (Riemannian Gradient)**：
>     
>     将欧氏梯度投影到流形的切空间 (Tangent Space) 上，确保优化的搜索方向是沿着流形表面的：
>     
>     $$\text{grad} f(\mathbf{v}) = \nabla_{\mathbf{v}} f(\mathbf{v}) - \Re\{\nabla_{\mathbf{v}} f(\mathbf{v}) \odot \mathbf{v}^*\} \odot \mathbf{v}$$
>     
>     _(这里 $\odot$ 是哈达玛乘积)_
>     
> 3. **回撤操作 (Retraction)**：
>     
>     沿着切空间走了一步之后，变量会脱离流形表面。回撤操作负责把它强行拉回单位圆上：
>     
>     $$\mathbf{v}^{(t+1)} = \mathcal{R}_{\mathbf{v}^{(t)}}(-\alpha \cdot \text{grad} f(\mathbf{v}^{(t)})) = \frac{\mathbf{v}^{(t)} - \alpha \cdot \text{grad} f(\mathbf{v}^{(t)})}{|\mathbf{v}^{(t)} - \alpha \cdot \text{grad} f(\mathbf{v}^{(t)})|}$$
>     
>     _(其中 $\alpha$ 是通过 Armijo 准则等线搜索方法得到的步长)_。可以配合共轭梯度法 (CG) 在流形上加速收敛。

#### 子问题 3：优化波导管物理位置 $\mathbf{u}$ (带刚体耦合约束)

**1. 物理模型到数学的映射 (降维重构)**

我们不再优化所有阵元的绝对位置 $\mathbf{x}$，而是优化 **$N_{\mathrm{RF}}$ 根波导管的滑动位移 $\mathbf{u} = [u_1, u_2, \dots, u_{N_{\mathrm{RF}}}]^T$**。

对于第 $n$ 根波导管上的第 $m$ 个阵元，它的绝对物理位置可以写成：

$$x_{n,m} = u_n + d_m$$

其中：

- $u_n$ 是第 $n$ 根波导管当前的基准位置（我们的**唯一优化变量**）。
    
- $d_m$ 是第 $m$ 个阵元在波导管上的**固定局部偏移量**（例如阵元间距为半波长，则 $d_m = (m-1)\frac{\lambda}{2}$，这是常量）。
    

代入空间导向矢量中，该阵元对角度 $\theta$ 的相位响应变为极其优美的可分离形式：

$$e^{j\frac{2\pi}{\lambda} x_{n,m} \sin\theta} = \underbrace{e^{j\frac{2\pi}{\lambda} u_n \sin\theta}}_{\text{波导管整体滑动带来的相位}} \cdot \underbrace{e^{j\frac{2\pi}{\lambda} d_m \sin\theta}}_{\text{阵元固定排布带来的常量相位}}$$

---

**2. 求解策略 1（链式梯度下降法 Gradient Descent）**

因为 $N_M$ 个阵元的位置 $x_{n,m}$ 都由同一个电机变量 $u_n$ 决定，所以在算目标函数 $f$ 对 $u_n$ 的梯度时，必须对这根波导管上的所有阵元**求偏导并累加（全微分的链式法则）**：

$$\nabla_{u_n} f = \sum_{m=1}^{N_M} \frac{\partial f}{\partial x_{n,m}} \cdot \frac{\partial x_{n,m}}{\partial u_n}$$

因为 $\frac{\partial x_{n,m}}{\partial u_n} = 1$（刚体平移，位移量一比一传递），所以梯度极为简单，就是把这根管子上所有阵元的偏导数加起来：

$$\nabla_{u_n} f = \sum_{m=1}^{N_M} \frac{\partial f}{\partial x_{n,m}}$$

**迭代公式：**

$$\mathbf{u}^{(t+1)} = \mathbf{u}^{(t)} - \beta \nabla_{\mathbf{u}} f(\mathbf{u}^{(t)})$$

最后加入边界投影：如果 $u_n$ 超出了滑轨的物理极限 $[0, L_{max}]$，则 $u_n = \max(0, \min(L_{max}, u_n))$。

---

**3. 求解策略 2（连续凸近似 SCA，推荐用于顶刊）**

针对非线性的波导管滑动相位项 $e^{j\frac{2\pi}{\lambda} u_n \sin\theta}$，我们在前一次的迭代点 $u_n^{(t)}$ 处进行一阶泰勒展开（Taylor Expansion）。

令常数系数 $c(\theta) = \frac{2\pi}{\lambda} \sin\theta$，则：

$$e^{j c(\theta) u_n} \approx e^{j c(\theta) u_n^{(t)}} + j c(\theta) e^{j c(\theta) u_n^{(t)}} \cdot (u_n - u_n^{(t)})$$

**化腐朽为神奇的时刻：**

通过泰勒展开，原本极其难搞的指数函数（非凸的），变成了一个关于变量 $u_n$ 的**纯线性函数（Linear Function）**！

此时，你的目标函数变成了一个关于 $\mathbf{u}$ 的二次型（Quadratic）或者线性约束凸问题。

你可以直接把这个近似后的目标函数丢进 **CVX (凸优化工具箱)** 里，加上线性不等式约束 $0 \le u_n \le L_{max}$，CVX 会在零点几秒内给你返回一个全局最优解。

---