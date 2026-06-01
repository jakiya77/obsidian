## V. 仿真结果与分析

本节通过数值仿真验证所提出的 FADLS 方法的有效性。仿真的核心目的不是简单比较若干算法曲线，而是围绕前文建立的物理机理逐步验证以下问题：

1. 多径信道是否确实在有限孔径内形成可利用的 desired 强响应区域和 jammer 弱响应区域；
    
2. 有限孔径和最小间距约束是否会限制 strict dual-favorable positions 的可用数量；
    
3. 所提出的 FADLS 是否不仅适用于 two-path 理想场景，也能推广到一般多径场景；
    
4. 在有限目标函数评估预算下，FADLS 是否能够作为高质量初始化，提高局部搜索效率；
    
5. FADLS 中 desired enhancement、jammer suppression 和 desired-jammer decorrelation 三个设计因素是否都对性能提升有贡献。
    

除非特别说明，所有方法最终均采用相同的 maximum output SINR (\Gamma^\star(\mathbf p)) 进行评价。需要强调的是，FADLS score 仅用于物理引导的位置选择，不作为最终性能指标。因此，所有性能比较都是在同一个接收机和同一个 SINR 评价准则下完成的。

---

### A. 仿真设置

考虑一维有限孔径 MA 接收阵列，阵元数量为 (N)，孔径长度为 (A)，最小阵元间距为 (d_{\min})。连续孔径在数值实现时以 (\Delta p=\lambda/100) 进行采样。该采样仅用于数值计算和候选位置搜索，不代表理论模型中的 MA 位置受到离散端口限制。

本文仿真包含两类信道场景。第一类为 two-path 场景，用于展示 desired constructive region 与 jammer weak-response region 的空间结构，并验证 dual-favorable 位置的物理意义。第二类为一般多径场景，其中 desired 和 jammer 均包含多条随机路径，用于验证所提方法在更一般传播环境下的有效性。对于一般多径仿真，每次 Monte Carlo 试验中随机生成 desired 和 jammer 的 AoA 以及复路径系数，并对路径功率进行归一化，以保证不同信道 realization 之间的公平性。

对比方法包括 Fixed ULA、Desired-only selection、Naive dual selection、FADLS、Random search、Random-init BCD/AO 以及 FADLS-init BCD/AO。其中 Fixed ULA 表示固定均匀阵列；Desired-only 仅根据 desired profile 选择位置；Naive dual 根据 (H(p)(1-G(p))) 选择位置；FADLS 进一步考虑 selected antenna set 的 desired-jammer channel correlation；Random search 和 BCD/AO 用于模拟直接黑盒搜索或局部优化；FADLS-init BCD/AO 则使用 FADLS 结果作为局部搜索初始化。

---

### B. Two-Path 机制验证：位置域峰谷结构

首先考虑 representative two-path 场景，绘制有限孔径内的 normalized desired profile (H(p)) 和 jammer profile (G(p))，并标出 Fixed ULA 与 FADLS 所选位置。

该图的目的不是直接展示最终 SINR 增益，而是验证前文的物理机理：在多径传播下，desired channel 和 jammer channel 会在位置域形成不同的峰谷结构。desired constructive regions 对应较大的 (H(p))，而 jammer weak-response regions 对应较小的 (G(p))。FADLS 所选择的位置应尽量靠近 desired 强、jammer 弱的 dual-favorable 区域，而不是像 Fixed ULA 那样固定分布。

仿真结果应说明：FADLS 的位置选择具有明确的物理解释，它利用了多径诱导的空间结构，而不是在孔径内盲目搜索。这为后续的 finite-aperture dual-favorable region 和 set-level selection 提供了直观支撑。

---

### C. 有限孔径下 Dual-Favorable 可用性

接下来分析 finite aperture 对 dual-favorable positions 可用性的影响。前文已经定义 strict dual-favorable region：

# [  
\Omega_{\rm dual}(A;\tau_s,\tau_j)

{p\in\mathcal A:H(p)\ge\tau_s,;G(p)\le\tau_j}.  
]

为了衡量该区域是否能够容纳多个满足最小间距约束的 MA 位置，引入 dual-favorable packing number (N_{\rm dual}^{\rm pack})。该指标表示 strict dual-favorable region 内在几何上最多能够放置多少个满足 (d_{\min}) 的 MA 位置。

仿真中可以通过 heatmap 展示 (N_{\rm dual}^{\rm pack}) 随 (A/\lambda) 和 (T_j/T_s) 的变化。该图的目的在于说明：即使 desired constructive region 和 jammer weak-response region 在理论上存在，它们在有限孔径内也不一定充分重合，或者不一定足够容纳全部 (N) 个 MA。特别是在 aperture 较小、period mismatch 明显或相位对齐不利时，可能出现 (N_{\rm dual}^{\rm pack}<N) 的情况。

该结果验证了一个重要结论：仅依赖 exact lattice intersection 或 strict dual-favorable region 是不够的。因此，实际算法需要 relaxed candidate screening 和 set-level position selection。这也解释了为什么 FADLS 不只是简单选择 (H(p)) 高且 (G(p)) 低的位置，而是进一步考虑 selected set 的整体信道结构。

---

### D. 一般多径下的性能验证

为了验证 FADLS 不仅适用于 two-path 解析场景，还需要在一般多径信道下进行性能比较。该实验采用 (L_s=L_j=4) 的 general multipath channel，每次 Monte Carlo 试验随机生成 AoA 和路径系数。

主性能图建议采用 aperture size (A/\lambda) 作为横轴，average maximum output SINR 作为纵轴。这样可以直接对应本文的核心主题：有限孔径越大，MA 能够利用的位置域机会越多，FADLS 越能发挥其物理引导选位优势。

预期结果应体现以下趋势：Fixed ULA 性能最低，说明固定阵列无法主动利用位置域结构；Desired-only 相比 Fixed ULA 有提升，但由于没有显式避开 jammer-dominant region，在强干扰环境下表现有限；Naive dual 由于同时考虑 desired strength 和 jammer suppression，性能进一步提升；FADLS 由于加入 set-level desired-jammer decorrelation，通常优于 Desired-only 和 Naive dual；FADLS-init BCD/AO 则进一步通过局部 refinement 获得最高或接近最高的 SINR。

该图的核心结论是：FADLS 的作用不是 two-path 特例中的偶然现象，而是在一般多径环境下仍然能够通过 position-domain profiles 和 set-level correlation 选出高质量 MA 位置。

---

### E. 有限搜索预算下的效率比较

接下来验证 FADLS 的有限预算优势。这里的横轴为 objective-function evaluation budget (B)，即允许直接计算 (\Gamma^\star(\mathbf p)) 的次数。该实验用于回答一个关键问题：既然最终目标是最大化 (\Gamma^\star(\mathbf p))，为什么不直接使用 random search、BCD/AO 或其他黑盒优化方法？

结果应表明，Random search 和 Random-init BCD/AO 随着 (B) 增大逐渐提升，但在小预算或中等预算下性能有限。这说明直接黑盒搜索虽然最终可能找到较好的位置，但需要大量目标函数评估。相比之下，FADLS 不依赖大量 (\Gamma^\star(\mathbf p)) 评估，而是通过 (H(p))、(G(p)) 和 (\rho_{hg}(\mathbf p)) 提供一个较好的物理引导初始位置。因此，FADLS 本身在小预算下就能达到较高性能。

更重要的是，当 BCD/AO 使用 FADLS 作为初始化时，FADLS-init BCD/AO 在相同预算下明显优于 Random-init BCD/AO。这说明 FADLS 并不是要取代所有局部优化算法，而是提供了一个高质量的 physics-guided initialization，使后续 local refinement 更快进入高性能区域。

该实验是本文的重要性能证据之一。它支撑的结论是：FADLS 的主要价值在于提高有限预算下的位置搜索效率，而不是宣称超过全局最优搜索。

---

### F. 消融实验：FADLS 各组成部分的作用

为了进一步说明 FADLS 的设计合理性，需要进行 ablation study。该实验固定一般多径信道设置和干扰强度，比较以下方法：

1. Fixed ULA；
    
2. Desired-only：只根据 (H(p)) 选择位置；
    
3. Dual profile：根据 (H(p)(1-G(p))) 选择位置；
    
4. FADLS without correlation：考虑 selected set 的 average desired strength 和 average jammer suppression，但不考虑 (\rho_{hg})；
    
5. Full FADLS：同时考虑 desired strength、jammer suppression 和 desired-jammer correlation；
    
6. FADLS-init BCD/AO：在 Full FADLS 初始化基础上进行局部 refinement。
    

该实验的目的在于证明 FADLS 的增益不是来自某一个单独因素。Desired-only 可以提升 desired gain，但可能选到 jammer 也较强的位置；Dual profile 可以避开部分 jammer-dominant region，但仍然是单点启发式；Full FADLS 进一步考虑 selected antenna set 的 desired-jammer channel correlation，因此能够获得更好的 maximum output SINR。

该结果可以说明：FADLS 的性能提升来自 desired enhancement、jammer suppression 和 set-level decorrelation 的联合设计，而不是简单的 (H(p)) 或 (H(p)(1-G(p))) 排序。

---

### G. 网格精度敏感性

最后，为了说明数值离散化不会影响主要结论，可以给出一个 grid resolution sensitivity 表格。比较 (\Delta p=\lambda/50)、(\lambda/100)、(\lambda/200) 下的 FADLS 平均 SINR 和运行时间。

如果 (\lambda/100) 与 (\lambda/200) 的 SINR 差异很小，而运行时间明显低于更细网格，则可以说明 (\Delta p=\lambda/100) 是数值精度和计算复杂度之间的合理折中。这部分不需要作为大图展示，用一个小表格即可。

---

### H. 仿真结论总结

综上，仿真部分形成如下证据链：

首先，two-path 机制图展示了 desired 和 jammer 在位置域中的峰谷结构，验证了 dual-favorable 选位的物理基础。其次，finite-aperture heatmap 说明 strict dual-favorable region 在有限孔径下不一定足够支持所有 MA，因此需要 relaxed candidate screening 和 set-level selection。然后，一般多径性能图验证 FADLS 不局限于 two-path 特例，而是在 general multipath 环境下仍能提升 maximum output SINR。进一步地，有限预算搜索图表明 FADLS 能够作为高质量初始化，提高 random search 或 BCD/AO 等方法在有限目标函数评估预算下的搜索效率。最后，消融实验验证了 desired enhancement、jammer suppression 和 desired-jammer decorrelation 三个设计因素的必要性。

因此，仿真结果共同证明：所提出的 FADLS 方法能够有效利用 multipath-induced position-domain structure，在有限孔径和有限搜索预算下实现高质量的 MA 位置选择。