## 使用最小二乘投影法

**原理和公式：**

**最小二乘一维拟合 = 正交投影 = 匹配滤波估计**
# 1) 我们在解什么？

把一小段接收信号建模为  

$$
\mathbf r = \alpha\mathbf s + \mathbf v$$
$$
\quad \mathbf v=\text{jam+noise}  

$$
$$
其中 (\mathbf s) 是已知参考，(\alpha\in\mathbb C) 是未知复增益（幅度+相位），(\mathbf v) 是干扰+噪声。
$$

$$
目标：找一个 \alpha，使得 \mathbf r 与 \alpha\mathbf s 尽量接近（最小化均方误差）：  
$$
$$
\hat\alpha=\arg\min_\alpha |\mathbf r-\alpha\mathbf s|_2^2.  
$$
# 2) 为什么叫“最小二乘投影”？

$$
对复数最小二乘问题求导对 \alpha^*正规方程：  
$$
$$

$$
$$
-\mathbf s^{\rm H}\mathbf r + \hat\alpha
$$
$$
\mathbf s^{\rm H}\mathbf s=0  
\Rightarrow
\boxed{\hat\alpha=\dfrac{\mathbf s^{\rm H}\mathbf r}{\mathbf s^{\rm H}\mathbf s}}  
$$
对应代码中的：

```matlab
alpha_ls = (s_seg * r_seg') / (s_seg * s_seg'); % <s,r>/<s,s>
```

于是
$$

$$
$$
 \mathbf s_{\text{comp}}=\hat\alpha,\mathbf s 是 \mathbf r 在子空间 \text{span}{\mathbf s} 上的正交投影；
$$
$$
    
残差 \mathbf q=\mathbf r-\mathbf s_{\text{comp}} ~与 ~\mathbf s~正交~ \mathbf s^{\rm H}\mathbf q=0——这就是投影定理。
$$
    

# 3) “互相关归一化”的关系

(\mathbf s^{\rm H}\mathbf r) 就是**互相关**（匹配滤波）输出；再除以 (\mathbf s^{\rm H}\mathbf s) 等于“按 (\mathbf s) 的能量归一化”的版本。  
注意这**不是**通常的“相关系数” (\rho=\dfrac{\mathbf s^{\rm H}\mathbf r}{|\mathbf s|,|\mathbf r|})，而是估计把 (\mathbf s) **变成** (\mathbf r) 的最佳复比例 (\hat\alpha)。

# 4) 功率分解 & SINR 估计从哪来？

投影定理给出能量的毕达哥拉斯分解：  
$$
  
|\mathbf r|_2^2=|\mathbf s_{\text{comp}}|_2^2+|\mathbf r-\mathbf s_{\text{comp}}|_2^2.  
  
$$
将能量除以长度 L（样本数）就是**功率**：

- (P_r = \frac1L|\mathbf r|_2^2 \approx \text{mean}(|r|^2))
    
- (P_s = \frac1L|\mathbf s_{\text{comp}}|_2^2 \approx \text{mean}(|s_{\text{comp}}|^2))
    
- (P_{j+n} = \frac1L|\mathbf r-\mathbf s_{\text{comp}}|_2^2)
    

于是  
(\displaystyle \text{SINR} \approx \dfrac{P_s}{P_{j+n}})。

你现在写的是：

```matlab
P_r = mean(abs(r_seg).^2);
P_s = mean(abs(s_comp).^2);
P_jn = max(P_r - P_s, eps);
SINR_cc_full_dB = 10*log10(P_s / P_jn);
```

这用的是上面的能量等式（用 (P_r-P_s) 代替 (P_{j+n})）。数值上没问题，但**更稳**的写法是直接用残差算功率：

```matlab
res   = r_seg - s_comp;
P_jn  = mean(abs(res).^2);              % 更数值稳健
SINR_dB = 10*log10( P_s / max(P_jn,eps) );
```

# 5) 直观小结（一句话版）

- **最小二乘**：选 (\alpha) 让 (\alpha s) 最接近 (r)。
    
- **投影**：(\alpha s) 是把 (r) 投到 (s) 方向的影子；残差与 (s) 正交。
    
- **互相关**：(\alpha) 就是“匹配滤波/互相关”再按 (|s|^2) 归一后的复比例。
    
- **SINR**：用投影部分的功率当“信号功率”，残差功率当“干扰+噪声功率”。
    

# 6) 常见注意点

- **对齐**：(s) 必须与 (r) 中的真实信号**同相位/对齐**（时间、频率、载波相位）。否则 (P_s) 会被低估。
    
- **相关干扰**：若干扰与 (s)**相关**（比如欺骗式干扰），投影会把其一部分当成信号 → SINR 被高估。这时应用**加权最小二乘/白化**：  
    [  
    \hat\alpha = (s^{\rm H}R_{v}^{-1}s)^{-1}s^{\rm H}R_{v}^{-1}r  
    ]  
    用 (R_v)（干扰+噪声协方差）做加权，相当于**颜色噪声下的匹配滤波器**。
    

# 7) 参考代码（更稳健的三行替换）

```matlab
alpha_ls = (s_seg * r_seg') / (s_seg * s_seg');  % 复最小二乘/投影系数
s_comp   = alpha_ls * s_seg;                     % 投影分量（信号成分估计）
res      = r_seg - s_comp;                       % 残差（干扰+噪声估计）

P_s  = mean(abs(s_comp).^2);
P_jn = mean(abs(res).^2);                        % 推荐直接算残差功率
SINR_cc_full_dB = 10*log10( P_s / max(P_jn,eps) );
```

如果你愿意，我们还能把这块封成一个小函数（支持复数、自动检测行/列向量、返回 (\hat\alpha)、(P_s)、(P_{jn})、SINR），直接插到你的管线里用。
$$
\alpha = \frac{s_{seg}.r_{seg}^H}{s_{seg}.s_{seg}^H} ~~~ 
$$
* s 是参考信号，r是恢复信号
% 最小二乘投影（等价于互相关归一化），分解 r = s_comp + (j+n)

alpha_ls = (s_seg * r_seg') / (s_seg * s_seg'); % <s,r> / <s,s>

s_comp = alpha_ls * s_seg; % r 中的“信号成分”

P_r = mean(abs(r_seg).^2);

P_s = mean(abs(s_comp).^2);

P_jn = max(P_r - P_s, eps); % 避免数值负值

SINR_cc_full_dB = 10*log10(P_s / P_jn);

结论先说：**两种 SINR 算法在标准假设下是一致的，且可相互推导**。你代码里的“解析式”  
[  
\mathrm{SINR}_{\text{ana}}  
=\frac{|w^{\mathrm H}a_t|^2 P_s}{|w^{\mathrm H}a_j|^2 P_j+\sigma^2|w|^2}  
]  
与“最小二乘投影（互相关归一化）”得到的  
[  
\mathrm{SINR}_{\text{LS}}  
=\frac{P_s^{(\text{comp})}}{P_{j+n}^{(\text{res})}}  
]  
在**(i) 参考序列对齐正确、(ii) 各分量零均值互不相关、(iii) 采样段足够长**时，**一致（渐近相等）**。

---

## 为什么一致？（一行式推导）

阵列输出：  
(y[n]=w^{\mathrm H}x[n]=g_t,s[n]+g_j,j[n]+v[n])，  
其中 (g_t=w^{\mathrm H}a_t,; g_j=w^{\mathrm H}a_j,; \mathbb E|v|^2=\sigma^2|w|^2)。

做最小二乘投影：  
(\hat\alpha=\dfrac{\langle s,y\rangle}{\langle s,s\rangle}\xrightarrow[N\to\infty]{} g_t)（交叉项 (\langle s,j\rangle,\langle s,v\rangle) 期望为 0）。  
于是  
[  
P_s^{(\text{comp})}=\frac{|\hat\alpha s|^2}{N}\to |g_t|^2 P_s,\quad  
P_{j+n}^{(\text{res})}=\frac{|y-\hat\alpha s|^2}{N}\to |g_j|^2 P_j+\sigma^2|w|^2,  
]  
从而 (\mathrm{SINR}_{\text{LS}}\to \mathrm{SINR}_{\text{ana}})。

---

## 什么时候会不完全相等（偏差来源）

- **有限样本**：段长不够时，(\langle s,j\rangle) 等交叉项不为 0 → 统计起伏（几十分之几 dB 常见）。
    
- **参考不匹配/未对齐**：时间/相位/频偏没对齐，(\hat\alpha) 会低估 (g_t)。
    
- **相关干扰**：若干扰与 (s) 同型或相关（例如“欺骗式”干扰、同导频），投影会把干扰的一部分当成“信号成分”，(\mathrm{SINR}_{\text{LS}}) 偏高。
    
- **噪声/干扰有色**：解析式用的是 (\sigma^2 I)。一般情形应写成分母 (w^{\mathrm H}R_{v}w)（(R_v) 为干扰+噪声协方差）。你当前单一窄带干扰 + 白噪声时恰好化简为代码形式。
    

---

## 一段自检代码（把两者放到同一输出上对比）

```matlab
% 已有：X (Ntrainx3), w (3x1), a_t, a_j, Ps, Pj, sigma2
y = X * conj(w);                 % 阵列输出（等价于逐样 w'*x_n）
s = d;                           % 你的期望信号序列（确保与 y 对齐）
alpha_ls = (s' * y) / (s' * s);  % 最小二乘/投影
s_comp   = alpha_ls * s;
res      = y - s_comp;

SINR_LS_dB  = 10*log10( mean(abs(s_comp).^2) / max(mean(abs(res).^2),eps) );

Gt = abs(w' * a_t)^2; Gj = abs(w' * a_j)^2;
SINR_ana_dB = 10*log10( (Gt*Ps) / (Gj*Pj + sigma2*(w'*w)) );

disp([SINR_LS_dB, SINR_ana_dB]);   % 数值应非常接近（长段更接近）
```

---

## 可互推的直观映射

- **从 LS 推解析式**：(\hat\alpha\approx g_t)，用 (P_s^{(\text{comp})}\approx |g_t|^2 P_s)，残差功率 (\approx |g_j|^2 P_j+\sigma^2|w|^2)；代回即得解析式。
    
- **从解析式推 LS**：把 (g_t s) 视为投影分量，((g_j j+v)) 视为正交残差分量，按功率分解（投影定理）即可得到 LS 形式。
    

---

## 小建议

- 计算 LS 版本时，**用残差直接算功率**（而不是 (P_r-P_s)），更数值稳：
    
    ```matlab
    res = r_seg - s_comp; 
    P_jn = mean(abs(res).^2);
    ```
    
- 为了公平性，**SJR/SNR 的标定**要用同一功率定义（你已在 `signal_generation` 里用均方功率归一，👍）。
    

> 总之：两种方法在你的建模假设下是一回事；LS 是“**数据驱动估计**”，解析式是“**模型显式计算**”。段长够、参考对齐好，它们会对上。