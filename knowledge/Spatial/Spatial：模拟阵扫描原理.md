
场景：256个模拟阵元，分成2✖️2的子阵，一个子阵由8✖️8的方阵组成。整个模拟阵关于原点对称，需要在针对四个子阵得到每一个子阵对应的目标方向和干扰方向的增益矩阵。

$$
一维来波方向为\theta 时的导向矢量 ~~steering~vector= e^{jkdsin{\theta}} 
$$
$$
三维来波方向为(\theta,\phi)时的导向矢量~steering~vector = e^{jkr^{\!\top}·S}~~~~~~~~~
$$
$$
S=\begin{bmatrix}
sin\theta cos\phi \\
sin\theta sin\phi \\
cos\theta
	\end{bmatrix} ~~定义天顶方向为(\theta=0,\phi=0)~~~~~~~ r=[x,\,y,\,z]^{\top}.
$$
因此，针对上述情况，可以分成两步求取gain：
1- 先得到四个子阵列在目标方向上的[[Antennas：阵列因子的相位补偿 合相]]：
$$

W_{\text{sub-scan}}(\theta,\phi)
=\frac{1}{\sqrt{N_{\text{sub}}}}\,
\exp\!\big(-j k\,\mathbf r^{\!\top}\mathbf s(\theta,\phi)\big),
\quad \mathbf r=[x,\,y,\,z]^{\top}.


$$
2-  针对每一个子阵得到一个逐元素的响应：
$$
先得到扫描方向上的~~ SV= e^{jk[x,y]S(\theta_{scan},\phi_{scan})}
$$
$$
再得到目标方向和干扰方向的导向矢量 
$$
$$
SV_{\text{target}} = e^{jk[x,y]S(\theta_{\text{target}},\phi_{\text{tearget}})}
$$
$$
SV_{\text{target}} = e^{jk[x,y]S(\theta_{\text{jam}},\phi_{\text{jam}})}
$$
