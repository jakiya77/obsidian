# Physics-Guided Dual-Favorable Position Selection
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

## III. Proposed Physics-Guided Dual-Favorable Position Selection

### ### Main purpose

第三节是全文的方法核心。它需要完成从物理结构到具体算法的转化。

本节不应该只是单纯写算法步骤，而应该先说明：

> MA 位置选择并不是完全黑箱的非凸搜索问题。  
> desired channel 和 jamming channel 在有限孔径上会形成可利用的位置域起伏结构。  
> PG-DFPS 正是利用这种结构，先进行物理引导的候选点筛选，再进行集合级位置选择。

因此本节的整体逻辑应该是：


$$
\text{位置域信道结构}  
\rightarrow  
\text{dual-favorable profile}  
\rightarrow  
\text{候选池预筛选}  
\rightarrow  
\text{集合级 desired-jammer separability}  
\rightarrow  
\text{PG-DFPS 贪心选点}  
\rightarrow  
\text{复杂度讨论}  
$$


---

## A. Position-Domain Dual-Favorable Profiles

## A. 位置域双有利信道特征

### 这一小节的目的

这一小节回答一个核心问题：

> 为什么 MA 位置选择不是盲目搜索，而是有物理结构可以利用？

### 写作逻辑

由于 desired channel 和 jamming channel 都由多径叠加形成，因此单个 MA 阵元位于不同位置时，接收到的 desired signal 和 jammer strength 并不相同。也就是说，(h(p)) 和 (g(p)) 会随着位置 (p) 在有限孔径内发生空间起伏。

因此，可以定义 desired-channel gain profile：

[  
H(p)=  
\frac{|h(p)|^2}  
{\max_{q\in\mathcal A}|h(q)|^2}.  
]

其中，(H(p)) 越大，说明位置 (p) 对 desired signal 越有利。

同理，定义 jamming-channel gain profile：

[  
G(p)=  
\frac{|g(p)|^2}  
{\max_{q\in\mathcal A}|g(q)|^2}.  
]

其中，(G(p)) 越小，说明位置 (p) 处 jammer 越弱。为了更直观地表示 jammer weakness，可以使用：

[  
1-G(p).  
]

因此，如果一个位置同时满足：

[  
H(p)\ \text{large},  
\quad  
1-G(p)\ \text{large},  
]

那么该位置就是一个 dual-favorable position，即：

> desired channel strong and jammer channel weak.

基于这个思想，定义单点 dual-profile score：

# [  
s_{\rm dual}(p)

H(p)\bigl(1-G(p)\bigr).  
]

该分数用于进行候选点预筛选。也就是说，PG-DFPS 不是从所有网格点中盲目搜索，而是先根据 (H(p)(1-G(p))) 从有限孔径中选出更有物理意义的候选点集合。

候选池可以写成：

# [  
\mathcal C

{\rm Top}\text{-}C  
\left{  
p\in\mathcal A_\Delta:  
s_{\rm dual}(p)  
\right},  
]

其中，(\mathcal A_\Delta) 是离散化后的候选网格，(C) 是候选池大小。

### 这一小节的核心句

PG-DFPS uses the position-domain desired and jamming profiles as a physical prior to reduce the search range before directly evaluating the output SINR.

### Fig. 1 放置位置

Fig. 1 建议放在这一小节。

Fig. 1 应该展示：

- (H(p))
    
- (1-G(p))
    
- (H(p)(1-G(p)))
    
- PG-DFPS selected positions
    

图注想传达的核心信息是：

> PG-DFPS 所选位置集中在 desired-favorable and jammer-weak regions 附近，说明它利用的是位置域物理结构，而不是黑箱搜索。

---

## B. Set-Level Desired-Jammer Separability

## B. 集合级期望-干扰可分离性

### 这一小节的目的

这一小节回答第二个核心问题：

> 为什么不能只选 (H(p)(1-G(p))) 最大的前 (N) 个点？

原因是接收机不是逐点独立处理信道，而是对整个 MA 阵列的接收向量进行合并。最终影响 output SINR 的不是单个位置的 (h(p)) 和 (g(p))，而是整个位置集合对应的 channel vectors：

# [  
\mathbf h_{\mathcal S}

[h(u_1),\ldots,h(u_M)]^T,  
]

# [  
\mathbf g_{\mathcal S}

[g(u_1),\ldots,g(u_M)]^T.  
]

即使每个单点都具有较高的 dual-profile score，最终形成的 (\mathbf h_{\mathcal S}) 和 (\mathbf g_{\mathcal S}) 仍可能高度相关。如果 desired channel vector 和 jamming channel vector 非常相似，那么接收合并器很难有效区分 desired signal 和 jammer。

因此，需要引入集合级 desired-jammer separability：

# [  
\rho_{hg}(\mathcal S)

\frac{  
|\mathbf h_{\mathcal S}^H\mathbf g_{\mathcal S}|^2  
}{  
\lVert \mathbf h_{\mathcal S} \rVert^2  
\lVert \mathbf g_{\mathcal S} \rVert^2+\epsilon  
}.  
]

其中，(\rho_{hg}(\mathcal S)) 越小，说明 desired 和 jammer 在所选阵列位置上的空间可分离性越好。

这一点也和第二节中的展开式一致，因为 output SINR 中存在：

[  
|\mathbf g^H(\mathbf p)\mathbf h(\mathbf p)|^2  
]

这一项。该项越大，说明 desired 和 jammer channel vectors 越相关，会削弱接收机区分 desired signal 和 jammer 的能力。

### 这一小节的核心句

Single-position dual favorability is useful for candidate screening, but the final MA position set should further account for the vector-level separability between the desired and jamming channels.

---

## C. PG-DFPS-Based Greedy Position Selection

## C. 基于 PG-DFPS 的贪心位置选择

### 这一小节的目的

这一小节把前面的物理结构转化成具体算法。

PG-DFPS 包含两个阶段：

1. **candidate pre-screening**：基于 (s_{\rm dual}(p)=H(p)(1-G(p))) 构造候选池；
    
2. **set-level greedy selection**：从候选池中逐根选择 MA 位置，并使用集合级评分判断当前组合是否有利。
    

### 写作逻辑

首先，从空集合开始：

[  
\mathcal S_0=\emptyset.  
]

在第 (n) 根 MA 选择时，对每个候选位置 (p\in\mathcal C)，构造临时集合：

# [  
\mathcal S_{\rm tmp}

\mathcal S_{n-1}\cup{p}.  
]

如果该临时集合不满足最小间距约束，则直接跳过。

然后，计算集合平均 desired profile：

# [  
\bar H(\mathcal S)

\frac{1}{|\mathcal S|}  
\sum_{u\in\mathcal S}H(u).  
]

计算集合平均 jammer profile：

# [  
\bar G(\mathcal S)

\frac{1}{|\mathcal S|}  
\sum_{u\in\mathcal S}G(u).  
]

其中，(1-\bar G(\mathcal S)) 表示集合级 jammer weakness。

结合 desired-channel gain、jammer weakness 和 desired-jammer separability，定义 PG-DFPS 集合级评分：

# [  
s_{\rm PG\text{-}DFPS}(\mathcal S)

\omega_H\log\bigl(\bar H(\mathcal S)+\epsilon\bigr)  
+  
\omega_G\log\bigl(1-\bar G(\mathcal S)+\epsilon\bigr)  
+  
\omega_\rho\log\bigl(1-\rho_{hg}(\mathcal S)+\epsilon\bigr).  
]

三个部分分别对应：

- (\bar H(\mathcal S))：增强 desired channel；
    
- (1-\bar G(\mathcal S))：削弱 jammer channel；
    
- (1-\rho_{hg}(\mathcal S))：提升 desired-jammer separability。
    

在每一步选择中，PG-DFPS 选择使 (s_{\rm PG\text{-}DFPS}(\mathcal S_{\rm tmp})) 最大且满足间距约束的位置。重复该过程，直到选出 (N) 个 MA 位置。

### 这一小节的核心句

Different from naive dual-profile selection, PG-DFPS evaluates the quality of a position set rather than each position independently.

### Algorithm box

这里放 Algorithm 1。

算法名称建议写成：

[  
\text{Algorithm 1: PG-DFPS-Based Greedy MA Position Selection}  
]

输入包括：

- candidate grid (\mathcal A_\Delta)
    
- desired profile (H(p))
    
- jammer profile (G(p))
    
- channel samples (h(p)), (g(p))
    
- number of MAs (N)
    
- minimum spacing (d_{\min})
    
- candidate pool size (C)
    
- weights (\omega_H,\omega_G,\omega_\rho)
    

输出：

- selected MA position vector (\mathbf p_{\rm PG\text{-}DFPS})
    

---

## D. Complexity Discussion

## D. 复杂度讨论

### 这一小节的目的

这一小节回答：

> PG-DFPS 为什么比 direct SINR-driven search 更低复杂度？

### 写作逻辑

设离散候选网格大小为：

[  
K=|\mathcal A_\Delta|.  
]

PG-DFPS 首先计算所有位置的 single-position dual-profile score，然后排序得到候选池。排序复杂度为：

[  
\mathcal O(K\log K).  
]

之后的贪心选择只在候选池 (\mathcal C) 中进行，其中：

[  
C\ll K.  
]

因此，PG-DFPS 避免了在每一步都对全网格位置进行 output-SINR evaluation。

相比之下，full-grid SINR-greedy 每一步都需要扫描大量候选点，并对每个临时位置集合计算 (\Gamma^\star(\mathbf p))，因此 output-SINR evaluation 次数较多。

AO-SCA-CVX 方法虽然可以在连续孔径内进行局部 refinement，但通常需要多轮 SCA 和 repeated CVX-based subproblem solving，因此实际运行时间较高，并且结果依赖初始化和 trust-region 设置。

因此，PG-DFPS 的优势不是声称全局最优，而是：

> 用物理引导的 profile screening 和集合级 greedy selection，减少对全网格 output-SINR search 和 CVX-based local refinement 的依赖。

### 本小节核心句

PG-DFPS provides a low-complexity physics-guided position selection strategy by replacing repeated full-grid output-SINR evaluations with position-domain profile screening and set-level greedy selection.

---

## Section III 结尾段

本节最后建议加一个短段，防止审稿人误解 PG-DFPS 的 score 是最终性能指标。

可以写：

Although the PG-DFPS score is used for low-complexity position selection, it is not used as the final performance metric. For fair comparison, all schemes are evaluated using the same maximum output SINR (\Gamma^\star(\mathbf p)) defined in Section II.
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