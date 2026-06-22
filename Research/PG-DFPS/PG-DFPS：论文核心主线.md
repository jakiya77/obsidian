II. System Model and Problem Formulation
    A. Signal and Channel Model
    B. Post-MMSE Output SINR and Position Optimization

III. Position-Domain Dual-Favorable Structure
    A. Two-Path Motivating Example
    B. Desired-Favorable, Jammer-Weak, and Separable Position Profiles

IV. Proposed DFPS/FADLS Algorithm
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
    
9. 统一采用 post-MMSE output SINR：  
$$
    [  
    \Gamma(\mathbf p)=  
    P_s\mathbf h^H(\mathbf p)  
    \left(P_j\mathbf g(\mathbf p)\mathbf g^H(\mathbf p)+\sigma^2\mathbf I\right)^{-1}  
    \mathbf h(\mathbf p).  
    ]
$$
    

### Key point

This section only defines the physical model and evaluation metric. Do not introduce FADLS yet.

---

## III. Position-Domain Dual-Favorable Structure

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

## IV. Proposed FADLS Algorithm

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

## V. Baselines and Complexity Discussion

### Main purpose

解释为什么选择这些 baseline，以及复杂度优势如何严谨表达。

### Logic

1. Fixed ULA：固定阵列参考。
    
2. Proposed FADLS：本文方法。
    
3. Full-grid SINR-greedy：强 receiver-aware baseline，每一步扫描全网格并直接计算 post-MMSE SINR。
    
4. Local AO-SCA-CVX：优化型局部 refinement baseline，使用 dual-profile ranking 初始化，再通过 SCA-CVX 局部微调。
    
5. 强调 AO-SCA-CVX 不是全局最优，而是 local optimization benchmark。
    
6. 复杂度不要用混合 proxy 图，而是用表格说明。
    
7. FADLS 避免：
    
    - full-grid receiver-level SINR scoring；
        
    - repeated CVX-based convex subproblem solving。
        
8. Runtime 可以作为 empirical runtime，不等同于理论复杂度。
    

### Table

Table I: Complexity and runtime comparison.

Recommended columns:

- Method
    
- Selection principle
    
- Search range
    
- Receiver-level SINR evaluations
    
- CVX solves
    
- Complexity / cost order
    
- Runtime
    

### Key statement

Theoretical complexity and empirical runtime are reported separately.

---

## VI. Simulation Results

### Main purpose

用最少但最有力的图证明：机制成立、性能有效、复杂度低、鲁棒性可接受。

### Simulation setup

说明共同参数：  

$$
N,\ A,\ d_{\min},\ M,\ C,\ P_s,\ P_j,\ \sigma^2,\ \text{JSR}.  
$$

说明所有方法最终统一用 post-MMSE output SINR 评价。

---

### A. Position-Domain Mechanism

Use Fig. 1.

### Main message

FADLS selected positions are concentrated around dual-favorable regions. This validates that the algorithm exploits channel-profile structures rather than black-box search.

---

### B. Main Performance Comparison

Use Fig. 2 with two subfigures.

#### Fig. 2(a): SINR vs Number of Paths

Main message:  
FADLS remains effective as the number of multipath components increases, showing multipath robustness.

#### Fig. 2(b): SINR vs JSR / INR

Main message:  
FADLS maintains competitive output SINR under strong jamming, showing anti-jamming capability.

### Wording

Do not write “FADLS always outperforms all baselines.”  
Write “FADLS achieves competitive or slightly higher average post-MMSE output SINR with much lower complexity.”

---

### C. Representative Beam Pattern

Use Fig. 3.

### Main message

The beam pattern provides an intuitive visualization that FADLS enhances the desired direction and suppresses the jammer direction.

### Note

This figure is illustrative, not the main quantitative proof.

---

### D. Robustness to Estimation Error

Use Fig. 4.

### Main message

Since FADLS relies on estimated channel profiles or path parameters, this experiment evaluates its robustness under AoA/profile estimation errors.

### Recommended setting

[  
\hat\theta=\theta+\Delta\theta.  
]

Plot average output SINR versus AoA error level.

---

## VII. Appendix / Supplementary

### Main purpose

保留有价值但不占主文版面的补充实验。

### Recommended appendix

1. Candidate pool size sensitivity  
    Purpose: justify (C=300) and show performance-search tradeoff.
    
2. Small-scale exhaustive ceiling  
    Purpose: show FADLS is close to the discrete global optimum in small-scale settings.
    

### Not recommended

Do not include runtime curve or complexity proxy curve unless specifically required. Runtime is better reported in Table I. Complexity proxy curves are easy to misunderstand because different methods have different per-operation costs.

---

## VIII. Conclusion

### Main purpose

回扣主线，不夸大。

### Logic

1. 本文研究 receive-side MA-aided anti-jamming reception。
    
2. 揭示了 desired 和 jammer multipath channels 在 aperture 上产生不同 position-domain profiles。
    
3. 提出 FADLS，通过 desired-favorable、jammer-weak 和 desired-jammer separability 三类结构指标快速选点。
    
4. 仿真表明 FADLS 在 post-MMSE output SINR 上接近 full-grid SINR-greedy 和 local AO-SCA-CVX，同时显著降低搜索复杂度和 empirical runtime。
    
5. 未来可扩展到 imperfect CSI、多干扰、宽带干扰和二维/三维 MA array 场景。



