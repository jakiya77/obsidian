
# 1) 信号模型

- 三阵元复基带、窄带、平面波：  
$$
    \mathbf{x}[n] = s[n]\mathbf{a}_t + j[n]\mathbf{a}_j + \mathbf{n}[n]  
$$
$$
    其中 ( \mathbf{a}_t,\mathbf{a}_j \in \mathbb{C}^3 ) 为目标/干扰方向向量；  
   进行能量归一：(\mathbf{a}_t\leftarrow \mathbf{a}_t/\sqrt{3},;\mathbf{a}_j\leftarrow \mathbf{a}_j/\sqrt{3})。
   
$$

    
- 功率定义 (P_s=1)：  
$$
      \mathbb{E}{|s[n]|^2}=P_s,\quad \mathbb{E}{|j[n]|^2}=P_j,\quad \mathbb{E}{\mathbf{n}\mathbf{n}^H}=\sigma^2\mathbf{I}  
$$
    且用输入 SNR/SJR 设定：  
$$
      \text{SNR}_\text{in}=\frac{P_s}{\sigma^2},\quad \text{SJR}=\frac{P_s}{P_j}  
    \Rightarrow
    \sigma^2=\frac{P_s}{10^{\text{SNR}_\text{in}/10}}, P_j=\frac{P_s}{10^{\text{SJR}/10}}  
$$
    
$$
波束形成输出：y[n]=\mathbf{w}^H\mathbf{x}[n]，其中 \mathbf{w} 为 LMS 训练得到的收敛权。
$$
    

# 2) 输出分量功率

$$
在 (s,j,\mathbf{n}) 彼此独立假设下：
$$

- **信号分量功率**：  
    
$$
    P_\text{sig,out}=\mathbb{E}{|\mathbf{w}^H (s,\mathbf{a}_t)|^2}  
    = |\mathbf{w}^H\mathbf{a}_t|^2 P_s  
$$
    
    
- **干扰分量功率**：  
$$
   
    P_\text{jam,out}=\mathbb{E}{|\mathbf{w}^H (j,\mathbf{a}_j)|^2}  
    = |\mathbf{w}^H\mathbf{a}_j|^2 P_j  

    
$$
- **噪声分量功率**（空间白噪）：  
    
$$
    P_\text{noise,out}=\mathbb{E}{|\mathbf{w}^H\mathbf{n}|^2}  
    = \mathbf{w}^H(\sigma^2\mathbf{I})\mathbf{w}  
    = \sigma^2||\mathbf{w}||_2^2  
$$
    
    

# 3) 输出 SINR（线性 & dB）

因此**解析法**输出 SINR：  

$$
\boxed{\text{SINR}_\text{out}  
=\frac{|\mathbf{w}^H\mathbf{a}_t|^2 P_s}{|\mathbf{w}^H\mathbf{a}_j|^2 P_j + \sigma^2|\mathbf{w}|_2^2}}  
$$

转 dB：  

$$
\text{SINR}_\text{out,dB}=10\log_{10}\left(\frac{|\mathbf{w}^H\mathbf{a}_t|^2 P_s}{|\mathbf{w}^H\mathbf{a}_j|^2 P_j + \sigma^2|\mathbf{w}|_2^2} \right)  
$$


对应脚本里 `out_sinr_db` / `eval_once_metrics` 里的三项：`num = Gt_lin*Ps; den = Gj_lin*Pj + sigma2*||w||^2;`。
$$

> 备注：把 (\mathbf{a}_t,\mathbf{a}_j) 归一到 (|\cdot|=1)（(/\sqrt{3})）会影响 ( |\mathbf{w}^H\mathbf{a}|^2 ) 的标度，但同时作用于分子/分母的方向性增益因子，一致使用即可，不影响不同 (x) 之间的相对比较与优化。
$$

# 4) 与一般阵列协方差形式的关系

一般写法是  

$$
\text{SINR}_\text{out}=\frac{\mathbf{w}^H\mathbf{R}_s\mathbf{w}}{\mathbf{w}^H(\mathbf{R}_j+\mathbf{R}_n)\mathbf{w}}  
$$

$$
其中 (\mathbf{R}_s=P_s\mathbf{a}_t\mathbf{a}_t^H)，(\mathbf{R}_j=P_j\mathbf{a}_j\mathbf{a}_j^H)，(\mathbf{R}_n=\sigma^2\mathbf{I})。把它们代回去就得到上面的“标量版”。
$$

$$
> 多个干扰时：(\mathbf{R}_j=\sum_\ell P_{j,\ell}\mathbf{a}_{j,\ell}\mathbf{a}_{j,\ell}^H)。  
> 有色噪声时：(\mathbf{R}_n\neq\sigma^2\mathbf{I})，用 (\mathbf{w}^H\mathbf{R}_n\mathbf{w}) 取代 (\sigma^2|\mathbf{w}|^2)。
$$

# 5) ΔG 为什么与 SNR/SJR 无关


$$
G_\text{tgt} = 10\log_{10}|\mathbf{w}^H\mathbf{a}_t|^2,\quad  
G_\text{jam} = 10\log_{10}|\mathbf{w}^H\mathbf{a}_j|^2,\quad  
\Delta G = G_\text{tgt}-G_\text{jam}  
$$

它只反映**波束指向性**（几何+权值），不含 (P_s,P_j,\sigma^2)。所以常同时看 **SINR**（含功率与噪声）和 **ΔG**（纯指向）的曲线，解释更完整。

# 6) 代码里的实现

```matlab
% 已有：a_t, a_j 归一；Ps=1；Pj, sigma2 由 SJR_dB/SNR_dB 决定
Gt_lin = abs(w' * a_t)^2;  % 方向增益（线性）
Gj_lin = abs(w' * a_j)^2;

num = Gt_lin * Ps;
den = Gj_lin * Pj + sigma2 * (w' * w);  % = sigma2 * ||w||^2

SINR_dB = 10*log10(real(num/den) + eps);
```

# 7) 想“数值验算”而不是解析？

可以用**独立测试序列**直接测输出功率比，验证解析法是否吻合：

```matlab
% 生成测试集（与训练独立）
Ntest = 20000;
s  = (randn(Ntest,1)+1j*randn(Ntest,1))*sqrt(Ps/2);
j  = (randn(Ntest,1)+1j*randn(Ntest,1))*sqrt(Pj/2);
n  = (randn(Ntest,3)+1j*randn(Ntest,3))*sqrt(sigma2/2);
X  = s.*(ones(1,3).* (a_t.')) + j.*(ones(1,3).* (a_j.')) + n;

y  = X * conj(w);              % Ntest x 1
% 用“已知分量”计算每部分输出功率（与解析应一致）
y_sig  = (s.*(ones(1,3).*(a_t.'))) * conj(w);
y_jam  = (j.*(ones(1,3).*(a_j.'))) * conj(w);
y_noi  = n * conj(w);

SINR_emp = var(y_sig) / ( var(y_jam) + var(y_noi) );
SINR_emp_dB = 10*log10(SINR_emp);
```

一般在 (N_\text{train}) 足够、LMS 收敛、测试样本够大时，`SINR_emp_dB` 会与解析版 `SINR_dB` **非常接近**（差别来自有限样本统计误差）。
