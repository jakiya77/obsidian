>[!hint]+ 下标的含义
>- **$k$**：代表第 $k$ 个**用户终端**（User Terminal, UT 或 UE）。如果空间里有多个干扰源或多个用户，这就代表其中特定的一点。
>- **$m$**：代表接收端 6DMA 阵列上的第 $m$ 根**物理天线阵元**（Antenna Element at AP）。
>- **$k, m$（作为下标组合）**：代表从用户 $k$ 的发射天线到基站阵列的第 $m$ 根接收天线之间的**这一条特定的空间宏观链路**。
>- **$L$（准确地说是 $L_{k,m}$）**：代表在这一条 $(k, m)$ 宏观链路内部，大自然经过反射、折射、散射后，实际到达天线 $m$ 的**多径射线的总数量**（Number of multipath rays）。

针对接收信道的建模：
$$h_{k,m}(q_m, A_m) = \mathbf{f}_{k,m}^H(q_m) \mathbf{G}_{k,m}(A_m) \mathbf{a}_{k,m}$$
$$h_{k,m} = \underbrace{\mathbf{f}_{k,m}^H}_{1 \times L} \times \underbrace{\mathbf{G}_{k,m}}_{L \times L} \times \underbrace{\mathbf{a}_{k,m}}_{L \times 1}$$
    1.  $\mathbf{f}_{k,m}^H \iff \mathbf{v}$ 导向矢量，$\mathbf{q}_m$是天线的位置$\mathbf{q}_m = [x_m, y_m, z_m]^T \in \mathbb{R}^{3 \times 1}$
	2. $\mathbf{G}(\mathbf{A}_m) \iff \mathbf{EF}$ 天线辐射包络，硬件自身的电磁特性
	3. $\mathbf{a_{k,m}}\iff$ 空间多径响应的复数
	4. $\mathbf{A}_m = [U_m, V_m] \in \mathbb{R}^{3 \times 2}$ : 当下阵元的位姿态，两组orthogonal vector 因此$\mathbf{A_{m}^T}\mathbf{A_{m}}=\mathbf{I}$

![[Pasted image 20260430130957.png|364]]
在这个信道模型中特别关注EF的定义
$$[G_{k,m}(A_m)]_{l,l} = \frac{\lambda}{4\pi d_{k,m}} \sqrt{\hat{g}_{k,m}^l(A_m) \bar{g}_{k,m}^l(A_m)}$$
1. $\mathbf{d}(\psi_{k,m}^l) = [\cos \theta_{k,m}^l \cos \varphi_{k,m}^l, \cos \theta_{k,m}^l \sin \varphi_{k,m}^l, \sin \theta_{k,m}^l]^T \in \mathbb{R}^{3 \times 1}$ 
2. $G_{k,m}$ 是一个对角阵列，他里面的元素$$\hat{g}_{k,m}^l(A_m) \triangleq \hat{g}(\mathbf{u}_m; \psi_{k,m}^l) = \max \{ \mathbf{d}(\psi_{k,m}^l)^T \mathbf{u}_m, 0 \}$$
    这里的$\hat{g}_{k,m}^l{(A_m)}$ 是一个由信号来向

也就是说 他这一共有两个维度 一个天线个数的维度 一个是信号个数的维度(也就是多径的维度) 我一般进行处理是直接考虑天线个数维度的 多个天线阵列接收到一个信号和多个干扰的耦合作为信道；但是他这里是从信号的角度，观测的是多个信号在一个天线上的表现作为信道

