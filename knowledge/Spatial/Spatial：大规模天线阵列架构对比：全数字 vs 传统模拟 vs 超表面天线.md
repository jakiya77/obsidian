## 1. 全数字波束成形 (Full-Digital Array)

全数字架构被认为是理论上性能的“天花板”，具备最高的波束控制灵活性。

- **实现方式**：天线阵列上的**每个**天线阵元都直连一条专属的完整射频链路（包含高精度 ADC/DAC 和低噪声放大器 LNA）。
    
- **处理位置**：数字域。射频信号进入后立刻全部转换为数字信号，波束成形的权重相乘、信号相加，**100%在数字芯片里做数学运算**。
    

> [!warning] 致命弱点（瓶颈） 随着天线数量急剧增加，海量独立射频链路和高精度 ADC 会带来令人望而却步的硬件成本和极高的功耗。在强压制性干扰场景下，前端 ADC 极易因干扰过载而**饱和 (ADC Saturation)**，导致信号发生非线性失真（输出乱码），此时数字域算力再强也无法恢复有效信息 [^1] [^3]。

---

## 2. 传统模拟波束成形 (Analog Phased Array / 传统相控阵)

为大幅削减全数字架构的功耗和成本而诞生的折中方案。

- **实现方式**：采用多天线共享极少数（或单条）射频链路和 ADC 的结构。每个天线后端连接一个独立的物理器件——**射频移相器 (RF Phase Shifter)**。
    
- **处理位置**：模拟域。数字端算出权重后，下发指令控制移相器网络在纯射频模拟域逐个改变信号相位，最后通过庞大的**微波合成网络**（微带线或电缆）把所有信号汇聚成一路进 ADC。
    

> [!warning] 致命弱点（瓶颈） 波束控制自由度较低。更重要的是，传统的硬件移相器及微波合成网络依然存在**插入损耗大**、**体积笨重**且**昂贵**的固有的物理瓶颈 [^1]。

---

## 3. 动态超表面天线 / 可重构天线 (DMA / RA)

结合数字域计算与模拟域物理执行的新型软硬件协同架构。

- **实现方式**：采用**单射频链路 (1-RF-link)** 结构。利用新型可编程电磁材料（超表面阵元）直接替换传统的射频移相器网络。
    
- **处理位置**：数字与模拟跨界结合。
    
    1. **大脑计算（数字域）**：数字处理器计算出对抗干扰的最优波束成形权重（如 SVD 零空间计算）。
        
    2. **环境重构（控制传导）**：下发低频/直流指令改变阵元物理属性（如 PIN 二极管状态）。
        
    3. **物理合成（模拟域）**：高频射频信号在穿透阵元进入**物理波导管 (Waveguide)** 时，天然地发生物理干涉与叠加。
        

> [!success] 核心优势：天然的模拟滤波器 这种在波导内的物理汇聚是天线结构的“**天然副产品 (natural byproduct)**”，无需任何额外的专用合成硬件 [^1]。在不增加 ADC 数量的前提下，系统能在波导这种物理介质中直接形成抗干扰零陷，**将强干扰直接物理阻挡在低精度 ADC 之外**，从根本上解决了信号阻塞 (Signal Blocking) 问题，并大幅降低了系统功耗 [^2] [^3]。

---

## 📚 参考文献 (References)

[^1]: Wang, H., Shlezinger, N., Eldar, Y. C., Jin, S., Imani, M. F., Yoo, I., & Smith, D. R. (2021). Dynamic Metasurface Antennas for MIMO-OFDM Receivers With Bit-Limited ADCs. _IEEE Transactions on Communications_, 69(4), 2643-2659.
[^2]: Jiang, W., Huang, K., Yi, M., Chen, Y., & Jin, L. (2023). RIS-based Reconfigurable Antenna for Anti-jamming Communications with Bit-Limited ADCs. _2023 IEEE Globecom Workshops (GC Wkshps)_, 1433-1438. 
[^3]: Hao, Y., Wang, H.-M., Xiao, S., & Jin, L. (2026). Dynamic Metasurface Antenna Based Anti-Jamming With Bit-Limited ADC. _IEEE Transactions on Vehicular Technology_, 75(1), 1603-1607.