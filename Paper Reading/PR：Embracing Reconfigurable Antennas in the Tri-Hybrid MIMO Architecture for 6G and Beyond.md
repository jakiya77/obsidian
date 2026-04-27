
> [!info] Metadata
> 
> - **标题:** Embracing Reconfigurable Antennas in the Tri-Hybrid MIMO Architecture for 6G and Beyond
>     
> - **作者:** Robert W. Heath Jr. (UCSD) 等
>     
> - **期刊:** IEEE TRANSACTIONS ON COMMUNICATIONS
>     
> - **年份:** 2026年1月


>[!warning] 使用的是远场模型


在 **15 GHz** 的工作频率下 ，对多达 **9 种**不同的 MIMO 预编码架构进行了全面的系统级仿真对比 ：


1. **对比阵营：** 包含了全数字（Fully-digital）、全模拟（Fully-analog）、部分/全连接混合架构（Partially/Fully-connected hybrid），以及基于 DMA 的三混合（Tri-hybrid）、基于流体天线（FAA）的三混合（Fluid tri-hybrid）等 。
    
2. **核心指标：** 针对不同的输入功率（Pin​），分别测量了这 9 种架构的两个终极指标：
    
    - **频谱效率 (Spectral efficiency, bps/Hz)**
        
        
    - **能量效率 / 能效 (Energy efficiency, bps/Hz/W)**
        
        
3. **不同场景：** 为了保证仿真的严谨性，他们测试了两种数据流复用场景：基准的双数据流（Ns​=2）以及更高负载的四数据流（Ns​=4） 。为了公平起见，所有参与对比的架构都保持了相同的辐射阵元总数和相同的输入功率 。
    ![[Pasted image 20260415170737.png]]
    仿真图 9(a) 和 9(c) 显示，三混合架构的频谱效率低于全数字和传统混合架构 。这是因为 DMA 预编码存在相位和幅度的洛伦兹耦合约束，且信号在波导中传播时会产生衰减损耗
    $$C = \sum_{k=1}^{K} \log_2 \left| \mathbf{I}_{N_r} + \frac{1}{\sigma^2} \mathbf{H}[k] \mathbf{F}_{em} \mathbf{F}_{hyb}[k] \mathbf{F}_{hyb}^*[k] \mathbf{F}_{em}^* \mathbf{H}^*[k] \right|$$
    
    仿真图 9(b) 和 9(d) 在较宽的输入功率范围内，所有三混合架构的能效都显著超越了传统的模拟、混合和全数字架构 。这证明了三混合架构的真正价值在于能够在极低的功耗下实现中等偏上的频谱效率，完美切中了 6G 系统既要巨大天线阵列，又要降低功耗的痛点 。[[Telecom：频谱效率和能量效率]]
    
$$EE = \frac{C}{P_{cons}}$$

