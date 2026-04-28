---
tags:
  - 空域
  - 波束成形
aliases:
  - 方向图乘积定理
  - 阵元因子与阵列因子
---

# DMA单个阵元与阵列辐射特性理论体系

> [!abstract] 核心要点
> 1. 天线远场的方向图可以冲两个不同的角度进行定义，从算法的角度和从天线的角度，从而可以得到权重矢量 $\times$ 导向矢量 的形式以及当个天线的辐射包络 $\times$ 权重作用在空间位置后得到的人为干涉图案两种
> 2. 
> 

---

## 1. 核心定理：方向图乘积定理

无论是传统相控阵还是 DMA，其远场辐射方向图 $F(\theta, \phi)$ 均遵循以下乘积定理：

$$F(\theta, \phi) = \underbrace{\mathbf{w}^H}_{\text{数字端主观施加的权重}} \times \underbrace{EF(\theta, \phi)}_{\text{单天线客观存在的辐射包络}} \odot \underbrace{\mathbf{v}(\theta, \phi)}_{\text{阵元物理位置决定的空间相位延迟}}$$
V和EF是耦合还是独立的
### 视角一：算法与信号处理（导向矢量逻辑）

**算法**的角度，物理世界的东西打包在一起，作为环境给的已知量：

$$F(\theta, \phi) = \mathbf{w}^H \underbrace{\Big[ EF(\theta, \phi) \cdot \mathbf{v}(\theta, \phi) \Big]}_{\text{导向矢量 } \mathbf{a}(\theta, \phi)}$$
- 系统总方向图 $=$ $\underbrace{\text{设计的权重 (w)}}_{\text{主观调控}}$ 与$\underbrace{\text{系统硬件的空间签名 (a)}}_{\text{客观物理}}$ 的空间匹配内积。
    
### 视角二：天线与微波工程（乘积定理逻辑）

当我们站在**天线**的角度，我们会把阵列空间干涉打包在一起，认为它是宏观波束的基础：

$$F(\theta, \phi) = EF(\theta, \phi) \cdot \underbrace{\Big[ \mathbf{w}^H \mathbf{v}(\theta, \phi) \Big]}_{\text{阵列因子 } AF(\theta, \phi)}$$

- **逻辑表达**：系统总方向图 $=$ $\underbrace{\text{单天线的辐射包络 (EF)}}_{\text{物理边界}}$ $\times$  $\underbrace{\text{权重作用于空间位置后形成的人为干涉图案 (AF)}}_{\text{空间重塑}}$。

>[!faq] 这里需要注意阵元位置决定的空间相位延迟没有显式地体现在两个方程中
---
## 2. 算法层面：DMA中的阵元异构在远场中的体现

如果DMA 每个阵元的结构、指向或极化都不一样，那么 $EF(\theta, \phi)$ 就不再是一个标量，而是一个包含了所有阵元自身响应的向量 $\mathbf{e}(\theta, \phi)$。此时的导向矢量就是阵元响应和空间位置的哈达玛积。
$$F(\theta, \phi) = \mathbf{w}^H \underbrace{\Big[ \mathbf{e}(\theta, \phi) \odot \mathbf{v}(\theta, \phi) \Big]}_{\text{广义导向矢量 } \mathbf{a}_{\text{new}}(\theta, \phi)}$$
>[!hints]- Hadamard Product: Element-wise multiplication
>$$\mathbf{a} = \begin{bmatrix} e_1 \\ e_2 \\ \vdots \\ e_N \end{bmatrix} \odot \begin{bmatrix} v_1 \\ v_2 \\ \vdots \\ v_N \end{bmatrix} = \begin{bmatrix} e_1 \cdot v_1 \\ e_2 \cdot v_2 \\ \vdots \\ e_N \cdot v_N \end{bmatrix}$$
---

## 3. 天线层面：DMA的阵列因子与普通阵列的差异

### 3.1 阵列因子的数学表达
假设有一个包含 $N$ 个阵元的阵列，阵列因子表示为所有阵元辐射场在远场某点叠加的结果：

$$AF(\theta, \phi) = \sum_{n=1}^{N} w_n e^{j \mathbf{k} \cdot \mathbf{r}_n}$$

- $w_n = A_n e^{j \alpha_n}$: 第 $n$ 个阵元的复数激励权重（包含幅度 $A_n$ 和相位 $\alpha_n$）。
- $\mathbf{k}$: 自由空间波数矢量，大小为 $2\pi/\lambda$，方向为观察方向 $(\theta, \phi)$。
- $\mathbf{r}_n$: 第 $n$ 个阵元在坐标系中的位置矢量。

### 3.2 DMA 中的 AF 与传统阵列的区别
传统全数字相控阵中，$w_n$ 由基带的数字预编码器和射频移相器/衰减器独立控制。
![[png：dma.png]]

> [!important] DMA 的洛伦兹谐振约束 (Lorentzian Resonance)
> 在 DMA 中，权重 $w_n$ 不是随意的复数，它取决于微波网络与超材料阵元的耦合物理机制。通过调节 PIN 二极管或变容二极管，改变的是阵元的谐振频率，从而得到一个受洛伦兹模型约束的幅相响应：
> $$w_n = \frac{j \omega L}{R + j(\omega L - \frac{1}{\omega C_n})}$$
> （其中 $C_n$ 是可调电容）。
> 这意味着在进行波束优化（如推导最优导向矢量）时，幅度 $A_n$ 和相位 $\alpha_n$ 是强耦合的，这也是 DMA 信号处理与优化中常遇到的非凸约束挑战。

---

## 4. 总结与优化应用 (To-Do & Research Links)

如果要在基于 DMA 的系统中最大化某一点的接收能量：
1. **宏观层面 (AF)**：利用优化算法求解最优的权重向量 $\mathbf{w}$，使得 $AF$ 在目标方向形成尖锐的高增益波束。
2. **微观层面 (EF)**：需要确保系统的物理摆放使得目标用户处于单个阵元辐射的有效包络（EF）内。
3. **通道建模 (Channel Modeling)**：在构建具有物理意义的信道矩阵 $\mathbf{H}$ 时，真实的天线流形向量（Steering Vector）必须引入 $EF(\theta, \phi)$ 的衰减，即 $\mathbf{a}(\theta, \phi) = EF(\theta, \phi) \cdot [e^{j \mathbf{k}\cdot\mathbf{r}_1}, ..., e^{j \mathbf{k}\cdot\mathbf{r}_N}]^T$。

---