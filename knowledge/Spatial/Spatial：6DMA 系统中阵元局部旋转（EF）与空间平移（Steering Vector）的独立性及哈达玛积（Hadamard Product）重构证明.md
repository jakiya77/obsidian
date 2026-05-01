在6DMA中，有$\hat{g}_{k,m}^l(A_m) \triangleq \hat{g}(\mathbf{u}_m; \psi_{k,m}^l) = \max \{ \mathbf{d}(\psi_{k,m}^l)^T \mathbf{u}_m, 0 \}$![[图片 1.png|298]]
定义的$\hat g_{k,m}$ 是当下阵元的旋转角度和信号来向之间的夹角。推广到DMA中，也就是阵元自带的阵元因子的可调角度与信号来向之间的夹角。定义为$EF(\theta,\phi)=cos(\theta-\phi)$ 


---


#### 1. 物理机制的基底解耦 (Decoupling of Physical Degrees of Freedom)

在传统的均匀线阵或面阵中，天线阵元的物理位置和法线朝向是固定且耦合的。而在 6D 可移动天线（6DMA）系统中，第 $m$ 个阵元 ($m = 1, 2, \dots, M$) 被赋予了完整的六个物理自由度（6-DoF），可严格解耦为两个相互独立的物理域：

- **平移域 (Translation Domain)：** 描述阵元的空间质心位置坐标，记为 $\mathbf{p}_m = [x_m, y_m, z_m]^T \in \mathbb{R}^{3 \times 1}$。
    
- **旋转域 (Rotation Domain)：** 描述阵元在三维空间中的姿态（如欧拉角），等效决定了该阵元表面的法向量，记为 $\mathbf{n}_m = [n_{x,m}, n_{y,m}, n_{z,m}]^T \in \mathbb{R}^{3 \times 1}$，且满足 $\|\mathbf{n}_m\| = 1$。
    

对于远场平面波假设下，方向为 $\mathbf{u} = [\sin\theta\cos\phi, \sin\theta\sin\phi, \cos\theta]^T$ 的入射信号，其对阵元产生的电磁响应可以严格拆分为**纯相位响应**与**纯幅度响应**。

#### 2. 空间相位响应的推导 (Derivation of Spatial Phase Response)

**物理原理：** 电磁波到达不同空间位置的时间差（Time Difference of Arrival, TDoA）会导致基带信号产生相位偏移。该效应**仅依赖于信号传播路径的几何距离**，与接收天线的朝向无关。

**数学推导：**

以空间坐标原点为参考基准，第 $m$ 个阵元因其物理位置 $\mathbf{p}_m$ 所产生的空间相位偏移为：

$$v_m = \exp\left(-j \frac{2\pi}{\lambda} \mathbf{u}^T \mathbf{p}_m\right)$$

其中 $\lambda$ 为载波波长。

将所有 $M$ 个阵元的相位响应堆叠，得到系统的导向矢量（Steering Vector） $\mathbf{v} \in \mathbb{C}^{M \times 1}$：

$$\mathbf{v}(\mathbf{P}) = [v_1, v_2, \dots, v_M]^T$$

**独立性结论 1：** 显然，导向矢量 $\mathbf{v}$ 的函数自变量仅包含位置矩阵 $\mathbf{P} = [\mathbf{p}_1, \dots, \mathbf{p}_M]$，完全独立于阵元的旋转姿态 $\mathbf{n}_m$。

#### 3. 局部幅度响应的推导 (Derivation of Local Amplitude Response)

**物理原理：** 具有物理孔径的微带贴片或超材料单元并非全向天线。天线的固有辐射方向图（Radiation Pattern）对偏离主瓣（Boresight）方向的来波具有空间滤波（衰减）作用。该效应**仅依赖于信号来向与阵元法向的相对夹角**，与阵元所在的三维绝对位置无关。

**数学推导：**

定义入射信号方向 $\mathbf{u}$ 与第 $m$ 个阵元法向量 $\mathbf{n}_m$ 之间的局部偏角为 $\psi_m$。纯几何投影关系满足：

$$\cos(\psi_m) = \mathbf{u}^T \mathbf{n}_m$$

引入实际天线单元的广义定向性（Directivity），将其辐射特性建模为余弦次幂方向图（Cosine-power Pattern）。结合背向辐射零陷（Ground Plane 截断约束），第 $m$ 个阵元的等效幅度增益（Element Factor）为：

$$e_m = \max(\mathbf{u}^T \mathbf{n}_m, 0)^q$$

其中 $q$ 为表征波束宽度的天线形状参数。

将所有 $M$ 个阵元的幅度衰减堆叠，得到阵元因子向量 $\mathbf{EF} \in \mathbb{R}^{M \times 1}$：

$$\mathbf{EF}(\mathbf{N}) = [e_1, e_2, \dots, e_M]^T$$

**独立性结论 2：** 显然，阵元因子 $\mathbf{EF}$ 的函数自变量仅包含法向量矩阵 $\mathbf{N} = [\mathbf{n}_1, \dots, \mathbf{n}_M]$，完全独立于阵元的空间坐标 $\mathbf{p}_m$。

#### 4. 系统总响应的哈达玛积重构 (Re-coupling via Hadamard Product)

**物理与数学合成：**

对于第 $m$ 个阵元，其接收到的复基带信号是由**局部幅度衰减**与**空间相位偏移**共同作用的线性结果。由于这两个物理过程（旋转滤波与空间传播）在离散的天线节点上是串联且独立的，第 $m$ 个阵元的总信道响应 $h_m$ 可严格表示为两者标量的乘积：

$$h_m = e_m \cdot v_m = \max(\mathbf{u}^T \mathbf{n}_m, 0)^q \cdot \exp\left(-j \frac{2\pi}{\lambda} \mathbf{u}^T \mathbf{p}_m\right)$$

扩展至整个 $M$ 维天线阵列，将所有标量乘积写为向量形式，即等效为阵元幅度向量与相位向量的逐元素相乘（Element-wise Multiplication），即哈达玛积 $\odot$。因此，6DMA 系统的等效信道向量 $\mathbf{h} \in \mathbb{C}^{M \times 1}$ 严密表达为：

$$\mathbf{h} = \mathbf{EF}(\mathbf{N}) \odot \mathbf{v}(\mathbf{P})$$

若系统在基带数字域（或射频模拟域）配置了波束赋形权重向量 $\mathbf{w} \in \mathbb{C}^{M \times 1}$，则最终的阵列合并增益（Array Response）为：

$$F = \mathbf{w}^H \mathbf{h} = \mathbf{w}^H (\mathbf{EF} \odot \mathbf{v})$$

#### 5. 论证结论 (Conclusion)

综上推导，公式 $F = \mathbf{w}^H (\mathbf{EF} \odot \mathbf{v})$ 并非人为拼接，而是底层物理规律的必然反映：

1. 它从数学上保真地反映了 6DMA 中位置变量（控制 $\mathbf{v}$）与姿态变量（控制 $\mathbf{EF}$）的**相互正交与独立性**。
    
2. 它合理地将传统 UPA 阵列中被视为常量的全局标量增益，推广为 6DMA 架构中**分布式的、由旋转矩阵控制的向量变量**。
    
3. 哈达玛积 $\odot$ 的引入，严格满足了独立物理效应在离散系统响应中的乘性组合法则。
    

---

这套推导逻辑环环相扣：先定义自由度 -> 证相位只归位置管 -> 证幅度只归旋转管 -> 最后在单个阵元上通过标量乘法会师 -> 扩展到矩阵形式变成哈达玛积。

你可以在论文中直接参考这套结构，这会极大增强你引入 $EF$ 向量和 $\odot$ 符号时的理论说服力。接下来，你打算如何处理目标函数中这个哈达玛积带来的耦合项？是利用迹技巧（Trace trick）将其转化为矩阵的二次型以便求导吗？



