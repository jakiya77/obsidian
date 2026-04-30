
针对接收信道的建模：
$$h_{k,m}(q_m, A_m) = \mathbf{f}_{k,m}^H(q_m) \mathbf{G}_{k,m}(A_m) \mathbf{a}_{k,m}$$
$$h_{k,m} = \underbrace{\mathbf{f}_{k,m}^H}_{1 \times L} \times \underbrace{\mathbf{G}_{k,m}}_{L \times L} \times \underbrace{\mathbf{a}_{k,m}}_{L \times 1}$$
    1.  $\mathbf{f}_{k,m}^H \iff \mathbf{v}$ 导向矢量，$\mathbf{q}_m$是天线的位置$\mathbf{q}_m = [x_m, y_m, z_m]^T \in \mathbb{R}^{3 \times 1}$
	2. $\mathbf{G}(\mathbf{A}_m) \iff \mathbf{EF}$ 天线辐射包络，硬件自身的电磁特性
	3. $a_{k,m}$ 想当于是一个空间多径的复数响应
在这个信道模型中特别关注EF的定义
$$[G_{k,m}(A_m)]_{l,l} = \frac{\lambda}{4\pi d_{k,m}} \sqrt{\hat{g}_{k,m}^l(A_m) \bar{g}_{k,m}^l(A_m)}$$
2. $G_{k,m}$ 是一个对角阵列，他里面的元素$$\hat{g}_{k,m}^l(A_m) \triangleq \hat{g}(\mathbf{u}_m; \psi_{k,m}^l) = \max \{ \mathbf{d}(\psi_{k,m}^l)^T \mathbf{u}_m, 0 \}$$

也就是说 他这一共有两个维度 一个天线个数的维度 一个是信号个数的维度(也就是多径的维度) 我一般进行处理是直接考虑天线个数维度的 多个天线阵列接收到一个信号和多个干扰的耦合作为信道；但是他这里是从信号的角度，观测的是多个信号在一个天线上的表现作为信道

