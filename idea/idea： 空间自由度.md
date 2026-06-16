---
codex_dataview: true
type: idea
project: FADLS / TVT论文
area: idea
category: idea
parent: 研究想法
status: active
priority: high
created: '2026-05-08'
updated: '2026-05-11'
summary: 1. 空间的自由度由时间的多次采样来扩充
tags:
- idea
- DMA
- Movable-Antenna
aliases:
- 空间自由度
---
1. 空间的自由度由时间的多次采样来扩充
2. 我当下的文章中只用过幅度的调节而不是幅相一起或者单独相位调节是否是一种DMA 或者说异构天线


![[Pasted image 20260511100250.png|447]]
根据这个优化问题 使用二次流型约束 来解

$$\min_v \sum_{k=1}^{K} |v^H a_1(\theta_J^k)|^2$$

$$\text{s.t.} \quad |v(n_e)| = 1, \quad n_e = 1, 2, ..., N_E$$
