

### TL;DR
当前代码的核心结论是：

  

$$\boxed{\mathbf w_{\rm Path}[k] = \frac{1}{\sqrt N}\mathbf h[k] \propto \mathbf w_{\rm MRC}[k]}$$

所以在当前

$$\boxed{\text{perfect CSI} + \text{OFDM/CP 足够} + \text{full-digital receiver} + \text{single-user white noise}}$$

条件下，matched path beam + delay alignment + path combining 与 per-subcarrier MRC 本质上是同一个接收器的两种写法。

因此代码中的 Fixed+MRC = Fixed+Path-DA，以及 MA+MRC = MA+Path-DA，不是参数碰巧，而是数学上必然如此。代码本身也明确把这个实验设计为 perfect-CSI sanity check。

  

你现在只需要掌握：

  

Plaintext

```
path 参数
    ↓
构造宽带信道 h[k]
    ↓
MRC 为什么 w ∝ h
    ↓
Path-DA 最终等效成什么 w
    ↓
证明 w_Path ∝ h
```

到这里就够。ZF 的完整伪逆推导、DAM 历史、OFDM 连续时间严格证明，现在都不用钻。

  

### 问题结构树

- **1. 当前代码到底在比较什么**
    
      
    - 1.1 天线位置：Fixed vs MA
        
          
        
    - 1.2 接收方法：MRC vs Path-DA
        
          
        
    - 1.3 所有方法共同的系统模型
        
          
        
- **2. 数学模型是如何一步步变成 MATLAB 的**
    
      
    - 2.1 一条 path 有哪些物理参数
        
          
        
    - 2.2 AoA → 空间响应 $a_P(\theta)$
        
          
        
    - 2.3 Delay → 子载波相位
        
          
        
    - 2.4 多径叠加 → $h[k] = A d[k]$
        
          
        
    - 2.5 接收模型 → $y[k] = h[k]x[k] + n[k]$
        
          
        
- **3. MRC 与 Path-DA 为什么理论上相同**
    
      
    - 3.1 MRC 的数学原理
        
          
        
    - 3.2 Path beam 在做什么
        
          
        
    - 3.3 Delay compensation 在做什么
        
          
        
    - 3.4 Path-DA 的等效接收权重
        
          
        
    - 3.5 核心等价证明
        
          
        
    - 3.6 为什么 Rate 和 MA 最优位置都相同
        
          
        
- **4. 为什么“理想 OFDM”条件这么关键**
    
      
    - 4.1 OFDM 已经如何处理 multipath
        
          
        
    - 4.2 Perfect CSI 为什么让 path 信息变得冗余
        
          
        
    - 4.3 Full digital 为什么能够直接实现最优 $w[k]$
        
          
        
    - 4.4 哪些条件改变后，Path-wise processing 才可能真正不同
        
          
        
- **5. 你现在需要掌握到什么程度**
    
      
    - 5.1 必须完全掌握
        
          
        
    - 5.2 理解概念即可
        
          
        
    - 5.3 现在不要深入
        
          
        
    - 5.4 当前学习的停止标准
        
          
        
- **6. 下一步**
    
      
    - 6.1 对照现有 MATLAB
        
          
        
    - 6.2 弄懂后再回研究问题
        
          
        

### 1. 当前代码到底在比较什么

#### 1.1 第一个自由度：天线位置

代码中有两种阵列。

  

**1.1.1 Fixed ULA**

例如：

$$\mathbf P_{\rm fixed} = [0,\;0.5\lambda,\;\lambda,\;1.5\lambda]^T.$$

位置固定。

  

**1.1.2 MA**

位置：

  

$$\mathbf P = [p_1,\ldots,p_N]^T$$

可以在一个 aperture 内搜索。

因此 MA 回答的是：

  

$$\boxed{\text{天线应该放在哪里？}}$$

代码会枚举 feasible MA layouts，然后分别计算每个位置对应的 rate，再寻找最大值。

  

#### 1.2 第二个自由度：接收方法

给定位置以后，又有两种接收方式：

  

Plaintext

```
接收到 y[k]
│
├─ MRC
│   └─ 直接根据总信道 h[k] 设计 w[k]
│
└─ Path-DA
    ├─ 对每条 path 建一个 beam
    ├─ 补偿该 path 的 delay
    └─ 再进行 path combining
```

所以当前实验本质上是在研究：

  

$$\boxed{\text{Position Design} \times \text{Receiver Design}}$$

而不是四套毫无联系的方法。

  

#### 1.3 代码中的主要方案

当前脚本实际上包括：

  

Plaintext

```
位置固定
├─ Fixed + MRC
└─ Fixed + matched path + DA

位置可移动
├─ MA(MRC-opt) + MRC
├─ MA(path-opt) + matched path + DA
└─ MA(MRC-opt) + ZF-path optimal
```

其中 ZF-path optimal 主要是一个额外 sanity check：代码本身就预期它在理想条件下几乎恢复 full-digital MRC。

  

### 2. 数学模型是如何一步步变成 MATLAB 的

这部分是你现在最值得认真掌握的。

  

#### 2.1 一条 path 有哪些参数

第 $l$ 条 path 写成：

  

$$(\alpha_l,\theta_l,\tau_l).$$

可以理解为：

  

Plaintext

```
Path l
├─ α_l：复增益
│   ├─ 路径强度
│   └─ 路径自身初始相位
│
├─ θ_l：AoA
│   └─ 决定空间相位
│
└─ τ_l：传播 delay
    └─ 决定频率相关相位
```

当前程序两径设置就是：

  

$$L=2.$$

#### 2.2 AoA 如何形成空间响应

**2.2.1 从物理上看**

第 $l$ 条 path 从角度 $\theta_l$ 入射。

第 $n$ 根天线在：

  

$$p_n.$$

由于不同天线传播距离不同，所以产生空间相位：

  

$$e^{-j\frac{2\pi}{\lambda}p_n\sin\theta_l}.$$

因此：

  

$$\boxed{\mathbf a_{\mathbf P}(\theta_l) = \begin{bmatrix} e^{-j\frac{2\pi}{\lambda}p_1\sin\theta_l} \\ \vdots \\ e^{-j\frac{2\pi}{\lambda}p_N\sin\theta_l} \end{bmatrix}}$$

**2.2.2 MA 在哪里发挥作用**

注意公式里的：

  

$$p_1,p_2,\ldots,p_N.$$

这就是 MA 控制的变量。

所以：

  

$$\boxed{\mathbf P \rightarrow \mathbf a_{\mathbf P}(\theta_l)}$$

移动天线，本质上就是改变不同 path 的空间相位结构。

  

**2.2.3 MATLAB 对应**

代码：

  

Matlab

```
A(:,ell) = exp(-1j*2*pi/lambda * p * s);
```

其中：

  

Matlab

```
s = sin(theta_deg(ell)*pi/180);
```

正对应：

  

$$\mathbf a_{\mathbf P}(\theta_l).$$

如果有 $L$ 条 path，就把它们排成：

  

$$\boxed{\mathbf A = [\mathbf a_1,\mathbf a_2,\ldots,\mathbf a_L]}$$

尺寸：

  

$$\mathbf A\in\mathbb C^{N\times L}.$$

#### 2.3 Delay 在 OFDM 里面变成什么

第 $l$ 条 path 有：

  

$$\tau_l.$$

到了第 $k$ 个子载波，它产生：

  

$$\boxed{e^{-j2\pi f_k\tau_l}}$$

这个相位。

所以第 $l$ 条 path 对第 $k$ 个子载波信道的贡献为：

  

$$\boxed{\alpha_l e^{-j2\pi f_k\tau_l} \mathbf a_l.}$$

这里三部分非常清楚：

  

$$\underbrace{\alpha_l}_{\text{path gain}} \; \underbrace{e^{-j2\pi f_k\tau_l}}_{\text{delay}} \; \underbrace{\mathbf a_l}_{\text{space}}.$$

#### 2.4 所有 path 相加形成总信道

两径时：

  

$$\boxed{\mathbf h[k] = \alpha_1e^{-j2\pi f_k\tau_1}\mathbf a_1 + \alpha_2e^{-j2\pi f_k\tau_2}\mathbf a_2.}$$

为了方便，把：

  

$$\mathbf d[k] = \begin{bmatrix} \alpha_1e^{-j2\pi f_k\tau_1} \\ \alpha_2e^{-j2\pi f_k\tau_2} \end{bmatrix}$$

定义出来。

于是：

  

$$\boxed{\mathbf h[k] = \mathbf A\mathbf d[k].}$$

尺寸一定要会看：

  

$$\underbrace{\mathbf A}_{N\times L} \underbrace{\mathbf d[k]}_{L\times 1} = \underbrace{\mathbf h[k]}_{N\times 1}.$$

**2.4.1 MATLAB 对应**

  

Matlab

```
delay_phase = exp(-1j*2*pi*f_bb(k)*tau);
d_k = alpha .* delay_phase;
H(:,k) = A * d_k;
```

你以后看这三行，脑子里要能立即翻译成：

  

Plaintext

```
delay_phase
    ↓
每条 path 在第 k 个子载波上的 delay phase

alpha .* delay_phase
    ↓
每条 path 的完整频域系数 d_l[k]

A * d_k
    ↓
所有 path 在空间中相加

H(:,k)
    ↓
第 k 个子载波的总信道 h[k]
```

这就是“公式 → 代码”最核心的一步。

  

#### 2.5 OFDM 接收信号最终变成什么

CP 足够并完成 FFT 后，每个子载波可以独立写成：

  

$$\boxed{\mathbf y[k] = \mathbf h[k]x[k] + \mathbf n[k].}$$

其中：

  

$$\mathbf y[k] \in\mathbb C^{N\times1}.$$

例如 $(N=4)$：

  

Plaintext

```
天线 1 → y1[k]
天线 2 → y2[k]
天线 3 → y3[k]
天线 4 → y4[k]

组成

y[k] = [y1[k], y2[k], y3[k], y4[k]]ᵀ
```

从这一刻开始，接收端的问题就变成：

  

$$\boxed{\text{如何设计 }\mathbf w[k] \text{ 来组合 }\mathbf y[k]？}$$

### 3. MRC 与 Path-DA 为什么理论上相同

这一部分是整份笔记的核心。

  

#### 3.1 MRC 为什么是 $\mathbf w\propto\mathbf h$

**3.1.1 线性接收**

设计：

  

$$\mathbf w[k]\in\mathbb C^{N\times1},$$

输出：

  

$$z[k] = \mathbf w^H[k]\mathbf y[k].$$

代入：

  

$$\mathbf y = \mathbf h x+\mathbf n,$$

得到：

  

$$z = \underbrace{\mathbf w^H\mathbf h x}_{\text{desired signal}} + \underbrace{\mathbf w^H\mathbf n}_{\text{noise}}.$$

**3.1.2 输出 SNR**

假设：

  

$$\mathbf n \sim \mathcal{CN}(0,\sigma^2\mathbf I).$$

输出 SNR：

  

$$\boxed{\gamma = \frac{ P_s\vert{}\mathbf w^H\mathbf h\vert{}^2 }{ \sigma^2\vert{}\mathbf w\vert{}^2 }.}$$

**3.1.3 最大化这个 SNR**

Cauchy-Schwarz：

  

$$\vert{}\mathbf w^H\mathbf h\vert{}^2 \leq \vert{}\mathbf w\vert{}^2\vert{}\mathbf h\vert{}^2.$$

因此：

  

$$\gamma \leq \frac{ P_s\vert{}\mathbf h\vert{}^2 }{ \sigma^2 }.$$

什么时候取得等号？

当：

  

$$\boxed{\mathbf w=c\mathbf h.}$$

所以：

  

$$\boxed{\mathbf w_{\rm MRC}[k] \propto \mathbf h[k].}$$

这就是 Maximum Ratio Combining。

  

**3.1.4 为什么只写 $\propto$**

如果：

  

$$\mathbf w=\mathbf h,$$

或者：

  

$$\mathbf w=10\mathbf h,$$

输出 SNR 都一样。

因为整体放大同时放大 signal 和 noise。

因此 MRC 真正关心的是：

  

$$\boxed{\mathbf w\text{ 的方向}}$$

而不是整体尺度。

所以代码中甚至不用显式计算 $w$：

  

Matlab

```
snr_k = Ps * sum(abs(H).^2,1) / sigma2;
```

因为最优值直接就是：

  

$$\boxed{\gamma_{\rm MRC}[k] = \frac{P_s\vert{}\mathbf h[k]\vert{}^2}{\sigma^2}.}$$

#### 3.2 Path beam 在做什么

你的原始思想是：

  

Plaintext

```
                    y[k]
                     │
          ┌──────────┴──────────┐
          ↓                     ↓
       beam 1                 beam 2
        θ1                      θ2
          ↓                     ↓
       branch 1              branch 2
```

第 $l$ 条 path 使用 matched beam：

  

$$\mathbf v_l = \frac{\mathbf a_l}{\vert{}\mathbf a_l\vert{}}.$$

steering vector 每一个元素模长都是 1，因此：

  

$$\vert{}\mathbf a_l\vert{}=\sqrt N.$$

所以：

  

$$\boxed{\mathbf v_l = \frac1{\sqrt N}\mathbf a_l.}$$

组成矩阵：

  

$$\mathbf V = [\mathbf v_1,\ldots,\mathbf v_L].$$

那么：

  

$$\boxed{\mathbf V = \frac1{\sqrt N}\mathbf A.}$$

代码里的：

  

Matlab

```
V = A ./ sqrt(sum(abs(A).^2,1));
```

本质上就是这个操作。

  

#### 3.3 Delay compensation 在做什么

原来的第 $l$ 条 path 有：

  

$$e^{-j2\pi f_k\tau_l}.$$

因此显式 delay compensation 使用相反相位：

  

$$e^{+j2\pi f_k\tau_l}.$$

组成：

  

$$\boxed{\mathbf D[k] = \operatorname{diag} \left( e^{+j2\pi f_k\tau_1}, \ldots, e^{+j2\pi f_k\tau_L} \right).}$$

代码：

  

Matlab

```
Dk = diag(exp(1j*2*pi*f_bb(k)*tau));
```

#### 3.4 Path combining 后最终是什么

令：

  

$$\mathbf c = \boldsymbol\alpha.$$

整个 structured receiver：

  

$$\boxed{z[k] = \mathbf c^H \mathbf D[k] \mathbf V^H \mathbf y[k].}$$

流程可以理解成：

  

Plaintext

```
y[k]
  ↓
Vᴴ
  ↓
path-beam outputs
  ↓
D[k]
  ↓
delay compensation
  ↓
cᴴ
  ↓
path combining
  ↓
z[k]
```

#### 3.5 核心：把这一大串变成一个等效 $\mathbf w$

一般接收器写成：

  

$$z[k] = \mathbf w^H[k]\mathbf y[k].$$

而这里：

  

$$z[k] = \mathbf c^H \mathbf D[k] \mathbf V^H \mathbf y[k].$$

所以定义：

  

$$\boxed{\mathbf w_{\rm Path}[k] = \mathbf V \mathbf D^H[k] \mathbf c.}$$

因为：

  

$$\mathbf w_{\rm Path}^H = \mathbf c^H\mathbf D\mathbf V^H.$$

MATLAB 中：

  

Matlab

```
w_eq = V * Dk' * c_path;
```

这里 `w_eq` 的含义非常关键：

  

前面虽然写成了 beam → delay → combining 三个模块，但从天线输入到最终输出整体看，它仍然只是一个等效空间接收向量 $\mathbf w$。

这也是你以后判断“新算法到底是不是新的自由度”的重要方法。

  

#### 3.6 核心等价证明

现在把所有定义代回去。

  

**3.6.1 第一步**

  

$$\mathbf V = \frac1{\sqrt N}\mathbf A.$$

因此：

  

$$\mathbf w_{\rm Path}[k] = \frac1{\sqrt N} \mathbf A \mathbf D^H[k] \boldsymbol\alpha.$$

**3.6.2 第二步**

因为：

  

$$\mathbf D^H[k] = \operatorname{diag} \left( e^{-j2\pi f_k\tau_1}, \ldots, e^{-j2\pi f_k\tau_L} \right),$$

所以：

  

$$\mathbf D^H[k]\boldsymbol\alpha = \begin{bmatrix} \alpha_1e^{-j2\pi f_k\tau_1} \\ \vdots \\ \alpha_Le^{-j2\pi f_k\tau_L} \end{bmatrix}.$$

**3.6.3 第三步**

继续左乘 $\mathbf A$：

  

$$\mathbf A \mathbf D^H[k] \boldsymbol\alpha = \sum_l \alpha_l e^{-j2\pi f_k\tau_l} \mathbf a_l.$$

但根据最开始的定义：

  

$$\boxed{\mathbf h[k] = \sum_l \alpha_l e^{-j2\pi f_k\tau_l} \mathbf a_l.}$$

所以：

  

$$\boxed{\mathbf A \mathbf D^H[k] \boldsymbol\alpha = \mathbf h[k].}$$

最终：

  

$$\boxed{\mathbf w_{\rm Path}[k] = \frac1{\sqrt N}\mathbf h[k].}$$

而：

  

$$\boxed{\mathbf w_{\rm MRC}[k] = \mathbf h[k].}$$

因此：

  

$$\boxed{\mathbf w_{\rm Path}[k] \propto \mathbf w_{\rm MRC}[k].}$$

证明结束。

  

### 4. 为什么你的仿真结果必然重合

#### 4.1 Fixed ULA 情况

给定一个固定：

  

$$\mathbf P_{\rm Fixed},$$

信道：

  

$$\mathbf h[k;\mathbf P_{\rm Fixed}]$$

也就确定了。

MRC：

  

$$\mathbf w_{\rm MRC} = \mathbf h.$$

Path-DA：

  

$$\mathbf w_{\rm Path} = \frac1{\sqrt N}\mathbf h.$$

所以：

  

$$\boxed{\gamma_{\rm MRC}[k] = \gamma_{\rm Path}[k].}$$

自然：

  

$$\boxed{R_{\rm Fixed,MRC} = R_{\rm Fixed,Path}.}$$

这就是：

  

$$5.3536=5.3536.$$

#### 4.2 MA 情况

MA 时：

  

$$\mathbf P$$

可以改变。

但注意，对任意一个候选位置 $\mathbf P$，都有：

  

$$\boxed{\mathbf w_{\rm Path}[k;\mathbf P] = \frac1{\sqrt N} \mathbf h[k;\mathbf P].}$$

因此：

  

$$R_{\rm Path}(\mathbf P) = R_{\rm MRC}(\mathbf P)$$

对所有 $\mathbf P$ 都成立。

那么自然：

  

$$\boxed{\max_{\mathbf P}R_{\rm Path}(\mathbf P) = \max_{\mathbf P}R_{\rm MRC}(\mathbf P).}$$

不仅最终 rate 相同，整个关于 $\mathbf P$ 的 objective function 都相同。

所以：

  

$$\boxed{\mathbf P_{\rm Path}^\star = \mathbf P_{\rm MRC}^\star.}$$

这就是为什么你的代码同时得到：

  

$$5.3575=5.3575$$

以及：

  

$$[0.25,\;0.75,\;3,\;3.5]\lambda$$

完全一样。

程序也确实分别对 `rate_mrc_all` 和 `rate_path_all` 做 MA 位置搜索。

  

### 5. 为什么 perfect CSI + OFDM + full digital 很关键

不是说：

  

“任何情况下 Path-DA 都等于 MRC。”

真正的意思是：当前这些理想条件把 path-wise processing 原本可能发挥作用的限制全部拿掉了。

  

#### 5.1 OFDM 已经如何处理 multipath

时域中原本是：

  

Plaintext

```
x[n]
│
├─ delay τ1 → ×α1 ─┐
│                  │
├─ delay τ2 → ×α2 ─┼→ convolution
│                  │
└─ ...             ┘
```

这是一个多径卷积信道。

但在：

  

$$\boxed{\text{CP 足够}+\text{FFT}}$$

之后，时域 convolution 变成每个子载波上的乘法：

  

$$\boxed{\mathbf y[k] = \mathbf h[k]x[k]+\mathbf n[k].}$$

于是：

  

Plaintext

```
AoA
delay
path gain
   │
   ↓
所有 multipath
   │
   ↓
打包进 h[k]
```

也就是说，delay 并没有消失，而是已经进入：

  

$$e^{-j2\pi f_k\tau_l}$$

并最终成为 $\mathbf h[k]$ 的一部分。这个也是你原先笔记中强调的核心物理解释。

  

#### 5.2 Perfect CSI 为什么很关键

Perfect CSI 意味着接收端直接知道：

  

$$\boxed{\mathbf h[k]}.$$

注意它已经是：

  

$$\mathbf h[k] = \sum_l \alpha_l e^{-j2\pi f_k\tau_l} \mathbf a_l.$$

也就是说：

  

$$\boxed{\theta_l,\tau_l,\alpha_l \text{ 的综合作用已经全部包含在 }\mathbf h[k].}$$

对于：

  

$$\mathbf y[k] = \mathbf h[k]x[k]+\mathbf n[k],$$

MRC 实际上不关心：

  

Plaintext

```
h[k] 是
├─ 2 条 path 组成
├─ 5 条 path 组成
└─ 10 条 path 组成
```

它只需要最终的：

  

$$\boxed{\mathbf h[k]}.$$

#### 5.3 Full digital 为什么很关键

Full digital 意味着每个子载波都能够单独实现：

  

$$\mathbf w[k].$$

即：

  

Plaintext

```
subcarrier 1 → w[1]
subcarrier 2 → w[2]
subcarrier 3 → w[3]
...
subcarrier K → w[K]
```

因此我们可以毫无限制地选择：

  

$$\boxed{\mathbf w[k]=\mathbf h[k].}$$

这已经是白噪声单用户条件下的最优 MRC。

所以你再：

  

Plaintext

```
先拆 path
   ↓
beam
   ↓
delay compensation
   ↓
combine
```

如果最后仍然只是构造出一个任意的：

  

$$\mathbf w[k],$$

就没有理由超过已经最优的 MRC。

  

#### 5.4 所以 Delay Alignment 到底去哪了？

这里非常容易产生误解。

不是：

  

$$\boxed{\text{MRC 没有处理 delay}}$$

而是：

  

Plaintext

```
Path-DA
τ1 → 显式补偿
τ2 → 显式补偿
      ↓
再 combine

MRC
τ1
τ2
 ↓
共同进入 h[k]
 ↓
直接匹配整个 h[k]
```

所以：

  

$$\boxed{\text{Path-DA 是先把 effective channel 拆开再重新构造；} \\ \text{MRC 是直接匹配最终 effective channel。}}$$

当前代码中，前一种方式最后恰好又重构出了：

  

$$\mathbf h[k].$$

### 6. MA 在这里到底负责什么

这件事情也需要和 Path-DA 分开。

  

#### 6.1 MA 控制的是 $\mathbf P$

$$\boxed{\mathbf P \rightarrow \mathbf A(\mathbf P) \rightarrow \mathbf h[k;\mathbf P].}$$

所以 MA 实际负责：

  

$$\boxed{\text{改变信道本身的空间结构。}}$$

#### 6.2 MRC 负责的是给定信道之后怎么收

信道确定以后：

  

$$\mathbf h[k;\mathbf P]$$

已经确定。

MRC 再设计：

  

$$\boxed{\mathbf w[k]\propto\mathbf h[k;\mathbf P].}$$

因此完整逻辑：

  

Plaintext

```
MA position P
      ↓
改变 spatial response A(P)
      ↓
改变 h[k;P]
      ↓
收到 y[k]
      ↓
设计 digital combiner w[k]
      ↓
SNR / Rate
```

所以可以先这样记：

  

$$\boxed{\text{MA：改变信道}}$$

$$\boxed{\text{MRC：利用已经形成的信道}}$$

而当前这个 matched Path-DA：

  

$$\boxed{\text{只是从 path 参数重新构造 MRC 所需要的 }\mathbf h[k].}$$

### 7. ZF-path optimal 现在需要学多少

#### 7.1 只需要知道它想解决什么

Matched beam：

  

$$\mathbf v_l = \frac{\mathbf a_l}{\sqrt N}$$

并不能保证 path 真的分开，因为：

  

$$\mathbf a_1^H\mathbf a_2 \neq 0$$

时会互相串扰。

ZF-path 想做：

  

Plaintext

```
A=[a1,a2,...]
      ↓
设计 V
      ↓
VᴴA = I
      ↓
真正把 path 解耦
```

#### 7.2 但 ZF 会改变噪声

原来：

  

$$\mathbf n\sim\mathcal{CN}(0,\sigma^2\mathbf I).$$

经过：

  

$$\mathbf V^H\mathbf n$$

以后，噪声 covariance 变为与 $\mathbf V$ 有关的矩阵。

所以不能：

  

“ZF 分开 path 后直接加起来。”

必须正确考虑新的噪声 covariance。

代码中的 Scheme 5 正是在同一个 MA 位置上做 ZF path separation + delay alignment + optimal path-domain combining，并用它验证和 full-digital MRC 的等价上界。

  

#### 7.3 当前不用继续推

你现在知道下面这句话就够：

  

$$\boxed{\text{如果 path-domain 变换没有丢信息，} \\ \text{且后续正确考虑 transformed noise，} \\ \text{它并不会凭空创造比 full-digital MRC 更多的信息。}}$$

ZF 伪逆完整证明，放以后。

  

### 8. 你现在需要学到什么程度

这部分是为了防止继续钻牛角尖。

  

#### 8.1 必须完全掌握

这些要达到不看笔记能够自己讲出来。

  

**8.1.1 宽带多径信道**

  

$$\boxed{\mathbf h[k] = \sum_l \alpha_l e^{-j2\pi f_k\tau_l} \mathbf a_{\mathbf P}(\theta_l)}$$

而且知道：

  

Plaintext

```
α_l → path gain
θ_l → 空间结构
τ_l → 频率相关 phase
P   → MA position
```

**8.1.2 OFDM 子载波模型**

  

$$\boxed{\mathbf y[k] = \mathbf h[k]x[k]+\mathbf n[k].}$$

**8.1.3 MRC**

知道：

  

$$\boxed{\mathbf w_{\rm MRC}[k]\propto\mathbf h[k].}$$

并能解释这是通过最大化：

  

$$\frac{ \vert{}\mathbf w^H\mathbf h\vert{}^2 }{ \vert{}\mathbf w\vert{}^2 }$$

得到的。

不要求你背完整证明，但 Cauchy-Schwarz 那一步要看得懂。

  

**8.1.4 Path-DA 等效权重**

从：

  

$$z = \mathbf c^H\mathbf D\mathbf V^H\mathbf y$$

能够得到：

  

$$\boxed{\mathbf w_{\rm Path} = \mathbf V\mathbf D^H\mathbf c.}$$

**8.1.5 最重要的一条证明**

必须自己能够推：

  

$$\mathbf V = \frac1{\sqrt N}\mathbf A$$

所以：

  

$$\begin{aligned} \mathbf w_{\rm Path}[k] &= \mathbf V\mathbf D^H[k]\boldsymbol\alpha \\ &= \frac1{\sqrt N} \mathbf A\mathbf D^H[k]\boldsymbol\alpha \\ &= \frac1{\sqrt N}\mathbf h[k]. \end{aligned}$$

因此：

  

$$\boxed{\mathbf w_{\rm Path} \propto \mathbf w_{\rm MRC}.}$$

这一条能自己推出来，这一阶段就算过关。

你的原始笔记也把这五项列为了当前必须掌握的 Level 1。

  

#### 8.2 理解概念即可

下面这些现在只需要知道“它在解决什么”。

  

Plaintext

```
CP
├─ 为什么最大时延在 CP 内时不会产生跨 symbol ISI
└─ 为什么可以构造循环卷积

FFT
└─ 为什么循环卷积变成逐频点乘法

ZF
├─ 为什么可以尝试分离 path
└─ 为什么可能放大/改变 noise

MMSE
└─ 为什么 interference/colored noise 后出现 R⁻¹h
```

以后真正用到哪个，再往下补哪个。

  

#### 8.3 现在不要深入

暂时不用碰：

  

Plaintext

```
连续时间 OFDM 的完整严格推导
DFT 矩阵所有性质
ZF 伪逆完整证明
RAKE 的历史演化
各种 DAM 变种
sufficient statistics
复杂随机过程证明
TTD / PS 完整硬件架构
```

这些都不会改变当前最重要的研究判断：

  

$$\boxed{\text{当前 Path-DA 是否产生了新的自由度？}}$$

答案现在已经非常清楚：

  

$$\boxed{\text{没有。}}$$

### 9. 如何判断自己是不是在钻牛角尖

#### 9.1 一个非常实用的判断标准

问自己：

  

> 搞懂这个知识以后，会不会改变我对研究方案是否成立的判断？
> 
>   

如果会，就值得现在学。

如果不会，先放下。

  

#### 9.2 当前值得学的例子

为什么 $\mathbf w_{\rm Path}\propto\mathbf h$？

值得。

因为它直接告诉你：

  

$$\boxed{\text{Path-DA 目前并不是一个新的接收自由度。}}$$

#### 9.3 当前不值得继续钻的例子

Cauchy-Schwarz 能不能从内积空间公理开始证明？

现在不值得。

因为无论证明得多深，都不会改变：

  

$$\mathbf w_{\rm MRC}\propto\mathbf h$$

以及后面的研究判断。

  

### 10. 当前阶段的学习闭环

你真正要形成的是下面这棵树：

  

Plaintext

```
(α_l, θ_l, τ_l)
        ↓
    A(P)
        ↓
    d[k]
        ↓
h[k] = A(P)d[k]
        ↓
y[k] = h[k]x[k] + n[k]
        ↓
┌───────────────────┴───────────────────┐
↓                                       ↓
MRC                                  Path-DA
w_MRC ∝ h                     w_Path = V Dᴴ α
                                         ↓
                                    V = A/√N
                                         ↓
                                 w_Path = h/√N
                                         ↓
                   ┌─────────────────────┘
                   ↓
          w_Path ∝ w_MRC
                   ↓
      perfect-CSI 条件下性能重合
```

这棵树如果你能够自己完整讲出来，就停止继续补这部分理论。

  

### 11. 下一步

#### 11.1 现在不要急着做新的 sweep

当前最合理的动作是把现有代码真正变成你自己的。

只拆三个函数：

  

Plaintext

```
build_channel
├─ A 是什么
├─ d[k] 是什么
├─ H(:,k) 为什么 = A*d_k
└─ 每个变量尺寸是什么

snr_mrc
├─ 输出 SNR 从哪来
├─ 为什么不显式生成 w
└─ 为什么直接计算 ||h[k]||²

snr_matched_path_receiver
├─ V 从哪里来
├─ Dk 从哪里来
├─ alpha 为什么作为 c
├─ w_eq 为什么 = V Dkᴴ alpha
└─ 为什么最终 = h[k]/√N
```

#### 11.2 然后才回到真正研究问题

等这一层完全吃透，下一步不再问：

  

“还能不能设计一个更厉害的 delay alignment？”

而应该问：

  

$$\boxed{\text{既然理想条件下 Path-DA=MRC，} \\ \text{到底打破哪个理想条件之后，} \\ \text{path structure 才真正有新的价值？}}$$

这才是后面的研究入口。可能的分支是 imperfect CSI、limited RF chains、interference、CP 不足等，但现在先不要展开。先把上面的等价链彻底掌握，再进入下一层。