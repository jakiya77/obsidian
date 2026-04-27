>🐻：
> 这篇论文的寻找最优权重的方法很常规，是通过先MUSIC估计出干扰的来向，然后通过向干扰子空间投影的方式对干扰信号进行抑制；创新的部分就是如何让离散的，相位有限的超表面可以从幅度和相位上逼近求出来的最优权重向量。
> 1️⃣、干扰的抑制
> [[Spatial：向干扰子空间的正向投影矩阵（Projection Matrix onto the Jamming Subspace）]]
> 求出对应的$$w_{opt} = R_{yy}^{-1} R_{s+j} P_j a(\theta_0)$$
> 2️⃣、把离散的相位拟合到这个w上面
> 需要做两个事情：1. 幅度的拟合 2.相位的拟合
> 

$$W_{D,m}(t) = \sum_{l=0}^{L_s-1} e^{-j\varphi_l} g(t-lT_s)$$$$W_{D,m}(t) = \sum_{k=-\infty}^{\infty} \left[ \sum_{l=0}^{L_s-1} e^{-j\varphi_l} c_k(l) \right] e^{j k w_0 t}$$
$$W_{D,m}(t) = \sum_{k=-\infty}^{\infty} \underbrace{\left[ \frac{1}{L_s} Sa\left(\frac{k\pi}{L_s}\right) \sum_{l=0}^{L_s-1} e^{-j\left( \varphi_l + \frac{(2l+1)k\pi}{L_s} \right)} \right]}_{F_k} e^{j k w_0 t}$$
$$\mathcal{F}(W_{D,m}(t-t_*)) = \underbrace{\left( e^{-j w t_*} \right)}_{\text{【相位控制项】}} \cdot \sum_{k=-\infty}^{\infty} \underbrace{\left[ \frac{1}{L_s} Sa\left(\frac{k\pi}{L_s}\right) \sum_{l=0}^{L_s-1} e^{-j\left(\varphi_l + \frac{(2l+1)k\pi}{L_s}\right)} \right]}_{\text{【幅度与基础相位合成项】}} \delta(w - k w_0)$$
$$F_1 = \underbrace{\left( \frac{1}{L_s} Sa\left(\frac{\pi}{L_s}\right) \left| \sum_{l=0}^{L_s-1} e^{-j\left(\varphi_l + \frac{(2l+1)\pi}{L_s}\right)} \right| \right)}_{\text{【幅度控制 (Amplitude Control)】}} \cdot \underbrace{\left( e^{j \left[ \text{angle}\left( \sum_{l=0}^{L_s-1} e^{-j\left(\varphi_l + \frac{(2l+1)\pi}{L_s}\right)} \right) - w_0 t_* \right]} \right)}_{\text{【相位控制 (Phase Control)】}}$$
- **幅度控制部分**
    
    - **核心手段**：优化离散相位序列 $\varphi_l$ 的排列 。
        
    - **原理**：通过在 $L_s$ 个时隙内精准切换相位，使得多个单位矢量叠加后的合矢量模长刚好等于目标幅度值 。
        
    - **外部包络**：$Sa(\cdot)$ 函数作为固定权重，决定了该谐波能达到的最大理论幅度 。
        
	- **使用的算法**：**交替优化 **步骤 A：固定相位序列 $\varphi$，求解最优参考相位 $\psi$** 当给定调制序列时，最优的相位补偿值直接由该序列产生的一阶谐波相位决定，即 $\psi_{opt} = angle(\vartheta^H \varphi)$ 。**步骤 B：固定参考相位 $\psi_{opt}$，求解最优相位序列 $\varphi$** 此时问题转化为一个**离散二次规划问题** 。
- **相位控制部分：
    
    - **核心手段**：调节时间延迟 $t_*$ 。
        
    - **原理**：虽然序列内部的 $\varphi_l$ 会产生一个初始相位（即公式中 $\text{angle}(\cdot)$ 部分），但这个相位是离散且不灵活的 。
        
    - **精细调节**：通过连续调节 $t_*$，利用频域相移特性 $-w_0 t_*$ 对初始相位进行补偿，从而实现 $0 \sim 2\pi$ 范围内的**连续相位控制** 。
    
	- **使用的算法**：半正定松弛 (Semidefinite Relaxation, SDR)

$$t_* = \frac{T_p (angle(w_m) - angle(\varphi_{opt}^H \vartheta))}{2\pi}$$
