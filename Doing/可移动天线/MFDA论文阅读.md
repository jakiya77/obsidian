## 系统模型

$$
\begin{aligned}
\mathbf{h}_{ab}(t,b)
&= Lfs(b)
\begin{bmatrix}
h_1(t,b) \\
\vdots \\
h_M(t,b)
\end{bmatrix}, \\
\mathbf{h}_{ae_k}(t,e_k)
&= Lfs(e_k)
\begin{bmatrix}
h_1(t,e_k) \\
\vdots \\
h_M(t,e_k)
\end{bmatrix}.
\end{aligned}

$$

$$h_{m}(t,q)=e^{-j2\pi f_{m}(t-\frac{||q-t_{m}||_{2}}{c})}$$
$$q = \{b, e_k\}$$
这里的q是**通用的位置变量（Placeholder）**，代表**接收端的目标用户**。
- **$b$ (Bob):** 合法用户的位置坐标，即 $b = [x_b, y_b, z_b]^T$ 。
    
- **$e_k$ (Eve):** 第 $k$ 个窃听者的位置坐标，即 $e_k = [x_{e_k}, y_{e_k}, z_{e_k}]^T$ 
$$||q - t_m||_2$$
计算的是**发射天线到接收者之间的直线距离**

$$
\begin{aligned}
y_b(t) &= \mathbf{h}_{ab}^\dagger \mathbf{w}(t)\, s + n_b(t), \\
y_{e_k}(t) &= \mathbf{h}_{ae_k}^\dagger \mathbf{w}(t)\, s + n_{e_k}(t).
\end{aligned}

$$
$$\text{相角} = \underbrace{2\pi f_m t}_{\text{时间项}} - \underbrace{2\pi f_m \frac{\|q-t_m\|}{c}}_{\text{空间滞后项}}$$
- **第一项 $2\pi f_m t$（时间项）：**
    
    - 普通的波束成形（Phased Array）所有天线频率 $f_m$ 都是一样的 $f_c$。
        
    - 但在 **FDA** 里，每个天线 $m$ 的频率 $f_m$ 是**不一样的**。
        
    - 信道 $h$ 是**随时间 $t$ 自动变化**的。即使天线不动，波束也会在空间中自动扫描或变化。_"channels are dependent on time t"_ 。
        
- **第二项 $2\pi f_m \frac{\text{距离}}{c}$（空间项）：**
    
    - $\frac{\|q-t_m\|}{c}$ 就是信号飞过这段距离所需的时间（延迟）。
        
    - 乘以 $2\pi f_m$ 就变成了相位差。
        
    - 这里包含了 **$t_m$ (天线位置)**。
        
    - 改变天线位置 $t_m$，就是在直接改变这一项相位。通过精细调节 $t_m$，让这些相位在 Bob 处完全对齐（相加），在 Eve 处完全抵消（相减）。

### 1. 针对系统容量的定义和推导Cest
#### 1. 瞬时速率 $R_b(t)$ 和 $R_{e_k}(t)$ 的推导 (Eq. 6 & 7)

这部分基于经典的**信噪比 (SNR)** 计算。

第一步：写出接收信号模型

根据论文中的公式 (4) 和 (5) 1，Bob 和 Eve 接收到的信号 $y$ 分别为：

- **信号部分 (Signal):** $h^\dagger w(t) s$
    
    - $h^\dagger$: 信道向量的共轭转置（代表经过信道传输）。
        
    - $w(t)$: 发射机的波束成形向量（Alice 的预编码）。
        
    - $s$: 发射的有用符号。
        
- **噪声部分 (Noise):** $n$
    
    - $n \sim \mathcal{CN}(0, \sigma^2)$: 加性高斯白噪声。
        

第二步：计算接收端的信噪比 (SNR)

信噪比的定义是 信号功率 除以 噪声功率。

- 信号功率 (Signal Power):
    
    假设发射信号 $s$ 的功率归一化为 1（即 $E[|s|^2]=1$）。
    
    接收到的有用信号功率为模的平方：
    
    $$P_{signal} = |h^\dagger w(t)|^2$$
    
- 噪声功率 (Noise Power):
    
    题目直接给出噪声功率为 $\sigma^2$ 2。
    

所以，Bob 的信噪比 $SNR_b$ 为：

$$SNR_b = \frac{|h_{ab}^\dagger w(t)|^2}{\sigma_b^2} = \sigma_b^{-2} |h_{ab}^\dagger w(t)|^2$$

第三步：代入香农公式

根据香农定理，高斯信道的频谱效率（单位 Hz 的速率）公式为：

$$R = \log_2(1 + SNR)$$

将上面的 SNR 代入，就直接得到了公式 (6) 3：

$$R_b(t) = \log_2\left( 1 + \sigma_b^{-2} |h_{ab}^\dagger w(t)|^2 \right)$$

同理可得 Eve 的公式 (7)。

---

#### 2. 安全容量 $C_{sec}$ 的推导 (Eq. 8)

这里涉及到物理层安全的核心定义。你问到的 _"We assume that Eves are not cooperative and consider the worst case"_ 是理解这个公式的关键。

物理层安全的定义：

安全容量通常定义为：**合法信道的速率** 减去 **窃听信道的速率**。

$$C_{sec} = [R_b - R_e]^+$$

(如果不安全，即 $R_e > R_b$，则安全容量为 0)

为什么是 Max (最坏情况)?

文中假设有 $K$ 个窃听者 (Eves)，且她们不合作 (Non-cooperative) 。

- **不合作**意味着：每个窃听者只能靠自己接收到的信号来解密，她们之间不会把接收到的信号拿来合并处理。
    
- **最坏情况 (Worst Case)**意味着：只要有一个窃听者能解密，通信就不安全了。所以对 Alice 来说，威胁最大的是那个**信道质量最好、速率最高**的窃听者。
    

因此，我们要减去的是所有窃听者中最强的那一个的速率：

$$\max_{E} \{ R_{e_k}(t) \}$$

最终得出了论文中的公式 (8) 5：

$$C_{sec}(t) = R_b(t) - \max_{k \in \{1...K\}} \{ R_{e_k}(t) \}$$



### 2. 优化目标的拆解
$$
\begin{aligned}
\max_{\mathbf{w}(t),\,\mathbf{x},\,\mathbf{f}} \quad
& C_{\mathrm{sec}}(t) \quad (1) \\[2mm]
\text{s.t.} \quad
& \|\mathbf{w}(t)\|_2^2 \le P, \quad (2) \\
& \mathbf{t}_m \in \mathcal{C}_{\mathbf{t}_m}, \ \forall m, \quad (3) \\
& \|\mathbf{t}_i - \mathbf{t}_j\|_2 \ge D,
\ \forall i\neq j, \quad (4) \\
& f_C \le f_m \le f_C + \Delta F,
\ \forall m. \quad (5)
\end{aligned}
$$

(4)(5)的目的是在可移动天线和fda的维度上，防止耦合和满足We assume 0 ≤ ∆fm ≤ ∆F and ∆F ≪

fC .的要求fm = fC + ∆fm, m = 1, ..., M。[[OPT：convex function 凸优化和非凸问题的处理]]
>最后拆解的结果就相当于是先对位置x和频率f进行一个BUSM 来求解非凸问题，然后得到的一组优化后的（x_opt,f_opt）用于代入求解**凸优化问题**得到最优解w_opt「这里的凸问题也是使用Fenchel 共轭变换处理后的非」
