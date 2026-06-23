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
    

## III. Proposed Physics-Guided Dual-Favorable Position Selection 

### Main purpose
本节是全文的方法核心，旨在完成从**物理信道结构**到**具体工程算法**的转化。
- 可动天线（MA）的位置选择问题**并不是一个完全黑箱的非凸搜索问题**。
- 期望信道（Desired Channel）和干扰信道（Jamming Channel）在有限孔径上由于多径叠加，会形成天然可利用的**位置域起伏结构**。

PG-DFPS 正是利用这种结构，先进行物理引导的候选点筛选，再进行集合级的位置选择。本节的整体逻辑架构如下：

>[!hint]+ 位置域信道结构 → 双优剖面定义 → 候选池预筛选 → 空间可分离性指标 → 算法步进与指标联合 → 算法计算代价与对比 → 明确指标界限


## A. Position-Domain Dual-Favorable Profiles (位置域双有利信道特征)

### 1. 物理结构起伏

由于期望信道和干扰信道均由多径叠加形成，因此单个 MA 阵元位于不同位置时，接收到的期望信号和干扰强度并不相同。换言之，$h(p)$ 和 $g(p)$ 会随着位置 $p$ 在有限孔径内发生空间起伏。这也是位置选择算法能够利用的底层物理先验。

### 2. 双优剖面定义与候选池预筛选

为此，定义**归一化期望信道增益剖面 (Desired-channel gain profile)**：

$$H(p) = \frac{|h(p)|^2}{\max_{q\in\mathcal{A}}|h(q)|^2}$$

其中 $H(p)$ 越大，说明位置 $p$ 对期望信号越有利（Desired-favorable）。

同理，定义**归一化干扰信道增益剖面 (Jamming-channel gain profile)**：

$$G(p) = \frac{|g(p)|^2}{\max_{q\in\mathcal{A}}|g(q)|^2}$$

其中 $G(p)$ 越小，说明位置 $p$ 处的干扰越弱。为了更直观地表示**干扰弱度 (Jammer weakness)**，定义其补项为：$1-G(p)$。

> [!SUCCESS] 双优位置 (Dual-Favorable Position)
> 
> 如果一个位置同时满足 $H(p)$ 较大且 $1-G(p)$ 较大，即满足**期望信道强、干扰信道弱**，则该位置是一个双优位置。

基于该思想，定义**单点双剖面评分 (Single-position dual-profile score)**：

$$s_{\rm dual}(p) = H(p)\bigl(1-G(p)\bigr)$$

该分数用于进行候选点预筛选，从而避免在全网格盲目搜索。**候选池 (Candidate Pool)** 构建如下：

$$\mathcal{C} = \text{Top-}C \left\{ p\in\mathcal{A}_\Delta: s_{\rm dual}(p) \right\}$$

其中，$\mathcal{A}_\Delta$ 是离散化后的候选网格，$C$ 是指定的候选池大小。

> [!NOTE] 核心结论 (Key Message)
> 
> PG-DFPS uses the position-domain desired and jamming profiles as a physical prior to reduce the search range before directly evaluating the output SINR.

🖼️ **图表埋点：此处插入 Fig. 1**

- **图片内容建议**：在一维/二维空间孔径上，同时绘制出 $H(p)$、$1-G(p)$ 以及相乘后的 $s_{\rm dual}(p)$ 曲线，并高亮标出 PG-DFPS 最终选中的点。
    
- **图注核心信息 (Caption)**：PG-DFPS 所选位置集中在 desired-favorable and jammer-weak regions 附近，说明它利用的是位置域物理结构，而不是黑箱搜索。
    

## B. Set-Level Desired-Jammer Separability (集合级期望-干扰可分离性)

### 1. 独立评分的局限性

回答核心问题：**为什么不能只选 $H(p)(1-G(p))$ 最大的前 $N$ 个点？**

原因在于接收机并不是逐点独立处理信道的，而是对整个 MA 阵列的接收向量进行空间合并。最终影响输出信噪比（Output SINR）的不是孤立单点的信道，而是整个位置集合 $\mathcal{S}$ 对应的**信道向量 (Channel Vectors)**：

$$\mathbf{h}_{\mathcal{S}} = [h(u_1),\ldots,h(u_M)]^T$$

$$\mathbf{g}_{\mathcal{S}} = [g(u_1),\ldots,g(u_M)]^T$$

### 2. 空间可分离性指标

即使每个选定单点都具有极高的双优评分，最终形成的 $\mathbf{h}_{\mathcal{S}}$ 和 $\mathbf{g}_{\mathcal{S}}$ **仍可能高度相关**。如果期望向量与干扰向量过于相似，空间接收合并器将无法在空间上有效区分二者。

因此，引入**集合级期望-干扰相关性**：

$$\rho_{hg}(\mathcal{S}) = \frac{|\mathbf{h}_{\mathcal{S}}^H\mathbf{g}_{\mathcal{S}}|^2}{\lVert \mathbf{h}_{\mathcal{S}} \rVert^2 \lVert \mathbf{g}_{\mathcal{S}} \rVert^2+\epsilon}$$

- $\rho_{hg}(\mathcal{S})$ 越小，说明期望和干扰在所选阵列位置上的**空间可分离性越好**。
    
- 该项与第二节推导出的 Output SINR 展开式中的 $|\mathbf{g}^H(\mathbf{p})\mathbf{h}(\mathbf{p})|^2$ 项直接对应，揭示了向量级相关性对抑噪抗干扰能力的削弱。
    

> [!NOTE] 核心结论 (Key Message)
> 
> Single-position dual favorability is useful for candidate screening, but the final MA position set should further account for the vector-level separability between the desired and jamming channels.

## C. PG-DFPS-Based Greedy Position Selection (基于 PG-DFPS 的贪心位置选择)

### 1. 算法双阶段机制

将前面的物理结构推导转化为具体算法。PG-DFPS 包含两个核心阶段：

1. **候选点预筛选 (Candidate pre-screening)**：基于单点双优评分构造候选池 $\mathcal{C}$。
    
2. **集合级贪心选择 (Set-level greedy selection)**：从候选池中逐根选择 MA 位置，并使用集合级评分进行联合评估。
    

### 2. 算法步进与指标联合定义

算法从空集开始初始化：$\mathcal{S}_0=\emptyset$。

在第 $n$ 根 MA 选择时，对每个候选位置 $p\in\mathcal{C}$，构造临时集合：

$$\mathcal{S}_{\rm tmp} = \mathcal{S}_{n-1}\cup\{p\}$$

- **间距约束检查**：若 $\mathcal{S}_{\rm tmp}$ 中任意点违反最小间距约束 $d_{\min}$，则直接丢弃该候选点。
    

对于通过检查的候选点，计算临时集合的**平均期望剖面值**与**平均干扰剖面值**：

$$\bar{H}(\mathcal{S}) = \frac{1}{|\mathcal{S}|} \sum_{u\in\mathcal{S}}H(u), \quad \bar{G}(\mathcal{S}) = \frac{1}{|\mathcal{S}|} \sum_{u\in\mathcal{S}}G(u)$$

结合期望增益、干扰弱度以及向量可分离性，定义 **PG-DFPS 集合级综合评分**：

$$s_{\rm PG\text{-}DFPS}(\mathcal{S}) = \omega_H\log\bigl(\bar{H}(\mathcal{S})+\epsilon\bigr) + \omega_G\log\bigl(1-\bar{G}(\mathcal{S})+\epsilon\bigr) + \omega_\rho\log\bigl(1-\rho_{hg}(\mathcal{S})+\epsilon\bigr)$$

> 📌 **评分项对应关系：**
> 
> - $\bar{H}(\mathcal{S})$ $\rightarrow$ 增强期望信号信道
>     
> - $1-\bar{G}(\mathcal{S})$ $\rightarrow$ 削弱干扰信号信道
>     
> - $1-\rho_{hg}(\mathcal{S})$ $\rightarrow$ 提升期望与干扰的向量级空间可分离性
>     

在每一步选择中，算法选择使 $s_{\rm PG\text{-}DFPS}(\mathcal{S}_{\rm tmp})$ 最大且满足间距约束的位置。重复该过程，直到选满 $N$ 个 MA 位置。

> [!NOTE] 核心结论 (Key Message)
> 
> Different from naive dual-profile selection, PG-DFPS evaluates the quality of a position set rather than each position independently.

### 📋 算法伪代码框 (Algorithm 1)

|**Algorithm 1: PG-DFPS-Based Greedy MA Position Selection**|
|---|
|**Input:**<br><br>  <br><br>• 候选网格 $\mathcal{A}_\Delta$，期望与干扰剖面 $H(p), G(p)$<br><br>  <br><br>• 信道采样 $h(p), g(p)$<br><br>  <br><br>• MA 天线数量 $N$，最小间距限制 $d_{\min}$<br><br>  <br><br>• 候选池大小 $C$，权重系数 $\omega_H, \omega_G, \omega_\rho$<br><br>  <br><br>**Output:**<br><br>  <br><br>• 选定的 MA 位置向量 $\mathbf{p}_{\rm PG\text{-}DFPS}$|

## D. Complexity Discussion (复杂度讨论)

### 1. 算法计算代价分析

设离散候选网格总数为 $K = |\mathcal{A}_\Delta|$。

- **预筛选阶段**：计算所有位置的单点双优评分并进行排序，复杂度仅为 $\mathcal{O}(K\log K)$。
    
- **贪心选点阶段**：由于 $C \ll K$，后续的贪心搜索完全在缩减后的候选池 $\mathcal{C}$ 中进行，大幅缩减了搜索空间。因此，PG-DFPS 成功避免了在每一步都对全网格位置进行耗时的 Output-SINR 评估。
    

### 2. 基线方案对比

- **全网格 SINR 贪心算法 (Full-grid SINR-greedy)**：作为强基线方案，其每一步都需要扫描海量的候选网格点，并对每个临时位置集合重复计算复杂的矩阵求逆与 $\Gamma^\star(\mathbf{p})$ 评估，计算开销随网格密度呈指数级上升。
    
- **AO-SCA-CVX 算法**：该方案虽然可以在连续孔径内进行局部细化（Refinement），但通常需要多轮的连续凸近似（SCA）迭代，且高度依赖初始化参数与置信域（Trust-Region）的繁琐调优，实际运行时间极高。
    

> [!NOTE] 核心结论 (Key Message)
> 
> PG-DFPS provides a low-complexity physics-guided position selection strategy by replacing repeated full-grid output-SINR evaluations with position-domain profile screening and set-level greedy selection.

## 🔗 章节结尾过渡段 (Section III Summary)

为了防止审稿人误将 PG-DFPS 的联合评分（Score）当成最终的优化性能指标，在此明确区分选择准则与评估指标：

> Although the PG-DFPS score is used for low-complexity position selection, it is not used as the final performance metric. For fair comparison, all schemes are evaluated using the same maximum output SINR $\Gamma^\star(\mathbf{p})$ defined in Section II.