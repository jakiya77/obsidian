
针对接收信道的建模：
$$h_{k,m}(q_m, A_m) = \mathbf{f}_{k,m}^H(q_m) \mathbf{G}_{k,m}(A_m) \mathbf{a}_{k,m}$$
	1. 这里的$f_{k,m}^H$ 相当于是我的导向矢量，$\mathbf{q}_m$是天线的位置$\mathbf{q}_m = [x_m, y_m, z_m]^T \in \mathbb{R}^{3 \times 1}$
	2. $G_{k,m}$ 相当于是我的EF
	3. $a_{k,m}$ 想当于是一个空间多径的复数响应
在这个信道模型中特别关注EF的定义
$$[G_{k,m}(A_m)]_{l,l} = \frac{\lambda}{4\pi d_{k,m}} \sqrt{\hat{g}_{k,m}^l(A_m) \bar{g}_{k,m}^l(A_m)}$$
1. $G_{k,m}$ 是一个对角阵列