

### 1. 整体系统关系

假设单天线接收模型为：

  

$$y = hs + n$$

其中：

  

- $s$：发射符号，尺寸为 $1\times1$
    
      
    
- $h$：复信道系数，尺寸为 $1\times1$
    
      
    
- $n$：噪声
    
      
    

若定义发射信号功率为 $E\vert{}s\vert{}^2 = P_s$，噪声功率为 $E\vert{}n\vert{}^2 = \sigma^2$，则接收信噪比为：

  

$$\mathrm{SNR} = \frac{P_s\vert{}h\vert{}^2}{\sigma^2}$$

在这里，**$\vert{}h\vert{}^2$ 就是最简单情况下的信道功率增益。**

  

### 2. 信道增益解析

#### 2.1 单天线场景

假设信道系数 $h = 0.5$，则信道功率增益为 $\vert{}h\vert{}^2 = 0.25$。

若发射功率 $P_s = 1$，则接收到的信号功率为 $P_r = \vert{}h\vert{}^2P_s = 0.25$。

转换为 dB 表示：

  

$$10\log_{10}(0.25) \approx -6.02 \text{ dB}$$

> **注：** 这完全是传播信道造成的损耗，与多天线相干合并没有任何关系。
> 
>   

#### 2.2 多天线场景

假设有 $N=4$ 根天线，信道向量为：

  

$$\mathbf{h} = \begin{bmatrix} h_1 \\ h_2 \\ h_3 \\ h_4 \end{bmatrix} \in \mathbb{C}^{4\times1}$$

若具体数值为 $\mathbf{h} = [0.5, 1.5, 0.5, 1.5]^T$，则：

  

$$\vert{}\mathbf{h}\vert{}^2 = 0.5^2 + 1.5^2 + 0.5^2 + 1.5^2 = 5$$

这里的 $5$ 表示**四个接收维度上的总信道能量**。它不能直接解释为“4根天线带来了5倍阵列增益”，因为每根天线上的信道幅度本身不同。

  

### 3. 阵列相干增益解析

考虑最理想的情况（4根天线，每根天线收到相同功率的期望信号）：

  

$$\mathbf{h} = \begin{bmatrix} 1 \\ 1 \\ 1 \\ 1 \end{bmatrix}$$

单根天线的 SNR 为：

  

$$\mathrm{SNR}_{1} = \frac{P_s}{\sigma^2}$$

现在使用最大比合并（MRC），令接收权重 $\mathbf{w} = \mathbf{h}$。

接收输出为：

  

$$\hat{s} = \mathbf{w}^H\mathbf{y} = \mathbf{h}^H(\mathbf{h}s + \mathbf{n}) = \mathbf{h}^H\mathbf{h}s + \mathbf{h}^H\mathbf{n}$$

因为 $\mathbf{h}^H\mathbf{h} = 4$，信号部分变为 $4s$，信号功率变为 $\vert{}4s\vert{}^2 = 16P_s$。

同时，噪声也被合并，输出噪声功率为 $\sigma^2\vert{}\mathbf{h}\vert{}^2 = 4\sigma^2$。

  

最终输出 SNR：

  

$$\mathrm{SNR}_{\rm out} = \frac{16P_s}{4\sigma^2} = 4\frac{P_s}{\sigma^2}$$

因此，真正的 SNR 增益为 $4$（即 $N$），对应 dB 值为：

  

$$10\log_{10}(4) \approx 6.02 \text{ dB}$$

> **结论：** 这才是我们通常所说的**4天线理想相干阵列增益**。
> 
>   

### 4. 仿真的归一化意义

在仿真中进行归一化的核心目的之一是：**把“信道本身强弱”与“阵列处理带来的增益”分开。**

  

为了研究 $N$ 根天线相比 1 根天线提供了多少相干合并增益（Coherent Combining Gain），最干净的方法是让每根天线上的平均信道功率一致（$\vert{}h_n\vert{}^2 = 1$），于是 $\vert{}\mathbf{h}\vert{}^2 = N$。

此时 MRC 的输出 SNR 为：

  

$$\mathrm{SNR}_{\rm out} = N\frac{P_s}{\sigma^2}$$

阵列增益自然为 $G_{\rm array} = N$，转换为 dB 为 $10\log_{10}(N)$。

  

|**天线数 (N)**|**理想阵列增益 (线性)**|**理想阵列增益 (dB)**|
|---|---|---|
|1|1|0 dB|
|2|2|3.01 dB|
|4|4|6.02 dB|
|8|8|9.03 dB|
|16|16|12.04 dB|

### 5. Steering Vector 的归一化陷阱

这是阵列仿真里最容易导致结果“差一截”的地方。

  

#### 5.1 不归一化 Steering Vector

通常定义导向矢量为：

  

$$\mathbf{a}(\theta) = \begin{bmatrix} 1 \\ e^{-j\phi} \\ e^{-j2\phi} \\ \dots \end{bmatrix}$$

此时每个元素模长为 1，故 $\vert{}\mathbf{a}(\theta)\vert{}^2 = N$。

若信道 $\mathbf{h} = \alpha\mathbf{a}(\theta)$，则 $\vert{}\mathbf{h}\vert{}^2 = \vert{}\alpha\vert{}^2 N$。

MRC 后的 SNR 可以直观地拆解为三部分：

  

$$\mathrm{SNR}_{\rm out} = \underbrace{\vert{}\alpha\vert{}^2}_{\text{Channel Gain}} \times \underbrace{N}_{\text{Array Gain}} \times \underbrace{\frac{P_s}{\sigma^2}}_{\text{Input SNR}}$$

#### 5.2 除以 $\sqrt{N}$ 的归一化

部分论文会将导向矢量定义为：

  

$$\tilde{\mathbf{a}}(\theta) = \frac{1}{\sqrt{N}}\mathbf{a}(\theta)$$

此时 $\vert{}\tilde{\mathbf{a}}\vert{}^2 = 1$。

若信道 $\mathbf{h} = \alpha\tilde{\mathbf{a}}$，则 $\vert{}\mathbf{h}\vert{}^2 = \vert{}\alpha\vert{}^2$。

在这个公式里，表面上看不到显式的 $N$。但这**不意味着天线阵列没有增益**，而是阵列规模对应的能量尺度已经被重新归一化了。

  

> **避坑指南：** 比较不同论文或 MATLAB 代码时，一定要先检查底层定义是 $\vert{}\mathbf{a}(\theta)\vert{}^2 = N$ 还是 $\vert{}\mathbf{a}(\theta)\vert{}^2 = 1$。
> 
>   

### 6. 信道增益与阵列增益的拆解实例

考虑最简单的单径模型：

  

$$\mathbf{h} = \alpha\mathbf{a}(\theta)$$

假设 $\vert{}\mathbf{a}(\theta)\vert{}^2 = N$，则 $\vert{}\mathbf{h}\vert{}^2 = \vert{}\alpha\vert{}^2 N$。天然分成了两部分：

  

1. **$\vert{}\alpha\vert{}^2$：** 路径传播造成的信道功率增益。
    
      
    
2. **$N$：** 阵列相干处理能够利用的阵列规模增益。
    
      
    

#### 数值推演

设 $P_s = 1, \sigma^2 = 0.01$，若无信道衰减，基础 SNR 为 $100$ ($20 \text{ dB}$)。

设信道系数 $\alpha = 0.5$（$\vert{}\alpha\vert{}^2 = 0.25$，损耗为 $-6.02 \text{ dB}$）。

  

- **单天线接收：**
    
      
    
    $$\mathrm{SNR}_{1} = \vert{}\alpha\vert{}^2\frac{P_s}{\sigma^2} = 0.25 \times 100 = 25 \quad (13.98 \text{ dB})$$
    
- **4 天线 MRC 接收：**
    
    若 $\mathbf{h} = 0.5 \times [1, 1, 1, 1]^T$，则 $\vert{}\mathbf{h}\vert{}^2 = 4 \times 0.25 = 1$。
    
      
    
    $$\mathrm{SNR}_{4} = 1 \times 100 = 100 \quad (20 \text{ dB})$$
    

**总结效果：**

单天线 SNR ($13.98 \text{ dB}$) 到 4 天线 SNR ($20 \text{ dB}$) 的提升为 **$6.02 \text{ dB}$**。

这 $6.02 \text{ dB}$ 就是纯粹的**4天线阵列增益**，而信道本身的 **$-6.02 \text{ dB}$** 衰减并没有消失，只是被阵列增益抵消了。

  

### 7. 多径环境的复杂性与仿真建议

#### 7.1 多径容易引发的混淆

多径信道模型为：

  

$$\mathbf{h} = \sum_{l=1}^{L} \alpha_l \mathbf{a}(\theta_l) e^{-j2\pi f\tau_l}$$

此时 $\vert{}\mathbf{h}\vert{}^2$ 中不仅包含各路径增益 $\vert{}\alpha_l\vert{}^2$，还包含不同路径间的**相干叠加或相消**。

因此，$\vert{}\mathbf{h}\vert{}^2$ 可能大于 $N$，也可能小于 $N$，且会随频率发生变化。在多径场景下，绝对不能简单看到 $\vert{}\mathbf{h}\vert{}^2 = 5$ 就得出“提供了 5 倍阵列增益”的结论。

  

#### 7.2 对 Multibeam + Taps 仿真的指导

在当前的仿真中，对于频率 $k$：

  

$$\mathbf{h}_k = \alpha_1 \mathbf{a}_1 e^{-j2\pi f_k\tau_1} + \alpha_2 \mathbf{a}_2 e^{-j2\pi f_k\tau_2}$$

不同子载波 $k$ 的 $\vert{}\mathbf{h}_k\vert{}^2$ 本身就不同。

计算 $\mathrm{SNR}_{\rm FD, k} = \frac{P_s\vert{}\mathbf{h}_k\vert{}^2}{\sigma^2}$ 时，其中已经揉合了：

  

1. **频率选择性信道本身的 Gain**
    
      
    
2. **Full-digital MRC 对能量的最佳利用**
    
      
    

> **分析建议：** 分析 multibeam gain 时，必须按以下阶段进行拆解，而不能仅从最终的 SNR 数字反推“阵列增益”：
> 
>   
> 
> 1. 传播信道本身有多少能量？
>     
>       
>     
> 2. 模拟 Beamforming (Analog) 保留了多少能量？
>     
>       
>     
> 3. 数字 Taps (Digital) 又恢复/利用了多少能量？
>     
>       
>     
> 4. 最终输出的 SNR。
>     
>       
>     

### 8. 最终记忆版

如果采用 $\vert{}\mathbf{a}(\theta)\vert{}^2 = N$ 的导向矢量定义，在最干净的单径模型 $\mathbf{h} = \alpha\mathbf{a}(\theta)$ 中，增益的分离方式最为清晰：

$$\vert{}\mathbf{h}\vert{}^2 = \underbrace{\vert{}\alpha\vert{}^2}_{\text{信道/路径增益}} \times \underbrace{N}_{\text{阵列规模带来的可利用相干增益}}$$