
## I. Introduction

### Main purpose

说明为什么接收端 MA 阵列适合用于抗干扰，以及为什么现有 MA 位置优化方法仍然存在复杂度问题。

### Logic

1. 无线接收端在强方向性干扰下性能下降，传统固定阵列只能依赖数字波束形成，空间采样位置固定。
    
2. MA 引入阵元位置自由度，使接收机可以在有限孔径内选择更有利的空间采样点。
    
3. 现有 MA 优化多将位置设计建模为非凸优化问题，常依赖 full-grid search、AO、SCA、CVX 或黑箱优化，计算代价较高。
    
4. 本文不直接黑箱优化最终 SINR，而是先分析 desired 和 jammer 信道在位置域的空间变化结构。
    
5. 基于该结构提出 FADLS，实现快速、物理启发式的接收端 MA 抗干扰位置选择。
    

### Key claim

The key idea is to exploit position-domain channel structures rather than repeatedly solving receiver-level SINR optimization problems.

### Contributions

1. Reveal a position-domain dual-favorable structure induced by desired and jamming multipath channels.
    
2. Propose FADLS, which jointly considers desired-channel gain, jammer-channel weakness, and desired-jammer channel separability.
    
3. Show that FADLS achieves competitive post-MMSE output SINR compared with full-grid SINR-greedy and local AO-SCA-CVX baselines, while avoiding repeated CVX solves and reducing empirical runtime.
    

---

## II. System Model

### Main purpose

建立接收端 MA 抗干扰模型，明确变量、信道、约束和统一性能指标。

>[!hint]+ 场景→位置约束→信道模型→接收信号→output SINR→位置优化问题→引出下一节
>This section only defines the physical model and evaluation metric. Do not introduce FADLS yet.



### Logic

1.  receive-side anti-jamming communication system。
    
2. 单天线 desired transmitter 向接收机发送信号。
    
3. 接收机配备 (N)-element MA array，阵元位于一维有限孔径内。
    
4. 单天线 jammer 发射强干扰。
    
5. MA 位置向量为  
    
$$
    \mathbf p=[p_1,p_2,\ldots,p_N]^T.  
$$
    
    
6. 位置满足孔径约束和最小间距约束：  
$$
    [  
    -\frac{A}{2}\le p_1<\cdots<p_N\le \frac{A}{2},  
    ]  
    [  
    p_{n+1}-p_n\ge d_{\min}.  
    ]
$$

    
7. desired 和 jammer 信道由多径叠加形成：  
$$
    [  
    h(p)=\sum_{\ell=1}^{L_s}\alpha_\ell e^{-j\frac{2\pi}{\lambda}p\sin\theta_{s,\ell}},  
    ]  
    [  
    g(p)=\sum_{\ell=1}^{L_j}\beta_\ell e^{-j\frac{2\pi}{\lambda}p\sin\theta_{j,\ell}}.  
    ]

$$

    
8. 接收信号为  
$$
    [  
    \mathbf y=\sqrt{P_s}\mathbf h(\mathbf p)x_s+\sqrt{P_j}\mathbf g(\mathbf p)x_j+\mathbf n.  
    ]
$$
    
9. 统一采用 output SINR：  
$$
    [  
    \Gamma(\mathbf p)=  
    P_s\mathbf h^H(\mathbf p)  
    \left(P_j\mathbf g(\mathbf p)\mathbf g^H(\mathbf p)+\sigma^2\mathbf I\right)^{-1}  
    \mathbf h(\mathbf p).  
    ]
$$
    这个部分分离出$h(p)g(p)$
    

### Key point



---

## III. Proposed Position-Domain Dual-Favorable Selection
A. Position-Domain Dual-Favorable Profiles
B. Set-Level Desired-Jammer Separability
C. FADLS Algorithm
D. Complexity Discussion

### Main purpose

解释为什么 MA 位置选择不是黑箱问题，而是存在可利用的物理结构。

### Logic

1. 多径叠加使 (h(p)) 和 (g(p)) 在 aperture 上随位置变化。
    
2. 定义归一化 desired-channel gain profile：  
    
$$
    H(p)=\frac{|h(p)|^2}{\max_q |h(q)|^2}.  
$$
  
    
3. 定义归一化 jammer-channel gain profile：  
    
$$
    G(p)=\frac{|g(p)|^2}{\max_q |g(q)|^2}.  
$$
    
    
4. 当 (H(p)) 大时，该位置对 desired signal 有利。
    
5. 当 (G(p)) 小时，该位置处 jammer 较弱。
    
6. 因此 (H(p)) 大且 (G(p)) 小的位置是 dual-favorable positions。
    
7. 用  
    
$$
    H(p)(1-G(p))  
$$
    
    作为单点 dual-profile score，解释候选池预筛选的物理来源。
    
8. 但单点好不代表阵列组合好，因此还需要集合级 desired-jammer separability。
    

### Key point

这一节是论文创新的物理基础。它要支撑 Fig. 1。

### Figure

Fig. 1: Position-domain profiles.

- (H(p))
    
- (1-G(p))
    
- (H(p)(1-G(p)))
    
- FADLS selected positions
    

### Caption message

FADLS exploits the position-domain dual-favorable regions rather than blindly searching over the aperture.

---

## IV. Simulation Results
A. Simulation Setup and Baselines
B. Position-Domain Mechanism
C. Main Performance Comparison
D. Beam Pattern Visualization
E. Robustness to Estimation Error
### Main purpose

把物理结构转化成具体低复杂度选点算法。

### Logic

1. FADLS 包含两个阶段：candidate pre-screening 和 set-dependent greedy selection。
    
2. 第一阶段：用单点 dual-score 选出 candidate pool：  
      
$$
    \mathcal C=\text{Top-}C{H(p)(1-G(p)),p\in\mathcal P}.  
$$
    
    
3. 第二阶段：从空集合开始逐根选择 MA 位置。
    
4. 第 (n) 根天线选择时，测试临时集合：  
    
$$
    \mathcal S_{\rm tmp}=\mathcal S_{n-1}\cup{p}.  
$$
    
    
5. 定义集合平均 desired gain：  

$$
    \bar H_{\mathcal S}=\frac{1}{|\mathcal S|}\sum_{p_i\in\mathcal S}H(p_i).  
$$
    
    
6. 定义集合平均 jammer weakness：  
    
$$
    1-\bar G_{\mathcal S}.  
$$
    
    
7. 定义 desired-jammer channel separability：  
    
$$
    \rho_{hg}(\mathcal S)=  
    \frac{|\mathbf h_{\mathcal S}^H\mathbf g_{\mathcal S}|^2}  
    {|\mathbf h_{\mathcal S}|^2|\mathbf g_{\mathcal S}|^2+\epsilon}.  
$$
    
    
8. FADLS score 为  
      
$$
    s_{\rm FADLS}(\mathcal S)  
    =  
    \omega_H\log(\bar H_{\mathcal S}+\epsilon)  
    +\omega_G\log(1-\bar G_{\mathcal S}+\epsilon)  
    +\omega_\rho\log(1-\rho_{hg}(\mathcal S)+\epsilon).  
$$
    
    
9. 选择使 score 最大且满足最小间距约束的位置。
    
10. 最终所有方法都用同一个 post-MMSE SINR 评价。
    

### Key point

FADLS 与 naive dual-score 的区别是：FADLS 是集合级选择，额外考虑 (\rho_{hg}(\mathcal S))。

---

## V. Conclusion