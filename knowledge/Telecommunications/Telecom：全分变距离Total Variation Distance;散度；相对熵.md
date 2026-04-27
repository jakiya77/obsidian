

### 1. 场景设定：

假设 Willie 在监听信道，他要在 $n$ 个时刻采样。每一个时刻 $i$，他观察到的样本是 $x_i$。

- 情况 $H_0$（Alice 没发，只有噪声）：
    
    $x_i \sim \mathcal{N}(0, \sigma^2)$
    
    这纯粹是背景噪声，均值为0，方差为 $\sigma^2$。记其分布为 $P_0$。
    
- 情况 $H_1$（Alice 发了，信号+噪声）：
    
    在隐蔽通信中，Alice 发送的通常也是类噪声信号（比如扩频信号）。
    
    $x_i \sim \mathcal{N}(0, \sigma^2 + P)$
    
    这里 $P$ 是信号功率。因为信号和噪声叠加，总功率变大了，方差变成了 $\sigma^2 + P$。记其分布为 $P_1$
    

**Willie 的任务：** 一堆数据，区分它是来自方差 $\sigma^2$ 的分布，还是方差 $\sigma^2 + P$ 的分布。

---

### 2. 全变分距离 (Total Variation Distance, $V_T$)

物理含义：

全变分距离直观地衡量了两个概率密度函数（PDF）重叠部分的缺失程度。

想象画出两个高斯钟形曲线：

- $P_0$ 瘦高一点点。
    
- $P_1$ 矮胖一点点（因为方差大）。
    
- $V_T$ 就是这两条曲线之间**没重叠的那部分面积的一半**。
    

公式与理解：

$$V_T(P_0, P_1) = \frac{1}{2} \int | p_0(x) - p_1(x) | dx$$

或者理解为：Willie 能把这两者区分开的最大可能性。

- 如果 $V_T = 0$：两曲线完全重合，Willie 根本分不出来（瞎猜）。
    
- 如果 $V_T = 1$：两曲线完全没重叠（比如一个在正无穷，一个在负无穷），Willie 100% 能抓住 Alice。
    

计算难度：

直接积分那个绝对值 $|p_0 - p_1|$ 非常难算，很难得到解析解。所以我们才需要 KL 散度。

---

### 3. 相对熵 / KL 散度 (Kullback-Leibler Divergence, $D$)

物理含义：

它不是严格的“距离”（因为它不对称），但在信息论里，它衡量的是：当你明明面对的是分布 $P_1$（有信号），却错误的假设它是 $P_0$（没信号）时，你会产生多少信息损失。

计算公式（通用）：

$$D(P_1 || P_0) = \mathbb{E}_{x \sim P_1} \left[ \log \frac{p_1(x)}{p_0(x)} \right] = \int p_1(x) \ln \frac{p_1(x)}{p_0(x)} dx$$
> 在通信和信息论中，logP(x)代表的是整个事件发生包含的信息量（比特数），这个log的比值也被叫做对数似然比（Log- likelihood Ratio，LLR），KL散度的原理就是求信息量差值的加权平均值。
### 实战计算（高斯分布的方差变化）：

直接代入高斯公式。

设 $P_0 = \mathcal{N}(0, \sigma^2)$， $P_1 = \mathcal{N}(0, \sigma^2 + P)$。

两个零均值高斯分布的 KL 散度公式是：

$$D(P_1 || P_0) = \frac{1}{2} \left[ \ln\frac{\sigma_0^2}{\sigma_1^2} + \frac{\sigma_1^2}{\sigma_0^2} - 1 \right]$$

代入我们的参数：

$$D(P_1 || P_0) = \frac{1}{2} \left[ \ln\frac{\sigma^2}{\sigma^2 + P} + \frac{\sigma^2 + P}{\sigma^2} - 1 \right]$$

让我们做个近似！因为是隐蔽通信，信号功率 $P$ 远小于噪声 $\sigma^2$（即 $\text{SNR} = \frac{P}{\sigma^2} \ll 1$）。

利用 $\ln(1+x) \approx x - \frac{x^2}{2}$ (泰勒展开)，我们可以推导出一个非常惊艳的简单结果：

$$D(P_1 || P_0) \approx \frac{1}{4} \left( \frac{P}{\sigma^2} \right)^2 = \frac{1}{4} \text{SNR}^2$$

结论：
KL 散度与 信噪比 (SNR) 的平方成正比。

---

### 4. 为什么要让SNR<<sqrt(n)

关键在于**随机过程的时间性**。Willie 不是只看 1 个点，而是观察 **$n$ 个点** (observations)。

对于独立同分布 (i.i.d.) 的样本，总的 KL 散度是可加的：

$$D(P_1^n || P_0^n) = n \times D(P_1 || P_0)$$

代入刚才的近似公式：

$$D_{total} \approx n \times \frac{1}{4} \text{SNR}^2$$

Pinsker 不等式连接一切：

Willie 的检错概率下界由 KL 控制：

$$\text{Willie的错误率} \ge 1 - \sqrt{\frac{1}{2} D_{total}}$$

$$\text{Willie的错误率} \ge 1 - \sqrt{\frac{1}{2} \cdot \frac{n}{4} \text{SNR}^2} \approx 1 - \text{SNR} \cdot \sqrt{n}$$


为了让 Willie 犯错（错误率 $\approx 1$），后面那一项 $\text{SNR} \cdot \sqrt{n}$ 必须接近 0。

这意味着：

$$\text{SNR} \ll \frac{1}{\sqrt{n}}$$

**这就是为什么 KL 必须非常小，而且信号功率必须随着采样数 $n$ 的增加而不断减小（平方根定律）。** 如果你保持常数功率发信号，只要 Willie 听得够久（$n$ 够大），总 KL 散度就会爆炸.

### 总结

1. **全变分距离 ($V_T$)**：最直观的“区分度”，也就是 Willie 抓你的成功率。但这东西难算。
    
2. **KL 散度 ($D$)**：好算的数学工具。在高斯白噪声下，它约等于 $\text{SNR}^2$。
    
3. **为什么必须极小**：因为 Willie 收集的数据量 $n$ 很大。单个样本的微小差异（KL），乘以 $n$ 后会变得巨大。为了抵消 $n$ 的增长，你的单次信号强度（KL）必须非常非常小，小到能被 $n$ “稀释”掉。