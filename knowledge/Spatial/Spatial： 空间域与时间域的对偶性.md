
> [!abstract] 核心思想
> 
> 在阵列信号处理中，**角度即频率**（Angle as Frequency）。天线阵列在空间上的等间距排布，本质上是对空间电磁场的抽样。

## 1. 核心映射对比表

| **特性**   | **时间域 (Time Domain)** | **角度域 (Spatial Domain)**                |
| -------- | --------------------- | --------------------------------------- |
| **独立变量** | 时间 $t$                | 空间位置 $x = nd$                           |
| **对偶变量** | 频率 $f$ (Hz)           | 空间频率 $\nu = \frac{\sin\theta}{\lambda}$ |
| **采样间隔** | 采样周期 $T_s$            | 阵元间距 $d$                                |
| **基本变换** | 傅里叶变换 (DFT)           | 阵列因子 (Array Factor)                     |
| **采样定理** | $f_s \ge 2f_{max}$    | $d \le \lambda/2$                       |
| **混叠现象** | 频谱混叠 (Aliasing)       | 栅瓣 (Grating Lobes)                      |
| **分辨率**  | 采样时间窗 $T_{total}$     | 阵列物理孔径 $L = Nd$                         |

---

## 2. 数学推导：从 DFT 到阵列因子

### 2.1 离散信号的频谱 (DFT)

对于一个长度为 $N$ 的离散序列 $x[n]$，其频谱函数（连续形式）为：

$$X(f) = \sum_{n=0}^{N-1} x[n] \cdot e^{-j 2\pi f \cdot n T_s}$$

这里 $n T_s$ 代表采样的时刻。

### 2.2 阵列因子的形成 (AF)

考虑一个等间距线阵 (ULA)，信号以角度 $\theta$ 入射。第 $n$ 个阵元相对于参考点的相位延迟为：

$$\psi = \frac{2\pi d}{\lambda} \sin \theta$$

阵列因子为：

$$AF(\theta) = \sum_{n=0}^{N-1} w_n \cdot e^{j n \psi} = \sum_{n=0}^{N-1} w_n \cdot e^{j 2\pi n d \frac{\sin \theta}{\lambda}}$$

### 2.3 变量对偶映射

对比上述两个公式，我们可以建立如下对应关系：

1. **输入权重**：$x[n] \longleftrightarrow w_n^*$
    
2. **归一化频率**：$f T_s \longleftrightarrow \frac{d \sin \theta}{\lambda}$
    

> [!important] 结论
> 
> 阵列方向图本质上就是对权重序列 $\{w_n\}$ 进行的**离散时间傅里叶变换 (DTFT)**，其中“频率”变量由入射角 $\theta$ 的正弦值映射。

---

## 3. 物理现象的深度对偶

### 3.1 为什么 $d \le \lambda/2$ 是空间采样定理？

栅瓣产生的本质是相邻阵元的总相位差达到了 $\pm 2\pi$。根据阵列相位公式，栅瓣出现的空间正弦位置为：

$$\sin\theta_{GL} = \sin\theta_0 \pm \frac{\lambda}{d}$$

物理空间的“可见区”严格限制在 $[-1, 1]$ 内。为了确保在**最极端的全空域扫描状态下**（即主瓣指向 $\theta_0 = 90^\circ$，$\sin\theta_0 = 1$），栅瓣依然无法落入可见区，我们必须要求：

$$\sin\theta_{GL} = 1 - \frac{\lambda}{d} < -1$$

解得：

$$d < \frac{\lambda}{2}$$
[[Spatial：Linear Array Beam Characteristics and Array Factor + Grating Lobes and Beam Squint]]
### 3.2 为什么波束宽度反比于孔径？

- **时域**：信号持续时间越长（数据量越多），频谱主瓣越窄，频率分辨率越高。
    
- **空域**：阵列物理长度（孔径 $Nd$）越长，波束主瓣越窄，空间分辨率（区分相邻信号的能力）越高。
    
