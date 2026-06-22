

## 📌 核心结论与当前状态

- **当前结果整体合理**：FADLS 性能在部分情况下略高于 AO-SCA-CVX **并非**评价口径错误，而是因为 AO-SCA-CVX 是基于局部微调的 baseline（Local Refinement），而非全局最优。
    
- **当前阶段任务**：停止怀疑结果，全面转向收束。重点在于完善论文叙事、图表规划、复杂度表述，并补充最后的鲁棒性实验。
    

## 1. 已确认的事项 (Grounded Facts)

### 1.1 exp12 评价口径对齐

- **比对结果**：通过单样本验证，`trace_ao_sca.sinr_linear(end)` 的输出与重新计算的 SINR 完全一致（误差为 0）。
    
- **结论**：AO 曲线记录的是真实的 post-MMSE output SINR，**评价口径正确**，代码无需因此修改。
    

### 1.2 FADLS > AO-SCA-CVX 的物理合理性

现在的性能趋势为：`FADLS ≈ SINR-greedy ≥ AO-SCA-CVX >> Fixed ULA`。

- **AO-SCA-CVX 局限**：采用 dual-score **单点**排序初始化 + 局部连续位置微调。由于受到 Trust Region（`abs(delta) <= trust_radius`）限制，无法进行全局位置替换，极度依赖初始组合。
    
- **FADLS 优势**：采用 **集合级** 评测指标（$\bar H_{\mathcal{S}}$, $1-\bar G_{\mathcal{S}}$, $1-\rho_{hg}(\mathcal{S})$），在选点时就考虑了整个阵列组合的干扰可分性与增益。
    

### 1.3 AO-SCA-CVX 的学术定位

- **修正定位**：绝不能写成“全局上界”，需定位为 **Local AO-SCA-CVX baseline (with dual-profile initialization)**。
    
- **论文标准表述 (English)**：
    
    > "AO-SCA-CVX baseline is initialized by the dual-profile ranking and then refined through local SCA-based convex subproblems. Since the refinement is trust-region constrained and does not perform global position replacement, it serves as a local optimization benchmark rather than a global optimum."
    

## 2. 复杂度与 Runtime 叙事重构

### 2.1 理论复杂度 vs 实测时间

必须在论文中严格将两者解耦，严禁混为一谈：

- **理论复杂度表**：推导其大 $O$ 阶数，例如 $O(M \log M + C N^2)$ vs $O(M N^4)$ vs $O(I_{AO} I_{SCA} T_{CVX})$。
    
- **实测时间 (Empirical Runtime)**：用 Average CPU time (ms) 表示，强调 CVX 带来的真实运算负担。
    

### 2.2 核心数据支撑 (EXP11)

|**算法**|**平均 SINR**|**Runtime (ms)**|**CVX Solves (次)**|**核心性质/选点原理**|
|---|---|---|---|---|
|**Proposed FADLS**|**11.251 dB**|**10.227 ms**|0|候选池结构评分|
|Full-grid SINR-greedy|11.196 dB|19.332 ms|0|全网格接收端级 SINR 穷举评分|
|AO-SCA-CVX baseline|11.060 dB|9597.293 ms|~18.3|局部 CVX 凸子问题反复迭代求解|

- **卖点提炼**：FADLS 有效规避了全网格接收端级的 SINR 逐点计算，同时避免了耗时的反复 CVX 凸子问题求解（`cvx_begin` ... `cvx_end`）。
    

## 3. 论文图表规划 (主文 4 图 + 1 表)

### 🖼️ 主文结构 (Main Body)

- [x] **Fig. 1：位置域机制图**
    
    - **内容**：展示 $H(p)$、$1-G(p)$、dual score 以及 FADLS 最终选中的位置。
        
    - **目的**：打消黑箱质疑，证明算法由 position-domain channel profile 物理驱动。
        
- [ ] **Table I：算法综合对比表**
    
    - **内容**：整合方法、选点原理、搜索范围、CVX 次数、理论复杂度、实测 Runtime 及平均 SINR。
        
- [ ] **Fig. 2：主性能曲线图 (双子图)**
    
    - (a) SINR vs Number of paths (验证路径数鲁棒性)
        
    - (b) SINR vs JSR / INR (验证强干扰下抑制能力)
        
- [ ] **Fig. 3：波束赋形图 (Beam Pattern)**
    
    - **内容**：直观展示 FADLS 在干扰方向形成零陷 (null)，同时保持对期望信号的增益。
        
- [ ] **Fig. 4：误差敏感度分析图 (CSI / Profile Error Sensitivity)**
    
    - **内容**：回应审稿人对理想 CSI 的质疑。
        

### 📦 附录内容 (Appendix / Supplementary)

以下图示珍贵，但不占用主文版面，一律收纳至附录：

- Performance-search tradeoff 曲线（累积搜索点数图）
    
- Candidate pool size sensitivity (候选池大小敏感度)
    
- Small-scale exhaustive ceiling (小规模穷举绝对上界对比)
    
- Runtime curve / Complexity proxy curve
    

## 4. IEEE TVT 故事线与创新点重塑

- **学术定位拔高**：从“低复杂度启发式算法”拔高至 **Physics-guided anti-jamming MA reconfiguration**（物理引导的抗干扰可重构移动天线技术）。
    
- **应用背景贴合**：叙事上主动向 `vehicular/mobile/UAV anti-jamming receiver`（车载/无人机抗干扰接收机）靠拢，突出高动态、低延迟对计算时间的严苛要求，合理化 FADLS 的毫秒级响应优势。
    

### 💡 三大核心贡献 (Contributions)

1. **物理机制揭示**：发现了位置域内的双重优良结构（Desired-favorable + Jammer-weak）。
    
2. **高效算法提出**：提出 FADLS，利用集合级指标实现极低复杂度的天线位置优选。
    
3. **完备仿真验证**：相比 Fixed ULA 性能飞跃，逼近穷举与凸优化上限，但计算开销降低数个数量级。
    

## 📅 后续行动项 (Action Items)

- [x] **Priority 1: 图例与正文表述纠偏**
    
    - 将所有代码和图表中的 `AO-SCA-CVX` / `AO SINR` 修改为 `Local AO-SCA-CVX` 或 `AO-SCA-CVX baseline`。
        
- [ ] **Priority 2: 补充 AoA Error 敏感度实验 (Fig. 4)**
    
    - **设置**：$\hat\theta = \theta + \Delta\theta$。
        
    - **横轴**：AoA error standard deviation；**纵轴**：Average output SINR。
        
- [ ] **Priority 3: 按照 Table I 模板整理复杂度与实验数据**
    
    - 停止使用混合 Proxy 逃避理论复杂度，把大 $O$ 阶数推导落实。
        
- [ ] **Priority 4: 冻结实验，启动论文撰写**
    
    - 停止无限制加图，围绕上述 4 图 1 表的框架开始 Outline 的打磨。
        

`#Paper/TVT` `#Antenna-Selection` `#Fluid-Antenna` `#Anti-Jamming`