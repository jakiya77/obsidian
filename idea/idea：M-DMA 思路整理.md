## 1. 物理架构
![[png：Dynamic Metasurface Antennas for  6G Extreme Massive MIMO Communications.png]]横向一排波导上嵌有若干个element，每一个波导连接着一个RF chain，设计每一个波导由电机驱动可以左右移动，即 $x_{n,m} = u_n + d_m$。
![[png：Modeling and Performance Analysis for Movable Antenna Enabled Wireless Communications.png]]
![[png：Movable-Element RIS-Aided Wireless Communications ：An Element-Wise Position Optimization Approach.png|452]]
上图分别是传统的可移动天线架构和可移动RIS架构。可以观察到早期全数字 MA 架构需要再每一个element上连接RF chain，不但面临高额的RF chain开销，还会存在每个天线独立 2D 运动所需的复杂机械解耦。而新型的ME-RIS架构不需要逐元素连接RF chain，只连接控制线，只能改变信号的传播路径，无法在本地进行任何信号剥离或自适应滤波。于是引入M-DMA架构，波导管作为信号传输与合成的物理媒介，实现对信号的波域处理和数字域处理。
 
## 2. 移动性的引入和过载零陷分析
$$K \le \frac{N_{\text{RF}}(N_M + 3)}{2} - 1$$

## 3. 优化问题

### Step 1 优化问题第一步是在波域对干扰进行对齐处理

---