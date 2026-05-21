
## 1. idea draft
导师这个思路非常有前瞻性。本质上，这是一个**空间域的 Rake 接收机（Spatial Rake Receiver）**，也就是把原本可能造成频率选择性衰落或破坏性干涉的“多径（Multipath）”，转化为可以提供空间分集和能量增益的“有用信号”。

引入可移动天线（Movable Antennas, MA）作为辅佐，是这个方案的破局点。传统的固定阵列（如 ULA 或 UPA）在形成双波束并进行相位对齐时，受限于固定的天线间距（通常为 $\lambda/2$），其空间分辨率和波束赋形能力是受限的。而 MA 可以通过灵活改变天线位置，重塑阵列流型（Array Manifold），从而完美捕获并合并这两条路径的能量。


### 1. 系统建模与核心机制

假设发射端为单天线（或已固定波束的多天线），接收端配备 $N$ 个可移动天线元素。

信道可以建模为包含直达径（LoS，即主信号）和一条强反射径（NLoS，即 Multipath）的双径几何信道模型：

$$ \mathbf{h}(\mathbf{p}) = \alpha_0 e^{-j \phi_0} \mathbf{a}(\theta_0, \mathbf{p}) + \alpha_1 e^{-j \phi_1} \mathbf{a}(\theta_1, \mathbf{p}) $$

- $\alpha_0, \alpha_1$：主径和多径的复增益（包含路径损耗）。
    
- $\theta_0, \theta_1$：主径和多径的到达角（AoA）。
    
- $\mathbf{p} = [p_1, p_2, \dots, p_N]^T$：MA 的位置矢量。
    
- $\mathbf{a}(\theta, \mathbf{p})$：依赖于 MA 位置的阵列响应矢量。
    

**叠加和（Coherent Combining）的本质：**

接收端的数字波束赋形向量为 $\mathbf{w}$。接收到的信号为 $y = \mathbf{w}^H \mathbf{h}(\mathbf{p}) s + n$。

为了同时接收这两个方向的信号并进行最大比合并（MRC），最优的接收波束其实就是信道本身的共轭（归一化）：

$$ \mathbf{w}_{\text{opt}} = \frac{\mathbf{h}(\mathbf{p})}{\|\mathbf{h}(\mathbf{p})\|} $$

此时，接收到的信号功率达到最大，等于 $\|\mathbf{h}(\mathbf{p})\|^2$。

### 2. MA 在这里的核心作用

如果使用固定阵列，$\|\mathbf{h}_{\text{fixed}}\|^2$ 的值是固定的，由于两条路径的相位差，它们在某些天线上可能会发生**破坏性干涉（Destructive Interference）**。

加入 MA 后，我们可以通过优化位置 $\mathbf{p}$ 来实现两个极其强大的效果：

1. **波束级匹配（空间功率最大化）：** MA 可以移动到特定的位置，使得两条路径的相位在各个天线阵元上实现**建设性干涉（Constructive Interference）**，从而最大化等效信道增益 $\|\mathbf{h}(\mathbf{p})\|^2$。
    
2. **双主瓣重塑：** 相比于固定 $\lambda/2$ 间距的阵列，MA 可以通过稀疏阵列分布扩大等效孔径，形成两个非常尖锐的接收波束，分别精准对准 $\theta_0$ 和 $\theta_1$，同时压低其他方向的旁瓣（降低环境噪声或干扰）。
    

### 3. 优化问题构建

这个思路可以直接落地为一个联合优化问题。目标是最大化合并后的接收信噪比（SNR）：

$$ \max_{\mathbf{w}, \mathbf{p}} \quad |\mathbf{w}^H \left( \alpha_0 \mathbf{a}(\theta_0, \mathbf{p}) + \alpha_1 \mathbf{a}(\theta_1, \mathbf{p}) \right)|^2 $$

$$ \text{s.t.} \quad \|\mathbf{w}\|^2 \leq 1 $$

$$ \quad \quad \mathbf{p} \in \mathcal{C} \quad (\text{MA的移动区域限制}) $$

$$ \quad \quad |p_i - p_j| \geq d_{\min}, \forall i \neq j \quad (\text{避免天线互耦的安全距离}) $$

### 4. 算法求解框架

这是一个典型的非凸优化问题，建议采用交替优化（Alternating Optimization, AO）的方法：

- **步骤 1：固定 MA 位置 $\mathbf{p}$，优化波束 $\mathbf{w}$**
    
    这个问题有闭式解，即标准的 MRC 接收：$\mathbf{w}^* = \frac{\mathbf{h}(\mathbf{p})}{\|\mathbf{h}(\mathbf{p})\|}$。
    
- **步骤 2：固定波束 $\mathbf{w}$（或者将 $\mathbf{w}^*$ 代入原目标函数），优化 MA 位置 $\mathbf{p}$**
    
    由于 $\mathbf{a}(\theta, \mathbf{p})$ 中包含关于位置的高频非线性相位项 $e^{j \frac{2\pi}{\lambda} p_i \sin\theta}$，这部分相对棘手。
    
    - _解法 A：_ 使用连续凸近似（SCA），对相位项进行泰勒展开线性化。
        
    - _解法 B：_ 既然你之前也在看流形优化（Manifold Optimization），由于阵列流型本身的非线性，可以尝试基于梯度的算法（如 PGD）或者流形上的搜索来寻找最优位置。
        
    - _解法 C：_ 粒子群算法（PSO）等启发式算法（用于基准对比）。
        

### 5. 进阶探讨（与抗干扰结合）

如果你想把这个思路和你一直在做的**抗干扰**结合起来，故事会非常漂亮：

“在存在强干扰机（Jammer）的环境中，主干 LoS 径可能正好处于干扰机的波束范围内或被阻挡。此时，我们利用 MA 构造出不规则的双波束，不仅把 LoS 和 Multipath 叠加起来增强信号，**同时还在干扰机的方向 $\theta_J$ 处生成一个极深的零陷（Nulling）**。”



可以。下面这三块可以直接扩展成论文里的理论分析、鲁棒性分析和仿真实验设计。

**1. Theorem 部分：相干位置、有限孔径和 `d_min` 上界**

设

```math
s_0=\sin\theta_0,\quad s_1=\sin\theta_1,\quad \Delta s=s_0-s_1,
```

```math
\Delta\phi=\phi_1-\phi_0,\quad k=\frac{2\pi}{\lambda}.
```

第 `i` 个天线处的双径信道为：

```math
h_i(p_i)=
\alpha_0 e^{-j\phi_0}e^{jkp_i s_0}
+
\alpha_1 e^{-j\phi_1}e^{jkp_i s_1}.
```

于是

```math
|h_i(p_i)|^2
=
\alpha_0^2+\alpha_1^2
+
2\alpha_0\alpha_1
\cos(kp_i\Delta s+\Delta\phi).
```

因此总接收功率为：

```math
\|\mathbf h(\mathbf p)\|^2
=
N(\alpha_0^2+\alpha_1^2)
+
2\alpha_0\alpha_1
\sum_{i=1}^N
\cos(kp_i\Delta s+\Delta\phi).
```

**Theorem 1: Coherence lattice**

如果 `\Delta s \neq 0`，第 `i` 个天线实现完全建设性叠加的充要条件是：

```math
kp_i\Delta s+\Delta\phi=2\pi m_i,\quad m_i\in\mathbb Z.
```

因此建设性相干位置集合为：

```math
p_i
=
\frac{\lambda}{\Delta s}
\left(
m_i-\frac{\Delta\phi}{2\pi}
\right).
```

相邻建设性位置的周期为：

```math
T_{\rm coh}
=
\frac{\lambda}{|\Delta s|}
=
\frac{\lambda}{|\sin\theta_0-\sin\theta_1|}.
```

如果所有天线都落在该 lattice 上，则

```math
\|\mathbf h(\mathbf p)\|^2_{\max}
=
N(\alpha_0+\alpha_1)^2.
```

如果所有天线都落在破坏性 lattice 上：

```math
kp_i\Delta s+\Delta\phi=(2m_i+1)\pi,
```

则

```math
\|\mathbf h(\mathbf p)\|^2_{\min}
=
N(\alpha_0-\alpha_1)^2.
```

这个 theorem 就是文章的第一个核心 insight：**MA 的最优相干位置不是黑盒结果，而是由 AoA 差和路径相位差决定的空间 lattice。**

**Theorem 2: 有限孔径和 `d_min` 可行性**

设移动区域为：

```math
\mathcal C=[p_{\min},p_{\max}],\quad A=p_{\max}-p_{\min}.
```

落在移动区域内的建设性 lattice 点数量为 `M_c`。定义：

```math
q=\left\lceil \frac{d_{\min}}{T_{\rm coh}}\right\rceil.
```

为了保证任意两个相邻选中 lattice 点间距不小于 `d_min`，至少需要间隔 `q` 个 lattice index。因此在有限孔径和最小间距约束下，最多可以放置的完全相干天线数为：

```math
N_{\rm coh}^{\max}
=
\left\lfloor
\frac{M_c-1}{q}
\right\rfloor+1.
```

如果

```math
N\leq N_{\rm coh}^{\max},
```

则存在一组位置 `\mathbf p`，使得所有天线都实现完全建设性双径叠加。

特别地，因为

```math
|\sin\theta_0-\sin\theta_1|\leq 2,
```

所以

```math
T_{\rm coh}\geq \frac{\lambda}{2}.
```

因此当工程上采用常见安全距离

```math
d_{\min}\leq \frac{\lambda}{2}
```

时，有

```math
q=1.
```

也就是说：**最小间距约束通常不会破坏相干 lattice，可行性主要由移动孔径内能容纳多少个 lattice 点决定。**

**Theorem 3: 近似相干下界**

如果第 `i` 个天线不能精确落在相干 lattice 上，而是存在相位误差：

```math
\epsilon_i=
kp_i\Delta s+\Delta\phi-2\pi m_i,
```

且

```math
|\epsilon_i|\leq \epsilon_{\max},
```

则

```math
\|\mathbf h(\mathbf p)\|^2
\geq
N(\alpha_0^2+\alpha_1^2)
+
2N\alpha_0\alpha_1\cos(\epsilon_{\max}).
```

相对于理想完全相干功率的下界为：

```math
\frac{\|\mathbf h(\mathbf p)\|^2}
{N(\alpha_0+\alpha_1)^2}
\geq
\frac{
\alpha_0^2+\alpha_1^2
+
2\alpha_0\alpha_1\cos(\epsilon_{\max})
}
{(\alpha_0+\alpha_1)^2}.
```

这个 theorem 可以支撑后面的鲁棒性分析。

**2. 鲁棒性分析**

鲁棒性可以统一写成“相干相位误差”模型。

实际误差来自 AoA、路径相位、位置量化：

```math
\epsilon_i
\approx
k p_i \delta_s
+
\delta_\phi
+
k\Delta s\,\delta p_i,
```

其中

```math
\delta_s
=
(\sin\theta_0-\sin\theta_1)
-
(\sin\hat\theta_0-\sin\hat\theta_1),
```

```math
\delta_\phi
=
(\phi_1-\phi_0)-(\hat\phi_1-\hat\phi_0).
```

如果角度误差较小，用弧度表示：

```math
|\delta_s|
\lesssim
|\cos\theta_0||\delta\theta_0|
+
|\cos\theta_1||\delta\theta_1|.
```

如果移动区域以 0 为中心，且

```math
|p_i|\leq A/2,
```

则有保守界：

```math
|\epsilon_i|
\leq
k\frac{A}{2}|\delta_s|
+
|\delta_\phi|
+
k|\Delta s||\delta p_i|.
```

如果位置量化步长为 `\Delta p_q`，则

```math
|\delta p_i|\leq \frac{\Delta p_q}{2}.
```

所以：

```math
\epsilon_{\max}
=
k\frac{A}{2}|\delta_s|
+
|\delta_\phi|
+
k|\Delta s|\frac{\Delta p_q}{2}.
```

然后直接代入 Theorem 3，就得到 SNR 鲁棒下界。

这部分可以形成一个很清楚的结论：

- AoA 误差会随着孔径 `A` 放大；
- 路径相位误差会整体平移 coherence lattice；
- 位置量化误差与 `|\Delta s|` 成正比；
- 大孔径有利于方向图分辨率，但也更敏感于角度估计误差。

互耦可以这样建模：

```math
\mathbf h_{\rm c}(\mathbf p)
=
\mathbf C(\mathbf p)\mathbf h(\mathbf p),
```

其中 `C(p)` 是位置相关互耦矩阵。如果

```math
\|\mathbf C(\mathbf p)-\mathbf I\|_2\leq \epsilon_c,
```

则

```math
(1-\epsilon_c)^2\|\mathbf h\|^2
\leq
\|\mathbf h_{\rm c}\|^2
\leq
(1+\epsilon_c)^2\|\mathbf h\|^2.
```

对于双波束零陷，互耦会导致零陷泄漏：

```math
|w^H(\mathbf C-\mathbf I)a(\theta_J)|
\leq
\|w\|\epsilon_c\|a(\theta_J)\|.
```

所以 `||w||^2` 越大，零陷越不鲁棒。这也解释了为什么仿真里要画 `||w||^2` 和 `cond(A_c^H A_c)`。

双波束部分还可以加入宽零陷设计。如果干扰角度有误差：

```math
\theta_J \rightarrow \theta_J+\delta\theta_J,
```

则

```math
w^H a(\theta_J+\delta\theta_J)
\approx
w^H a(\theta_J)
+
\delta\theta_J w^H a'(\theta_J).
```

普通零陷只保证第一项为 0。如果想鲁棒，可以额外加 derivative null：

```math
w^H a'(\theta_J)=0.
```

这样牺牲一个自由度，但零陷对角度误差更稳定。

**3. 完整仿真设计**

建议至少补这些图。

第一组：相干增益验证。

横轴可以是 `N`、孔径 `A/lambda` 或角度间隔 `|\sin\theta_0-\sin\theta_1|`。

比较方法：

- Fixed ULA
- Random feasible MA
- Black-box PGD / PSO MA
- Proposed coherence-lattice MA
- Proposed coherence-lattice + local refinement

指标：

```math
{\rm SNR}=10\log_{10}\frac{\|\mathbf h(\mathbf p)\|^2}{\sigma^2}
```

```math
C_{\rm mean}
=
\frac{1}{N}
\sum_i
\cos(kp_i\Delta s+\Delta\phi)
```

预期结果：

- Proposed coherence-lattice MA 接近理论上界 `N(\alpha_0+\alpha_1)^2`；
- Fixed ULA 随相位和角度出现起伏；
- 当角度差很小时，`T_coh` 很大，有限孔径内容纳的相干点变少，MA 增益下降。

第二组：有限孔径和 `d_min` 可行性。

画：

```math
N_{\rm coh}^{\max}
=
\left\lfloor
\frac{M_c-1}{q}
\right\rfloor+1
```

随 `A/lambda`、`d_min/lambda`、`|\Delta s|` 的变化。

这张图很重要，因为它把 theorem 和仿真连起来：什么时候能让所有天线完全相干，什么时候只能部分相干。

第三组：双波束/零陷方向图。

比较：

- Fixed ULA + LCMV
- Sparse fixed aperture
- Coherence-lattice MA + LCMV
- Proposed MA dual-beam synthesis

指标：

```math
{\rm PSL}=\max_{\theta\in\Theta_{\rm side}} |w^Ha(\theta,p)|^2
```

```math
{\rm ISL}=\frac{1}{|\Theta_{\rm side}|}
\int_{\Theta_{\rm side}} |w^Ha(\theta,p)|^2d\theta
```

```math
{\rm NullDepth}
=
10\log_{10}|w^Ha(\theta_J,p)|^2.
```

同时画：

```math
\|w\|^2,\quad
\kappa(A_c^H A_c).
```

第四组：鲁棒性。

横轴分别扫：

- AoA 误差标准差 `sigma_theta`
- 路径相位误差 `sigma_phi`
- 位置量化步长 `Delta p_q`
- 互耦强度 `epsilon_c`
- 干扰角误差 `delta theta_J`

指标：

- SNR loss
- Mean coherence
- Null depth degradation
- PSL / ISL
- Outage probability，比如 `Pr(SNR loss > 3 dB)`

第五组：复杂度。

比较：

- Black-box PGD
- PSO
- Coherence-lattice closed-form construction
- Coherence-lattice initialization + local search
- Dual-beam random/local search

复杂度可以写成：

```text
Coherence lattice: O(M_c log M_c)
PGD: O(I N)
PSO: O(P I N)
Dual-beam search: O(I R (N G + K^3))
```

其中 `G` 是角度网格数，`K` 是约束方向数，`R` 是每轮随机候选数。

最后论文主线可以这样收束：

```text
The proposed method first exploits the closed-form coherence lattice to obtain interpretable and low-complexity MA positions for multipath coherent combining. Then, by using these positions as geometry-aware initialization, the array can be further refined for dual-beam synthesis and interference nulling. This bridges channel-gain maximization and beampattern shaping under a unified movable-antenna framework.
```

## 2. Paper outlines
I. Introduction

II. System Model and Problem Formulation
    A. Two-Path Movable-Antenna Channel Model
    B. Received Power Maximization Problem

III. Coherence-Lattice Analysis for Multipath Combining
    A. Channel Power Decomposition
    B. Constructive Coherence Lattice
    C. Finite-Aperture and Minimum-Spacing Feasibility

IV. Placement Optimization and Beamforming Extension
    A. Placement Optimization
    B. LCMV Beam Synthesis with Optimized MA Positions

V. Robustness Analysis
    A. AoA and Phase Perturbation
    B. Position Quantization
    C. Mutual Coupling / Null Mismatch

VI. Simulation Results

VII. Conclusion
## 3. relative work research
结论：**这个思想的底层原理有人做过，但你现在这个写法不一定完全撞；关键看你怎么 claim。**

更准确地说：

### 1. “移动天线通过位置调相，让多径建设性叠加”已经有人做过

这个不能说是你的原创。已有 MA 文献已经明确讲过：MA 位置优化可以调节不同路径的相位，使多径分量建设性叠加，从而提高 channel power gain。比如 MA tutorial 里就直接说，MA 位置可以调节多径复系数的相位，使其 constructively superimpose；单路径时只改变相位，不带来 SNR gain，多路径时才有 gain。([arXiv](https://arxiv.org/html/2502.17905v1 "A Tutorial on Movable Antennas for Wireless Networks"))

所以这句话不能作为强 claim：

> We are the first to reveal that MA improves SNR by coherent multipath combining.

这不安全。

---

### 2. “two-path channel gain 展开成 cosine periodic function”也有人做过

Zhu, Ma, Zhang 的 TWC 2024 **Modeling and Performance Analysis for Movable Antenna Enabled Wireless Communications** 已经分析了 deterministic two-path 情况。他们指出 two-path channel gain 在空间中因为 cosine function 呈现 periodic character，而且 AoA 差越大，周期越小；他们还给出了最大 channel gain 的 tight upper bound 以及达到 upper bound 的位置条件。([arXiv](https://arxiv.org/html/2210.05325v2 "Modeling and Performance Analysis for Movable Antenna Enabled Wireless Communications"))

他们还进一步分析了 three-path 和 multi-path 情况：three-path 最大点是若干线的交点，multi-path channel gain 类似 2D DTFT，并讨论了 period / approximate period。([arXiv](https://arxiv.org/html/2210.05325v2 "Modeling and Performance Analysis for Movable Antenna Enabled Wireless Communications"))

所以你这个公式：

 $$
|\mathbf h(\mathbf p)|^2

N(\alpha_0^2+\alpha_1^2)  
+  
2\alpha_0\alpha_1  
\sum_i  
\cos(kp_i\Delta s+\Delta\phi)  
$$

里面的 **cosine 展开、周期性、建设性叠加位置**，本质上和已有 MA performance analysis 很接近。

---

### 3. “coherence lattice”这个名字本身我没看到是主流叫法，但不能只靠改名字算创新

我没有看到 MA 文献普遍把这个叫 **coherence lattice**。已有工作更多叫：

- periodic character of channel gain；
    
- maximum-gain positions；
    
- position satisfying phase alignment；
    
- constructive superposition positions。
    

所以你叫 coherence lattice 是可以的，而且听起来比“periodic maximum points”更有结构感。

但是注意：**不能把换名字包装成主要 novelty。**  
更稳的写法是：

> Motivated by the periodic channel-gain structure of MA channels, we characterize the constructive-combining positions as a coherence lattice and further analyze its feasibility under finite aperture and minimum-spacing constraints.

这样既承认已有周期性分析，又把你的扩展放出来。

---

### 4. 你真正可能有新意的是后面这几件事

你的强点不应该是“发现 cosine periodicity”，而应该是：

#### A. 从单 MA channel gain 扩展到 (N)-element MA placement

已有经典分析更多偏 single receive MA 的 channel gain map。你这里是

$$
 
|\mathbf h(\mathbf p)|^2

\sum_{i=1}^N |h_i(p_i)|^2  

$$

也就是多个可移动阵元分别选 lattice points。这个可以写成：

> Unlike single-MA channel-gain maximization, we characterize the number of array elements that can be simultaneously placed at constructive-combining locations.

这比单点最大化更有阵列设计味道。

---

#### B. 有限孔径 + 最小间距下最多能放几个 coherent antennas

这个是你比较有价值的部分：

 
$$
T_{\rm coh}

\frac{\lambda}{|\sin\theta_0-\sin\theta_1|}  

$$
$$
q=  
\left\lceil  
\frac{d_{\min}}{T_{\rm coh}}  
\right\rceil  
$$
$$
N_{\rm coh}^{\max}

\left\lfloor  
\frac{M_c-1}{q}  
\right\rfloor+1.  
$$


这个比单纯说“最大点周期存在”更进一步，因为你把它变成了 **array placement feasibility condition**。

这里可以 claim：

> We derive an explicit feasibility condition for the number of movable antennas that can simultaneously achieve constructive LoS-NLoS combining under finite-aperture and inter-element spacing constraints.

这个相对安全，也更像 TVT contribution。

---

#### C. 和 anti-jamming / LCMV / overloaded nulling 结合

如果你的主论文方向是 DMA/MA-enabled anti-jamming，那你的新意应该落在：

> coherence lattice is not only used for SNR enhancement, but also provides additional spatial degrees of freedom for overloaded anti-jamming and dual-beam/null synthesis.

也就是说，你不是单纯做 MA channel gain maximization，而是把 lattice-based multipath combining 放进抗干扰框架里。

这才是和已有 MA performance analysis 拉开距离的地方。

---

### 5. 所以这部分在论文里该怎么写 claim？

不建议写：

```latex
We first derive the coherence lattice of movable antennas for multipath coherent combining.
```

这个太危险，因为 two-path maximum-gain condition 已经有人做过。

更建议写：

```latex
We characterize the constructive LoS-NLoS combining positions of an MA array as a spatial coherence lattice. Different from existing single-MA channel-gain analyses, the proposed characterization is further used to quantify how many movable antenna elements can be simultaneously placed at constructive-combining locations under finite-aperture and minimum-spacing constraints.
```

或者更强一点：

```latex
Building upon the periodic channel-gain structure of MA channels, we derive an explicit coherence-lattice feasibility condition for multi-element MA placement, which reveals the joint impact of angular separation, aperture size, and inter-element spacing on achievable coherent combining gain.
```

这个表述比较安全，因为它没有否认已有工作。

---

### 6. 我的判断

| 内容                                            |     是否已有 |                 你能不能用 |
| --------------------------------------------- | -------: | --------------------: |
| 多径通过 MA 位置实现建设性叠加                             |       已有 | 可以作为背景，不要 claim first |
| two-path channel gain 的 cosine periodicity    |       已有 |         可以引用后再改写成你的模型 |
| 最大点/相干点的周期性位置                                 |   已有类似结果 |      可以用，但要避免过度 claim |
| “coherence lattice”这个命名                       |     不算主流 |            可以作为你的分析语言 |
| (N) 个 MA 同时选 lattice points                   |    可能有新意 |     可以作为 contribution |
| 有限孔径 + (d_{\min}) 下 (N_{\rm coh}^{\max})      |    比较有价值 |                 建议重点写 |
| 和 anti-jamming / LCMV / overloaded nulling 结合 | 最可能形成差异化 |                应该作为主线 |

---

最终建议：**这个部分可以保留，而且应该写，但不要把它包装成“首次发现 MA 的多径相干增益”。**

你应该把它定位成：

> 已有 MA 文献发现了 channel gain 的空间周期性；本文进一步将其结构化为 coherence lattice，并用于多阵元 placement feasibility、有限孔径分析、以及抗干扰 beam synthesis。

这样既不会撞已有工作，又能把你的理论部分立住。