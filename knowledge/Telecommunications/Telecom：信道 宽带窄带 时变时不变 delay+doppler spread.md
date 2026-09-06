| **物理域** | **核心影响源**                      | **决定参数**                                   | **产生效应**                             | **衡量指标与物理意义**                                                |
| ------- | ------------------------------ | ------------------------------------------ | ------------------------------------ | ------------------------------------------------------------ |
| **时域**  | 多普勒扩展<br><br>(Doppler Spread)  | 相干时间<br><br>(Coherence Time, $T_c$)        | 时间选择性<br><br>(Time selectivity)      | **$T_{\rm frame} / T_c$**<br>评估信号传输期间信道是否发生变化。               |
| **频域**  | 时延扩展<br><br>(Delay Spread)     | 相干带宽<br><br>(Coherence Bandwidth, $B_c$)   | 频率选择性<br><br>(Frequency selectivity) | **$\beta = B\Delta\tau$**<br><br>评估信号带宽内是否感受到多径相位差。          |
| **空域**  | 阵列孔径时延<br><br>(Aperture Delay) | **相对带宽**<br><br>**(Fractional Bandwidth)** | 空间宽带效应 / 波束偏移<br><br>(Beam Squint)   | **$B/f_c$**<br><br>评估系统带宽相对于中心频率的占比。占比越大，阵列在不同频率下的空间响应差异越明显。 |
|         |                                |                                            |                                      |                                                              |
# 二、频域 Delay Spread ——> Coherence Bandwidth
## 1. 多径效应与延迟扩展 (Delay Spread)

多径传播环境的存在（如不同路径时延为 $\tau_1, \tau_2, \cdots$）会造成延迟扩展。以双径模型为例，其时延差定义为：

$$\Delta\tau = \vert{}\tau_2 - \tau_1\vert{}$$

在频域中，不同频率分量在第二条路径上观测到的相位差为：

$$e^{-j2\pi f\Delta\tau}$$

若频率发生 $\Delta f$ 的偏移，相应的相位变化量为：

$$\Delta\phi = 2\pi\Delta f\Delta\tau$$

## 2. 信号带宽对信道特性的影响

### 2.1 小带宽场景 (Narrowband Approximation)

当信号带宽 $B$ 满足：

$$B\Delta\tau \ll 1$$

此时在整个信号带宽内，相位变化受到严格限制：

$$2\pi B\Delta\tau \ll 2\pi$$

这意味着不同频率观测到的多径相位差变化极小。因此，整个带宽内的信道频率响应可近似为中心频率处的值：
$$h(f) \approx h(f_c)$$
该状态被称为 **Frequency-flat (频率平坦)** 或窄带近似。

### 2.2 大带宽场景 (Wideband Channel)

当信号带宽 $B$ 增大，使得：
$$B\Delta\tau \sim 1$$

甚至：
$$B\Delta\tau > 1$$

从带宽的一端到另一端，多径相位 $e^{-j2\pi f\tau}$ 已经发生显著变化，无法再将其近似为常数。

该状态被称为 **Frequency-selective (频率选择性)** 宽带信道。这也是代码实现中定义特定参数 $\beta = B\Delta\tau$ 的核心依据。

## 3. 参数 $\beta$ 的物理本质

参数 $\beta$ 称为归一化延迟扩展（Normalized Delay Spread），定义为：

$$\beta = B\Delta\tau = \frac{\Delta\tau}{1/B}$$

基于数字系统采样周期与带宽的关系 $T_s = \frac{1}{B}$，可推导出：

$$\beta = \frac{\Delta\tau}{T_s}$$

**物理意义：**

该参数直观反映了两条路径的时延差等效于多少个数字采样周期。

  

- **$\beta = 0.1 \implies \Delta\tau = 0.1 T_s$**
    
    时延差远小于一个采样周期，此时多径延迟效应微弱。
    
      
    
- **$\beta = 1 \implies \Delta\tau = T_s$**
    
    两径时延差刚好为一个完整的采样周期，产生显著的宽带多径效应（体现为信道冲激响应中特定抽头的凸起现象）。
    
      
    
- **$\beta = 2 \implies \Delta\tau = 2 T_s$**
    
    延迟扩展跨越两个采样周期，在数字基带处理（如 Q=2 FIR 均衡器）中会带来更高的计算与处理复杂度。
    
      
    

## 4. 与相干带宽 ($B_c$) 的定量关系

相干带宽 $B_c$ 与延迟扩展近似成反比关系：

  

$$B_c \propto \frac{1}{\tau_{\text{spread}}}$$

_(注：具体比例系数取决于所采用的相关性阈值标准)_

  

由此推导出评估信道特性的经典判断准则：

  

- **$B \ll B_c \implies$ Frequency-flat**
    
      
    
- **$B \gtrsim B_c \implies$ Frequency-selective**
    
      
    

基于 $B_c \sim \frac{1}{\Delta\tau}$ 的近似关系，可以构建如下比例：

  

$$\frac{B}{B_c} \sim B\Delta\tau = \beta$$

**核心结论：**

代码框架中定义的 $\beta = B\Delta\tau$，其数学本质是提供一个高度简便的无量纲参数，用于直接评估**带宽与相干带宽的量级比值**，从而量化判定系统的频率选择性程度。