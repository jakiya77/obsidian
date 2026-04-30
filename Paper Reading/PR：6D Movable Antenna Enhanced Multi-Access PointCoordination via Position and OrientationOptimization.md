
针对接收信道的建模：
$$h_{k,m}(q_m, A_m) = \mathbf{f}_{k,m}^H(q_m) \mathbf{G}_{k,m}(A_m) \mathbf{a}_{k,m}$$
$$h_{k,m} = \underbrace{\mathbf{f}_{k,m}^H}_{1 \times L} \times \underbrace{\mathbf{G}_{k,m}}_{L \times L} \times \underbrace{\mathbf{a}_{k,m}}_{L \times 1}$$

	1. $\mathbf{f}_{k,m}^H \iff \mathbf{v}$ 导向矢量，$\mathbf{q}_m$是天线的位置$\mathbf{q}_m = [x_m, y_m, z_m]^T \in \mathbb{R}^{3 \times 1}$
	2. $\mathbf{G}(\mathbf{A}_m) \iff \mathbf{EF}$ 天线辐射包络，硬件自身的电磁特性
	3. $a_{k,m}$ 想当于是一个空间多径的复数响应
	4. $$y = \underbrace{\mathbf{w}^H}_{\text{数字基带}} \times \left( \underbrace{\mathbf{EF} \odot \mathbf{v}}_{\text{射频硬件阵列响应}} \right) \times \underbrace{\alpha}_{\text{大自然信道衰落}} \times \underbrace{s}_{\text{发射符号}}$$
在这个信道模型中特别关注EF的定义
$$[G_{k,m}(A_m)]_{l,l} = \frac{\lambda}{4\pi d_{k,m}} \sqrt{\hat{g}_{k,m}^l(A_m) \bar{g}_{k,m}^l(A_m)}$$
1. $G_{k,m}$ 是一个对角阵列，他里面的元素