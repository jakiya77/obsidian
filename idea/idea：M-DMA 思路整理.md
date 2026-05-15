## 1. 物理架构
![[png：Dynamic Metasurface Antennas for  6G Extreme Massive MIMO Communications.png]]横向一排波导上嵌有若干个element，每一个波导连接着一个RF chain，设计每一个波导由电机驱动可以左右移动，即 $x_{n,m} = u_n + d_m$。
![[png：Modeling and Performance Analysis for Movable Antenna Enabled Wireless Communications.png]]
![[png：Movable-Element RIS-Aided Wireless Communications ：An Element-Wise Position Optimization Approach.png|452]]
上图分别是传统的可移动天线架构和可移动RIS架构。可以观察到早期全数字 MA 架构需要再每一个element上连接RF chain，不但面临高额的RF chain开销，还会存在每个天线独立 2D 运动所需的复杂机械解耦。而新型的ME-RIS架构不需要逐元素连接RF chain，只连接控制线，只能改变信号的传播路径，无法在本地进行任何信号剥离或自适应滤波。于是引入M-DMA架构，波导管作为信号传输与合成的物理媒介，实现对信号的波域处理和数字域处理。
 
## 2. 移动性的引入和过载零陷分析
$$K \le \frac{N_{\text{RF}}(N_M + 3)}{2} - 1$$

## 3. 优化问题

### Step 1 优化问题第一步是在波域对干扰进行对齐处理

**【优化问题 $\mathcal{P}_1$】：**

$$\min_{\mathbf{u}, \mathbf{v}, \mathbf{q}} \sum_{j=1}^J |\mathbf{q}^H \mathbf{h}_j(\mathbf{u}, \mathbf{v})|^2$$

$$\text{s.t.} \quad \|\mathbf{q}\|^2 = 1 \quad \text{(防止产生无意义的全 0 解)}$$

$$\quad \quad |v_{n,m}| = 1, \forall n, m \quad \text{(超表面相位恒模约束)}$$

$$\quad \quad 0 \le u_n \le L_{\max}, \forall n \quad \text{(波导管物理滑轨边界约束)}$$

1. **固定 $\mathbf{u}, \mathbf{v}$ 求 $\mathbf{q}$**：此时干扰矩阵 $\mathbf{R}_J = \mathbf{H}_J\mathbf{H}_J^H$ 是确定的。使得 $\mathbf{q}^H \mathbf{R}_J \mathbf{q}$ 最小的 $\mathbf{q}$，**就是 $\mathbf{R}_J$ 最小特征值对应的特征向量！** （这就是你代码里 `eig` 分解求 `w_null` 的数学依据，瑞利商定理）。
    
2. **固定 $\mathbf{q}, \mathbf{u}$ 求 $\mathbf{v}$**：变成了一个标准的二次型最小化问题，直接丢给黎曼流形优化。
    
3. **固定 $\mathbf{q}, \mathbf{v}$ 求 $\mathbf{u}$**：直接计算一阶梯度下降。