---
codex_dataview: true
type: idea
project: FADLS / TVT论文
area: idea
category: idea
parent: 研究想法
status: active
priority: high
created: '2026-05-23'
updated: '2026-05-25'
summary: 期望信号和多径干扰在位置域分别形成可解析的相干/相消格点；MA 通过位置重构寻找 desired-constructive and jammer-destructive 的 dual-favorable positions；在有限孔径和最小间距约束下，这种位置是否存在、能放多少个、性能如何，是本文要解决的核心问题。
tags:
- idea
- MMSE
- MVDR
- Movable-Antenna
- Anti-Jamming
- Interference
aliases:
- Dual-lattice theory for finite-aperture movable antenna anti-jamming.
---
期望信号和多径干扰在位置域分别形成可解析的相干/相消格点；MA 通过位置重构寻找 desired-constructive and jammer-destructive 的 dual-favorable positions；在有限孔径和最小间距约束下，这种位置是否存在、能放多少个、性能如何，是本文要解决的核心问题。

1. 推导 desired constructive lattice；
2. 推导 jammer destructive lattice；
3. 定义 exact dual-lattice intersection；
4. 说明 exact intersection 不一定存在；
5. 定义 approximate dual-favorable region；
6. 推导 finite-aperture dual-lattice capacity；
7. 再提出 dual-lattice score：

$$
η(p)=∣hs(p)∣2∣gj(p)∣2+ϵ.\eta(p)= \frac{|h_s(p)|^2} {|g_j(p)|^2+\epsilon}.η(p)=∣gj​(p)∣2+ϵ∣hs​(p)∣2​.
$$

多 jammer 版本：

$$
η(p)=∣hs(p)∣2∑qρq∣gq(p)∣2+ϵ.\eta(p)= \frac{|h_s(p)|^2} {\sum_q \rho_q |g_q(p)|^2+\epsilon}.η(p)=∑q​ρq​∣gq​(p)∣2+ϵ∣hs​(p)∣2​.
$$

最后才是：

> 选出满足间距约束的 NNN 个 MA 位置，再接 MRC/MMSE/MVDR 处理残余干扰。



## Fig 1 

![[png：Figure_1 1.png|633]]Position-domain mechanism of dual-lattice MA selection
蓝色曲线 Hn(p)表示 desired normalized power。蓝色高峰对应 desired constructive regions，也就是期望信号两径相干增强的位置。
橙色虚线 1−Gn(p)表示 jammer suppression score。因为 Gn(p)是 jammer normalized power，所以 1−Gn(p)越高，说明 jammer leakage 越低，也就是 jammer destructive regions。
绿色曲线 η(p) 是 dual-lattice score：
$$

\eta(p)=\frac{H_n(p)}{G_n(p)+\epsilon}
$$

它把 desired enhancement 和 jammer suppression 合在一起。绿色曲线的峰值位置就是算法更倾向选择的 MA 位置。

紫色菱形表示 selected MA positions。可以看到它们不是均匀分布的，而是集中在 dual-lattice score 较高的位置附近。

灰色方块表示 fixed ULA positions。它们是均匀分布的，不会主动对齐位置域中的有利区域。

所以这张图支撑的结论是：

> movable antenna 可以利用位置域的 constructive/destructive 结构，把天线移动到 dual-favorable regions；而 fixed ULA 由于位置固定、均匀，可能错过这些有利位置。

Fig. X. Position-domain mechanism of the proposed dual-lattice MA selection. 
The desired normalized power $H_n(p)$, jammer suppression score $1-G_n(p)$, 
and dual-lattice score $\eta(p)$ are plotted over the movable aperture. 
The selected MA positions are concentrated around high-score regions, where the desired signal is reasonably strong and the jammer leakage is suppressed. 
In contrast, the fixed ULA positions are uniformly placed and are not aligned with the position-domain dual-favorable regions. 
The position markers are vertically shifted only for visualization.

这张图展示了所提出 dual-lattice MA 选择方法的位置域机制。蓝色曲线表示期望信号的归一化功率 Hn(p)H_n(p)Hn​(p)，其峰值对应 desired constructive regions；橙色虚线表示干扰抑制得分 1−Gn(p)1-G_n(p)1−Gn​(p)，其峰值对应 jammer destructive regions。绿色曲线为 dual-lattice score η(p)\eta(p)η(p)，用于同时刻画期望信号增强和干扰泄漏抑制。可以看到，所选 MA 位置并非均匀分布，而是集中在 dual-lattice score 较高的位置附近。相比之下，fixed ULA 的位置是均匀固定的，因此无法主动利用这些位置域峰谷结构。

Fig.~X illustrates the position-domain mechanism of the proposed dual-lattice MA selection. 
The desired normalized power $H_n(p)$ and the jammer-suppression score $1-G_n(p)$ exhibit different spatial lattice patterns over the movable aperture. 
The resulting dual-lattice score $\eta(p)$ becomes large at positions where the jammer leakage is strongly suppressed while the desired-channel gain remains non-negligible. 
Therefore, the selected MA elements are concentrated around the high-score regions, whereas the fixed ULA elements are uniformly placed and cannot adapt to the position-domain dual-favorable structure.

图 X 展示了 dual-lattice MA 选择的位置域机制。期望信号功率 Hn(p)H_n(p)Hn​(p) 和干扰抑制得分 1−Gn(p)1-G_n(p)1−Gn​(p) 在可移动孔径内呈现不同的空间晶格结构。由二者共同形成的 dual-lattice score η(p)\eta(p)η(p) 会在干扰泄漏较低且期望信号增益不弱的位置处变大。因此，所选 MA 阵元集中在这些高分区域附近，而 fixed ULA 阵元均匀放置，无法自适应利用这种位置域有利结构。