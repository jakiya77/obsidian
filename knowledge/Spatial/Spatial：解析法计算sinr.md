
# 1) 场景建模（窄带、平面波）

对一个窄带阵列接收模型，单个目标 + 单个干扰 + 加性噪声时：  
$$
\mathbf x(t)=\mathbf a_s, s(t)+\mathbf a_j, j(t)+\mathbf n(t),  

$$

$$
(\mathbf a_s,\mathbf a_j\in\mathbb C^{D\times 1})
$$
- ：目标/干扰的**阵列流形**（由方位角、阵列几何、元素方向图决定）
    
- (s(t),j(t))：目标与干扰的复基带信号
    
- n(t)：每通道独立白噪声（方差 (\sigma^2)）
    线性波束形成输出：  

$$
y(t)=\mathbf w^{\mathrm H}\mathbf x(t)=  
\underbrace{\mathbf w^{\mathrm H}\mathbf a_s}_{\text{对目标的复增益}}, s(t)+  
\underbrace{\mathbf w^{\mathrm H}\mathbf a_j}_{\text{对干扰的复增益}}, j(t)+  
\underbrace{\mathbf w^{\mathrm H}\mathbf n(t)}_{\text{输出噪声}}.  


$$
# 2) 功率分解（期望值层面）

若三路互不相关（常见假设），各自的**平均功率**为  
[  
P_s=\mathbb E|s|^2,\quad P_j=\mathbb E|j|^2,\quad  
\mathbb E[\mathbf n\mathbf n^{\mathrm H}]=\sigma^2\mathbf I,  
]  
则输出端三项功率为  
[  
\begin{aligned}  
P_{s,\text{out}} &= |\mathbf w^{\mathrm H}\mathbf a_s|^2, P_s,\  
P_{j,\text{out}} &= |\mathbf w^{\mathrm H}\mathbf a_j|^2, P_j,\  
P_{n,\text{out}} &= \mathbb E|\mathbf w^{\mathrm H}\mathbf n|^2  
= \mathbf w^{\mathrm H}(\sigma^2\mathbf I)\mathbf w  
= \sigma^2|\mathbf w|^2.  
\end{aligned}  
]

于是**解析型输出 SINR**就是  
$[  
\boxed{  
\mathrm{SINR}_{\text{out}}=  
\frac{P_s,|\mathbf w^{\mathrm H}\mathbf a_s|^2}  
{P_j,|\mathbf w^{\mathrm H}\mathbf a_j|^2+\sigma^2|\mathbf w|^2}  
}  
]  
这就是我给你的那条公式。

> 更一般的形式：(\displaystyle \mathrm{SINR}=\frac{\mathbf w^{\mathrm H}\mathbf R_s\mathbf w}{\mathbf w^{\mathrm H}(\mathbf R_j+\mathbf R_n)\mathbf w})。  
> 对于“单个窄带平面波”，(\mathbf R_s=P_s\mathbf a_s\mathbf a_s^{\mathrm H})、(\mathbf R_j=P_j\mathbf a_j\mathbf a_j^{\mathrm H})、(\mathbf R_n=\sigma^2\mathbf I)，就化成上面那条标量式。

# 3) 混合阵列（子阵 + 数字 4 通道）怎么套？

你代码里每个 8×8 子阵（64 元）先**模拟合成**成一个通道，再做 4 路数字合成。把子阵内的“模拟波束”作用等价进流形即可：

- 对第 (m) 个子阵：(,g_{t,m}=\mathbf w_m^{\mathrm H}\mathbf a_{t,m})，(,g_{j,m}=\mathbf w_m^{\mathrm H}\mathbf a_{j,m})  
    （(\mathbf w_m) 是子阵权，(\mathbf a_{t,m}) 是该子阵看到的目标流形）
    
- 4 路数字侧把 (\mathbf g_t=[g_{t,1},\dots,g_{t,4}]^{\mathrm T}) 视作“等效流形”，最终输出就是：  
    [  
    \mathrm{SINR}_{\text{out}}=  
    \frac{P_s,|\mathbf v^{\mathrm H}\mathbf g_t|^2}  
    {P_j,|\mathbf v^{\mathrm H}\mathbf g_j|^2+\sigma^2|\mathbf v|^2},  
    ]  
    其中 (\mathbf v) 即你求出来的 4×1 数字权（代码里的 `w_optimal`），(\sigma^2) 是**每数字通道**的噪声功率。  
    这与我在你代码中用 `gt, gj, w_optimal, noise_power` 的计算**完全一致**。
    

# 4) 为什么“解析法”通常更准？

- **不依赖有限窗估计**：它直接用“已知/可控”的 (P_s,P_j,\sigma^2) 和**瞬时权值** (\mathbf w)，避免“有限样本 + 投影估计”的正偏差（把噪声投到导频方向、让 (P_s) 偏大、(P_{jn}) 偏小）。
    
- **没有对齐误差/通道失配的耦合**：数据窗法要截窗、对齐、做 LS，任何 1–2 个采样的误差都会压低 (|s^Hr|)；解析法在**功率层面**，天然免疫这类“估计步骤的抖动”。
    
- **口径一贯**：只要你在仿真端用的 (P_s,P_j,\sigma^2) 与解析里的一致（你代码里就是 `signal_power / jam_power / noise_power`），且 (\mathbf w) 用和输出同一份（不要换成“另一个归一化版本”），就跟物理一致。
    

> 这就是为什么你修正“子阵二次除 (\sqrt{N})”之后，(θ=0°) 一带**解析值** ≈ 24.08 dB，而**数据窗**原来 24.19 dB 只是 +0.1 dB 的估计偏差；把数据窗做“无偏修正”后，两者就几乎重合。

# 5) 与 “10·log10 D 上限” 的关系

在**纯噪声场**（(P_j=0)）且把 (\mathbf w) 取为“指向目标的匹配滤波”时：  
[  
\frac{\mathrm{SNR}_{\text{out}}}{\mathrm{SNR}_{\text{in}}}  
=\frac{|\mathbf w^{\mathrm H}\mathbf a_s|^2}{|\mathbf w|^2}  
\le |\mathbf a_s|^2 = D_{\text{eq}},  
]  
这里 (D_{\text{eq}}) 是**等效相干阵元数**。你的结构中  
(D_{\text{eq}}=64)（子阵）× (4)（数字）= **256** ⇒ 上限 **10·log10(256)=24.08 dB**。  
解析法天然就会“看见”这个上限；而数据窗若不做无偏化，容易比它高出 ~0.1–0.3 dB。

# 6) 什么时候解析法会“不准”？

它基于的前提是“模型与现实匹配”。常见使之偏离的情况与修正方式：

- **噪声非白/有互相关**：把 (\sigma^2\mathbf I) 替换为真实 (\mathbf R_n)（上面一般式）。
    
- **宽带/频率选择性**：单一流形不够；需分频点/分子带做合成，或采用时域广义模型。
    
- **元素方向图/互耦/校准误差**：(\mathbf a(\theta,\phi)) 要包含真实幅相与标定。
    
- **模拟前端增益波动或 AGC**：确保你在解析里用的 (P_s,P_j,\sigma^2) 与实际前端的一致标度；或者把这些增益统一吸收到 (\mathbf a) 或 (\mathbf w) 的定义里。
    

---

**一句话总结**：  
解析法其实就是把“输出端三路功率”的**期望值**按物理直接写出来：  
目标项 = 阵列指向增益 × 输入功率；干扰与噪声项同理。  
它不吃有限样本与对齐损失，自然比“数据窗估计”更贴近“理论物理”，只要你的 (\mathbf a,\mathbf w, P_s,P_j,\sigma^2) 与仿真口径一致，它就会非常准。