# 1. 论文核心主线

## 一句话版本

This paper investigates receive-side movable antenna (MA)-aided anti-jamming reception under finite-aperture and minimum-spacing constraints. 

Instead of treating MA position optimization as a black-box non-convex problem, we reveal that desired and jamming multipath channels induce different position-domain spatial variations over the aperture. 
Based on this observation, we propose a physics-guided FADLS algorithm that selects MA positions from desired-favorable, jammer-weak, and desired-jammer separable regions. The proposed method achieves competitive post-MMSE output SINR compared with full-grid SINR-greedy and local AO-SCA-CVX baselines, while avoiding repeated receiver-level SINR search and CVX-based convex subproblem solving.

中文理解就是：

> 本文研究接收端 MA 阵列抗干扰。
> 
> 传统方法把位置优化当成黑箱非凸优化问题，先揭示 desired 和 jammer 在孔径上的位置域变化结构，然后基于 desired 强、jammer 弱、desired/jammer 可分离三个特征做快速选点。
> 
> 最终证明 FADLS 性能接近强 baseline，但复杂度和运行时间明显更低。

---

# 2. 你的论文不是 uplink 主线，而是 receive-side anti-jamming 主线

之前这句：

```text
We consider a narrowband far-field uplink anti-jamming communication system...
```

建议不要写 uplink，改成：

```text
We consider a narrowband far-field receive-side anti-jamming communication system...
```

因为你的重点不是“上行链路”，而是：

```text
单天线 desired transmitter → 多天线 MA receiver
单天线 jammer → 干扰 receiver
receiver 通过移动天线位置增强 desired、削弱 jammer
```

所以论文主线应叫：

> **Receive-side MA-aided anti-jamming reception**

而不是：

> uplink MA communication

---

# 3. 论文逻辑链条

## 第一步：问题背景

传统接收阵列抗干扰主要依赖数字端波束形成，例如 MMSE / MVDR。

但是固定阵列有一个限制：

> 阵元位置固定，空间采样点固定，无法利用 aperture 内不同位置的信道强弱变化。

MA 引入了一个新的自由度：

[  
\mathbf p=[p_1,p_2,\ldots,p_N]^T  
]

通过改变接收阵元位置，可以改变 desired channel 和 jammer channel：

[  
\mathbf h(\mathbf p),\quad \mathbf g(\mathbf p)  
]

最终改变 post-MMSE output SINR：

# [  
\Gamma_{\rm out}

P_s\mathbf h^H  
(P_j\mathbf g\mathbf g^H+\sigma^2\mathbf I)^{-1}  
\mathbf h.  
]

---

## 第二步：现有方法的问题

现有 MA 位置优化通常有两类：

### 1. 全网格 / 黑箱搜索

例如 full-grid SINR-greedy，每一步都扫描全部 (M) 个网格点，并计算 receiver-level SINR。

问题是：

```text
搜索点多；
每个候选点都要计算 post-MMSE SINR；
复杂度随 M 和 N 增长较快。
```

---

### 2. AO-SCA-CVX 优化

AO-SCA-CVX 通过局部凸近似和 CVX 求解连续位置 refinement。

问题是：

```text
本质是局部优化；
依赖初始化和 trust region；
需要反复 CVX solve；
实测 runtime 很高。
```

你现在实验已经验证：

```text
FADLS:          约 10 ms
SINR-greedy:    约 19 ms
AO-SCA-CVX:     约 9597 ms
```

所以你的叙事不是：

> AO 没用。

而是：

> AO-SCA-CVX 是有意义的局部优化 baseline，但 repeated CVX solve 代价很高，不适合快速重构。

---

# 4. 你的核心洞察

你的关键发现是：

> desired channel 和 jammer channel 在 aperture 上不是均匀的，而是由多径相位叠加形成不同的空间起伏。

定义：

[  
H(p)=\frac{|h(p)|^2}{\max_q |h(q)|^2}  
]

[  
G(p)=\frac{|g(p)|^2}{\max_q |g(q)|^2}  
]

那么：

```text
H(p) 大：desired-favorable position
G(p) 小：jammer-weak position
H(p) 大且 G(p) 小：dual-favorable position
```

但是只看单点还不够，因为接收端最终是一个阵列。  
所以你又引入集合级 separability：

[  
\rho_{hg}(\mathcal S)=  
\frac{  
|\mathbf h_{\mathcal S}^H\mathbf g_{\mathcal S}|^2  
}{  
|\mathbf h_{\mathcal S}|^2|\mathbf g_{\mathcal S}|^2+\epsilon  
}.  
]

这个指标说明：

> selected desired channel vector 和 jammer channel vector 是否相似。

如果 (\rho_{hg}) 小，说明 desired 和 jammer 在接收阵列上更容易被 MMSE 分离。

所以你的方法不是简单找 (H) 最大，也不是简单找 (G) 最小，而是找：

```text
desired 强
jammer 弱
desired/jammer 可分离
```

---

# 5. FADLS 方法主线

你的 FADLS 可以写成两层。

## 第一层：全孔径 profile 预筛选

先计算单点 dual-score：

[  
s_{\rm pre}(p)=H(p)(1-G(p)).  
]

然后从全网格 (\mathcal P) 中选出前 (C) 个候选点：

[  
\mathcal C=\text{Top-}C{H(p)(1-G(p))}.  
]

这一步作用是：

> 从 (M) 个网格点缩小到 (C) 个 candidate pool。

---

## 第二层：集合级 FADLS 贪心选择

第 (n) 根天线选择时，不是只看单点，而是看加入该点后的临时集合：

[  
\mathcal S_{\rm tmp}=\mathcal S_{n-1}\cup{p}.  
]

评分为：

# [  
s_{\rm FADLS}(\mathcal S)

\omega_H\log(\bar H_{\mathcal S}+\epsilon)  
+  
\omega_G\log(1-\bar G_{\mathcal S}+\epsilon)  
+  
\omega_\rho\log(1-\rho_{hg}(\mathcal S)+\epsilon).  
]

其中：

```text
第一项：desired channel gain
第二项：jammer weakness
第三项：desired-jammer separability
```

这就是你的核心算法贡献。

---

# 6. 为什么 FADLS 可以比 AO-SCA-CVX 好

这个现在已经解释清楚了，论文里可以写得很稳。

AO-SCA-CVX 虽然用了 dual-score 排序初始化，但它用的是：

[  
H(p)(1-G(p))  
]

这是单点排序。

而 FADLS 用的是集合级评分：

[  
\bar H_{\mathcal S},\quad  
1-\bar G_{\mathcal S},\quad  
1-\rho_{hg}(\mathcal S).  
]

所以：

```text
AO 初始化：单点好
FADLS 选择：组合好
```

此外 AO-SCA-CVX 后续 refinement 受 trust region 限制：

[  
|\Delta p_n|\le r_{\rm trust}.  
]

它不能全局换点，只能局部微调。

所以如果初始化组合不如 FADLS，AO 不一定能追上。

这就是为什么现在实验中：

```text
FADLS 有时高于 local AO-SCA-CVX
```

是合理的。

---

# 7. 实验主线怎么排

我建议主文最终按这个结构。

---

## Fig. 1：Position-domain mechanism

目的：

> 证明 FADLS 不是黑箱 heuristic，而是由位置域信道结构驱动。

内容：

```text
H(p)
1-G(p)
dual score H(p)(1-G(p))
FADLS selected positions
```

想传达：

> FADLS 所选位置集中在 desired-favorable and jammer-weak regions 附近。

---

## Table I：Complexity / runtime / CVX solves

目的：

> 说明 FADLS 的复杂度优势和实际运行时间优势。

表格建议列：

```text
Method
Selection principle
Search range
CVX solves
Theoretical cost
Runtime
Average SINR
```

重点不要再用一个混合 proxy。

AO-SCA-CVX 的复杂度应该写成：

[  
O(I_{\rm AO}I_{\rm SCA}T_{\rm CVX})  
]

而不是和 FADLS / SINR-greedy 的 candidate-check proxy 混在一起。

---

## Fig. 2：Main performance curves 双子图

这一张是主性能图，两个子图：

```text
(a) SINR vs Number of paths
(b) SINR vs JSR / INR
```

### Fig. 2(a)

验证：

> 多径数增加时，FADLS 仍然有效。

### Fig. 2(b)

验证：

> 干扰强度变化时，FADLS 仍然具有抗干扰能力。

这一张图是论文性能部分的核心。

---

## Fig. 3：Representative beam pattern

目的：

> 给导师和审稿人一个直观感受：FADLS 确实对 jammer 方向形成抑制，同时保持 desired 方向响应。

这张图不要放太多曲线，建议保留：

```text
Fixed ULA
Proposed FADLS
Full-grid SINR-greedy 或 AO-SCA-CVX
```

---

## Fig. 4：CSI / AoA estimation error sensitivity

目的：

> 回应审稿人可能质疑：你依赖 channel profile，如果估计不准怎么办？

建议优先做 AoA error：

[  
\hat\theta=\theta+\Delta\theta.  
]

横轴：

```text
AoA estimation error standard deviation
```

纵轴：

```text
Average output SINR
```

这张图很重要，因为你方法依赖测向 / profile 估计。

---

# 8. 附录内容建议

不要把所有图都放 appendix。Appendix 也要克制。

## 建议保留

### Appendix A：Candidate pool size sensitivity

回答：

> 为什么 (C=300)？

说明：

```text
C 太小性能不足；
C 增大后性能趋于饱和；
C=300 是性能和搜索开销的折中。
```

---

### Appendix B：Small-scale exhaustive ceiling

回答：

> FADLS 离离散全局最优有多远？

说明：

```text
小规模下 FADLS 接近 exhaustive optimum。
```

---

## 不建议放

```text
Runtime curve
Complexity proxy curve
Performance-search tradeoff curve
```

这些图有意义，但容易被误解或重复。  
runtime 放表格即可，complexity 用理论表格即可。

---

# 9. 论文贡献可以这样写

建议贡献点压成三条。

The main contributions of this paper are summarized as follows.

1. We reveal a position-domain dual-favorable structure for receive-side MA-aided anti-jamming reception. Specifically, the desired and jamming multipath channels induce different spatial variations over the finite aperture, which gives rise to desired-favorable and jammer-weak regions.
    
2. Based on this structure, we propose a physics-guided FADLS algorithm for MA position selection. The proposed algorithm jointly exploits desired-channel gain, jammer-channel weakness, and desired-jammer channel separability over the selected MA array, thereby avoiding repeated full-grid receiver-level SINR optimization.
    
3. Simulation results show that the proposed FADLS achieves competitive post-MMSE output SINR compared with full-grid SINR-greedy and local AO-SCA-CVX baselines, while substantially reducing the search effort, avoiding repeated CVX solves, and achieving much lower empirical runtime.
    

---

# 10. 当前推进到哪了

你现在已经推进到这个阶段：

```text
1. 系统模型口径确定：receive-side anti-jamming，不建议写 uplink。
2. 核心机制确定：position-domain dual-favorable structure。
3. 算法定位确定：FADLS 是集合级 physics-guided greedy selection。
4. AO-SCA-CVX 合理性确定：local refinement baseline，不是全局最优。
5. SINR 评价公平性已验证：AO trace SINR = recomputed post-MMSE SINR。
6. 复杂度表达方式确定：理论复杂度表 + runtime 表，不用柱状图和 proxy 曲线。
7. 主文图表框架基本确定：4 图 + 1 表。
```

下一步最该做的是：

> **把 Fig. 1 机制图、Fig. 2 主性能双子图、Table I 复杂度表先定稿，然后补 CSI/AoA error sensitivity。**

你现在已经不是“想法阶段”了，已经进入：

```text
实验收束 + 论文叙事成型阶段
```