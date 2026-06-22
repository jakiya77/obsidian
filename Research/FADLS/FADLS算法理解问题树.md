# FADLS / SINR-greedy 算法理解问题树

#FADLS #MovableAntenna #MMSE #SINR #问题树 #论文仿真

---

## 0. 总问题

> 我代码中的 FADLS、SINR-greedy、MMSE 输出 SINR、复杂度评估和小规模 exhaustive benchmark 到底在算什么？  
> 每个公式的物理意义、数学来源、代码对应关系是什么？

---

# 1. 信道模型与 (H,G) 的含义

## 1.1 主问题：(h(p),g(p)) 是什么？

### ✅ 已解决

代码中：

```matlab
h_grid = exp(-1j*2*pi/lambda*p_grid*sind(theta_s).')*alpha;
g_grid = exp(-1j*2*pi/lambda*p_grid*sind(theta_j).')*beta;
```

对应：

$$  
h(p)=\sum_{\ell=1}^{L_s}\alpha_\ell e^{-j\frac{2\pi}{\lambda}p\sin\theta_{s,\ell}}  
$$

$$  
g(p)=\sum_{\ell=1}^{L_j}\beta_\ell e^{-j\frac{2\pi}{\lambda}p\sin\theta_{j,\ell}}  
$$

其中：

- (h(p))：desired 信道在位置 (p) 的复响应
    
- (g(p))：jammer 信道在位置 (p) 的复响应
    
- (\alpha_\ell,\beta_\ell)：路径复增益
    
- (\theta_{s,\ell},\theta_{j,\ell})：路径 AoA
    
- (p)：可移动天线候选位置
    

### 关键理解

(h(p))、(g(p)) 是**信道条件**，不是实际接收信号功率。

实际接收功率需要乘发射功率：

$$  
P_{\text{desired,rx}}(p)=P_s|h(p)|^2  
$$

$$  
P_{\text{jammer,rx}}(p)=P_j|g(p)|^2  
$$

---

## 1.2 主问题：(H) 和 (G) 是什么？

### ✅ 已解决

代码中：

```matlab
H = abs(h_grid).^2;
G = abs(g_grid).^2;

H = H/(max(H) + eps);
G = G/(max(G) + eps);
```

对应：

$$  
H(p)=\frac{|h(p)|^2}{\max_{q\in\mathcal P}|h(q)|^2}  
$$

$$  
G(p)=\frac{|g(p)|^2}{\max_{q\in\mathcal P}|g(q)|^2}  
$$

### 物理意义

- (H(p))：desired 信道增益地图
    
- (G(p))：jammer 信道增益地图
    
- (1-G(p))：jammer 弱度地图
    

### 关键结论

(H,G) 反映的是**归一化信道强弱**，不显式包含 $P_s,P_j,\sigma^2$。

---

## 1.3 子问题：$\alpha,\beta$是不是 Rayleigh 信道？

### ✅ 已解决

代码中：

```matlab
alpha = (randn(Ls, 1) + 1j*randn(Ls, 1))/sqrt(2);
beta  = (randn(Lj, 1) + 1j*randn(Lj, 1))/sqrt(2);
```

表示：

$$  
\alpha_\ell,\beta_\ell \sim \mathcal{CN}(0,1)  
$$

所以单条路径幅度服从 Rayleigh 分布。

但如果后面有：

```matlab
alpha = alpha/norm(alpha);
beta  = beta/norm(beta);
```

则路径总能量被归一化，避免 Monte Carlo 中某次信道整体能量异常大。

### 关键结论

> 路径复增益是 Rayleigh fading 形式；但整个空间信道 profile 不是每个网格点独立 Rayleigh，而是有限多径叠加形成的空间相关信道。

---

# 2. FADLS 的两阶段选择逻辑

## 2.1 主问题：FADLS 是不是直接优化 SINR？

### ✅ 已解决

不是。

FADLS 不直接优化：

$$  
\Gamma_{\mathrm{MMSE}}(\mathcal S)  
$$

而是优化一个**物理启发式信道结构代理目标**。

FADLS 关注：

1. desired 信道是否强
    
2. jammer 信道是否弱
    
3. selected desired vector 和 jammer vector 是否低相关
    

---

## 2.2 子问题：为什么先用 (H(p)(1-G(p)))？

### ✅ 已解决

第一阶段是 candidate pool 预筛选。

代码：

```matlab
score_naive = H .* (1 - G);
[~, idx_rank_dual] = sort(score_naive, 'descend');
idx_candidate_pool = idx_rank_dual(1:candidate_pool_size);
```

公式：

$$  
s_{\text{pre}}(p)=H(p)(1-G(p))  
$$

### 作用

从全体候选位置：

$$  
\mathcal P={p_1,p_2,\ldots,p_M}  
$$

中筛出前 (C) 个位置，形成候选池：

$$  
\mathcal C

\text{Top-}C{H(p)(1-G(p)),p\in\mathcal P}  
$$

### 关键结论

(H(p)(1-G(p))) 是**单点预筛选分数**，只算一次，不随天线数量变化。

---

## 2.3 子问题：为什么用 (1-G(p))，不是最小化 (G(p))？

### ✅ 已解决

因为选择函数是“最大化 score”。

$$  
\max(1-G(p)) \Longleftrightarrow \min G(p)  
$$

前提是 (G(p)\in[0,1])。

### 直觉

- (G(p)) 大：jammer 强，不好
    
- (1-G(p)) 大：jammer 弱，好
    

所以 (1-G(p)) 是 jammer-weakness score。

---

# 3. FADLS 贪心选择依据

## 3.1 主问题：一开始没有天线，FADLS 怎么选？

### ✅ 已解决

FADLS 从空集合开始：

$$  
\mathcal S_0=\varnothing  
$$

然后一根一根加：

$$  
\mathcal S_1={p_1}  
$$

$$  
\mathcal S_2={p_1,p_2}  
$$

一直到：

$$  
\mathcal S_N={p_1,\ldots,p_N}  
$$

第 (n) 根天线选择：

$$  
p_n^\star

\arg\max_{p\in\mathcal C}  
s_{\mathrm{FADLS}}(\mathcal S_{n-1}\cup{p})  
$$

同时满足：

$$  
|p-p_i|\ge d_{\min},\quad \forall p_i\in\mathcal S_{n-1}  
$$

---

## 3.2 子问题：(H(\mathcal S_{\text{tmp}})) 是什么尺寸？

### ✅ 已解决

全局 (H) 是 (M\times1) 向量：

$$  
H=[H(p_1),H(p_2),\ldots,H(p_M)]^T  
$$

但：

$$  
H(\mathcal S_{\text{tmp}})  
$$

是从全局 (H) 中取出当前临时天线集合对应的子向量。

尺寸随天线数量增长：

```text
第1根：1×1
第2根：2×1
第3根：3×1
...
第N根：N×1
```

### 关键区别

- `score_naive = H.*(1-G)`：全网格 (M\times1)，只算一次
    
- `H(idx_tmp)`：当前临时组合的子向量，长度从 1 到 (N) 增长
    

---

## 3.3 子问题：为什么要用 (\bar H_{\mathcal S})、(\bar G_{\mathcal S})？

### ✅ 已解决

因为 (H(\mathcal S))、(G(\mathcal S)) 是向量，最终 score 需要标量。

所以定义：

$$  
\bar H_{\mathcal S}

\frac{1}{|\mathcal S|}  
\sum_{p_i\in\mathcal S}H(p_i)  
$$

$$  
\bar G_{\mathcal S}

\frac{1}{|\mathcal S|}  
\sum_{p_i\in\mathcal S}G(p_i)  
$$

推荐符号：

$$  
x_H(\mathcal S)=\bar H_{\mathcal S}  
$$

$$  
x_G(\mathcal S)=1-\bar G_{\mathcal S}  
$$

$$  
x_\rho(\mathcal S)=1-\rho_{hg}(\mathcal S)  
$$

---

## 3.4 子问题：(\rho) 是全网格相关性吗？

### ✅ 已解决

不是。

严格应写：

$$  
\rho_{hg}(\mathcal S)  
$$

表示**当前选定天线集合 (\mathcal S) 上** desired 信道向量和 jammer 信道向量的相关性。

定义：

$$  
\rho_{hg}(\mathcal S)

\frac{  
|\mathbf h_{\mathcal S}^H\mathbf g_{\mathcal S}|^2  
}{  
|\mathbf h_{\mathcal S}|^2|\mathbf g_{\mathcal S}|^2+\epsilon  
}  
$$

其中：

$$  
\mathbf h_{\mathcal S}=[h(p_i)]_{p_i\in\mathcal S}  
$$

$$  
\mathbf g_{\mathcal S}=[g(p_i)]_{p_i\in\mathcal S}  
$$

### 关键结论

(H,G) 是全局地图；但 (\rho_{hg}(\mathcal S)) 必须在当前 selected set 上计算。

---

# 4. FADLS score 与 log / 乘法逻辑

## 4.1 主问题：完整 FADLS score 是什么？

### ✅ 已解决

推荐写成：

$$  
s_{\mathrm{FADLS}}(\mathcal S)

\omega_H\log\left(\bar H_{\mathcal S}+\epsilon\right)  
+  
\omega_G\log\left(1-\bar G_{\mathcal S}+\epsilon\right)  
+  
\omega_\rho\log\left(1-\rho_{hg}(\mathcal S)+\epsilon\right)  
$$

或者：

$$  
s_{\mathrm{FADLS}}(\mathcal S)

\omega_H\log(x_H(\mathcal S)+\epsilon)  
+  
\omega_G\log(x_G(\mathcal S)+\epsilon)  
+  
\omega_\rho\log(x_\rho(\mathcal S)+\epsilon)  
$$

---

## 4.2 子问题：为什么用乘法型 utility？

### ✅ 已解决

三个指标：

$$  
x_H=\bar H_{\mathcal S}  
$$

$$  
x_G=1-\bar G_{\mathcal S}  
$$

$$  
x_\rho=1-\rho_{hg}(\mathcal S)  
$$

都是越大越好。

如果用加法：

$$  
s_{\text{add}}=\omega_Hx_H+\omega_Gx_G+\omega_\rho x_\rho  
$$

这是“加分制”，某一项差可以被其他项补回来。

如果用乘法：

$$  
s_{\text{mul}}=x_Hx_Gx_\rho  
$$

这是“闯关制”，任何一项很差都会压低整体分数。

### 逻辑直觉

FADLS 需要：

```text
desired 强 AND jammer 弱 AND h/g 不相关
```

不是：

```text
desired 强 OR jammer 弱 OR h/g 不相关
```

所以乘法更符合抗干扰的多条件联合逻辑。

---

## 4.3 子问题：为什么乘法里的权重要放指数上？

### ✅ 已解决

乘法加权写作：

$$  
s=x_H^{\omega_H}x_G^{\omega_G}x_\rho^{\omega_\rho}  
$$

取 log：


$$  
\log s

\omega_H\log x_H  
+  
\omega_G\log x_G  
+  
\omega_\rho\log x_\rho  
$$

所以：

> 乘法域里的指数权重 = log 域里的线性权重。

### 关键记忆

- 加法模型：权重乘在项前面
    
- 乘法模型：权重放在指数上
    
- log 以后：乘法模型变成加权和
    

---

## 4.4 子问题：为什么代码里直接用 log score？

### ✅ 已解决

代码里：

```matlab
score_tmp = omega_H*log(mean(H(idx_tmp)) + eps) ...
          + omega_G*log(1 - mean(G(idx_tmp)) + eps) ...
          + omega_rho*log(1 - rho_hg + eps);
```

本质上等价于最大化：

$$  
x_H^{\omega_H}x_G^{\omega_G}x_\rho^{\omega_\rho}  
$$

用 log 的原因：

1. 把乘法型目标变成加法型目标，方便计算
    
2. 避免直接相乘导致数值过小
    
3. 对短板进行惩罚
    
4. 方便调权重
    

---

# 5. SINR-greedy 的搜索范围与评价函数

## 5.1 主问题：SINR-greedy 是否使用 FADLS candidate pool？

### ✅ 已解决

不使用。

代码：

```matlab
for idx_try = 1:num_grid
```

说明它每轮扫描全体候选网格：

$$  
\mathcal P={p_1,p_2,\ldots,p_M}  
$$

不是扫描 FADLS 的候选池 (\mathcal C)。

---

## 5.2 子问题：(\Gamma_{\mathrm{MMSE}}(\mathcal S)) 里的 (\mathcal S) 是全体坐标吗？

### ✅ 已解决

不是。

(\mathcal P) 是搜索域，全体可选坐标。

(\mathcal S) 是当前选出的临时阵列集合。

第 (n) 根选择时：

$$  
p_n^\star

\arg\max_{  
p\in\mathcal P,;  
p\notin\mathcal S_{n-1},;  
|p-p_i|\ge d_{\min}  
}  
\Gamma_{\mathrm{MMSE}}  
\left(  
\mathcal S_{n-1}\cup{p}  
\right)  
$$

其中：

$$  
\mathcal S_{\text{tmp}}=\mathcal S_{n-1}\cup{p}  
$$

才是参与 SINR 计算的当前临时阵列。

---

## 5.3 子问题：SINR-greedy 为什么按道理更强？

### ✅ 已解决

因为 SINR-greedy 的评价函数更贴近最终性能：

$$  
\Gamma_{\mathrm{MMSE}}(\mathcal S)

P_s\mathbf h_{\mathcal S}^H  
\left(  
P_j\mathbf g_{\mathcal S}\mathbf g_{\mathcal S}^H  
+  
\sigma^2\mathbf I  
\right)^{-1}  
\mathbf h_{\mathcal S}  
$$

它显式考虑：

- desired 信道 (\mathbf h_{\mathcal S})
    
- jammer 信道 (\mathbf g_{\mathcal S})
    
- desired 功率 (P_s)
    
- jammer 功率 (P_j)
    
- 噪声功率 (\sigma^2)
    
- MMSE 接收结构
    

### 但它不一定总是最优

因为它仍然是 greedy：

$$  
\text{当前一步最优} \neq \text{最终 }N\text{ 根组合全局最优}  
$$

---

# 6. MMSE 输出 SINR 推导

## 6.1 主问题：分母为什么可以写成二次型？

### ✅ 已解决

原分母：

$$  
P_j|\mathbf w^H\mathbf g|^2+\sigma^2|\mathbf w|^2  
$$

先看第一项：

$$  
|\mathbf w^H\mathbf g|^2

# (\mathbf w^H\mathbf g)(\mathbf g^H\mathbf w)

\mathbf w^H\mathbf g\mathbf g^H\mathbf w  
$$

再看噪声项：

$$  
|\mathbf w|^2=\mathbf w^H\mathbf w=\mathbf w^H\mathbf I\mathbf w  
$$

所以：

$$  
P_j|\mathbf w^H\mathbf g|^2+\sigma^2|\mathbf w|^2

P_j\mathbf w^H\mathbf g\mathbf g^H\mathbf w  
+  
\sigma^2\mathbf w^H\mathbf I\mathbf w  
$$

合并：

$$

\mathbf w^H  
\left(  
P_j\mathbf g\mathbf g^H+\sigma^2\mathbf I  
\right)  
\mathbf w  
$$

定义：

$$  
\mathbf R=P_j\mathbf g\mathbf g^H+\sigma^2\mathbf I  
$$

则分母为：

$$  
\mathbf w^H\mathbf R\mathbf w  
$$

---

## 6.2 子问题：为什么可以把中间项提出来？

### ✅ 已解决

因为两个项都有共同外壳：

$$  
\mathbf w^H(\cdot)\mathbf w  
$$

令：

$$  
\mathbf A=P_j\mathbf g\mathbf g^H  
$$

$$  
\mathbf B=\sigma^2\mathbf I  
$$

则：

$$  
\mathbf w^H\mathbf A\mathbf w+\mathbf w^H\mathbf B\mathbf w

\mathbf w^H(\mathbf A+\mathbf B)\mathbf w  
$$

---

## 6.3 主问题：为什么最优接收权重是 (\mathbf R^{-1}\mathbf h)？

### ✅ 已解决，但可继续复习

输出 SINR：

$$  
\Gamma(\mathbf w)

\frac{  
P_s|\mathbf w^H\mathbf h|^2  
}{  
\mathbf w^H\mathbf R\mathbf w  
}  
$$

忽略常数 (P_s)，最大化：

$$  
\frac{  
|\mathbf w^H\mathbf h|^2  
}{  
\mathbf w^H\mathbf R\mathbf w  
}  
$$

用 Cauchy-Schwarz 推导可得：

$$  
\mathbf w_{\mathrm{opt}}\propto \mathbf R^{-1}\mathbf h  
$$

最大输出 SINR：

$$  
\Gamma_{\mathrm{out}}

P_s\mathbf h^H\mathbf R^{-1}\mathbf h  
$$

代码对应：

```matlab
R = Pj*(g_vec*g_vec') + sigma2*eye(length(idx_sel));
sinr_linear = real(Ps*(h_vec'*(R\h_vec)));
```

---

# 7. FADLS vs SINR-greedy 的定位

## 7.1 主问题：两者本质区别是什么？

### ✅ 已解决

|方法|搜索域|评价函数|是否显式使用 (P_j,\sigma^2)|定位|
|---|---|---|---|---|
|FADLS|候选池 (\mathcal C)|信道结构 surrogate|否|低复杂度物理启发式|
|SINR-greedy|全网格 (\mathcal P)|post-MMSE SINR|是|强性能基准，但 greedy|
|Exhaustive optimum|全组合|post-MMSE SINR|是|小规模离散全局最优天花板|

---

## 7.2 子问题：FADLS 的卖点是什么？

### ✅ 已解决

不是“评价函数更完整”。

FADLS 的卖点是：

> 用低复杂度的信道结构指标，快速逼近高 SINR 区域。

推荐表述：

> FADLS uses a physics-guided channel-structure surrogate to reduce the search effort while maintaining a favorable performance-complexity tradeoff.

---

# 8. 复杂度与图的问题

## 8.1 主问题：complexity proxy 图为什么看起来太惨烈？

### ✅ 已解决

因为横轴用了阶数加权 complexity proxy：

- FADLS：近似 (M\log M + CN^2)
    
- SINR-greedy：direct matrix solve 下近似 (MN^4)
    

差距会被视觉放大，蓝线贴在 y 轴附近，容易显得不真实。

### 建议

主文中优先用：

1. candidate checks tradeoff
    
2. receiver-level SINR evaluations
    
3. 理论复杂度表格
    

complexity proxy 图可改成 log-scale 或 normalized 版本。

---

## 8.2 子问题：small-scale exhaustive benchmark 图为什么奇怪？

### ✅ 已解决

原图扫 JSR 后，出现：

- FADLS 曲线几乎不变
    
- SINR-greedy 随 JSR 增大下降
    
- SINR-greedy 在低 JSR 更接近 optimum
    

解释：

1. FADLS 选点不显式依赖 JSR
    
2. SINR-greedy 每个 JSR 会重新选点
    
3. SINR-greedy 是局部贪心，不是全局最优
    
4. 高 JSR 下 greedy 短视性可能更明显
    

### 建议

exp10 最好定位为：

> small-scale sanity check against discrete-grid exhaustive optimum

主图保留：

1. SINR versus exhaustive ceiling
    
2. gap to optimum
    
3. gap CDF at representative JSR
    

---

# 9. 已解决问题清单

-  (h(p),g(p)) 是信道，不是信号功率
    
-  (H,G) 是归一化信道增益地图
    
-  (\alpha,\beta) 是路径复增益，Rayleigh fading 形式
    
-  (H(p)(1-G(p))) 是 candidate pool 单点预筛选分数
    
-  (1-G(p)) 等价于最小化 (G(p))
    
-  FADLS 从空集合开始，一根根贪心加天线
    
-  (H(\mathcal S))、(G(\mathcal S)) 是当前 selected set 上的子向量
    
-  (\bar H_{\mathcal S})、(\bar G_{\mathcal S}) 是 selected set 上的平均标量
    
-  (\rho_{hg}(\mathcal S)) 是 selected set 上的信道向量相关性
    
-  log score 等价于加权乘法 utility
    
-  乘法型 score 表示 AND / 闯关逻辑
    
-  指数权重是乘法域的权重
    
-  SINR-greedy 不使用 candidate pool，而是全网格搜索
    
-  (\Gamma_{\mathrm{MMSE}}(\mathcal S)) 中的 (\mathcal S) 是当前临时阵列，不是全体坐标
    
-  MMSE 分母可以写成二次型
    
-  最优接收权重为 (\mathbf R^{-1}\mathbf h)
    
-  最大输出 SINR 为 (P_s\mathbf h^H\mathbf R^{-1}\mathbf h)
    

---

# 10. 未解决 / 待继续问题

## 10.1 理论层面

-  是否需要给 FADLS score 一个更正式的 surrogate optimization 解释？
    
-  (\omega_H,\omega_G,\omega_\rho) 的取值是否需要敏感性分析？
    
-  是否需要证明 FADLS score 与 high-SINR condition 的近似关系？
    
-  单 jammer rank-one 情况下，是否应在论文中说明 SINR-greedy 可以用 Sherman-Morrison 加速？
    

## 10.2 实验层面

-  exp10 是否固定一个代表性 JSR，而不是扫 JSR？
    
-  complexity proxy 图是否改成 log-scale 或 normalized 版本？
    
-  candidate pool size (C) 是否需要做敏感性实验？
    
-  是否需要把 SINR-greedy 明确命名为 full-grid SINR-greedy，而不是 SINR-optimal？
    
-  exhaustive benchmark 是否只放主文一张三联图，其他统计放 appendix？
    

## 10.3 写作层面

-  FADLS 方法章节中应如何组织：
    
    1. channel profile
        
    2. dual-favorable screening
        
    3. greedy weighted-log score
        
    4. complexity analysis
        
    5. relation to SINR-greedy benchmark
        
-  是否需要在论文中避免使用 (\alpha,\beta) 表示 FADLS 权重，因为它们已用于路径增益？
    
-  推荐将权重写成：  
    $$  
    \omega_H,\omega_G,\omega_\rho  
    $$  
    而不是：  
    $$  
    \alpha,\beta,\gamma  
    $$
    

---

# 11. 最核心的理解

## 11.1 FADLS 一句话

> FADLS 不直接计算接收端 SINR，而是用 selected set 上的 desired 平均信道增益、jammer 平均信道弱度、desired/jammer 信道向量去相关性构造一个低复杂度物理启发式 score，一根根贪心选择天线位置。

## 11.2 SINR-greedy 一句话

> SINR-greedy 每一步在全网格中尝试所有可行位置，选择使当前临时阵列 post-MMSE 输出 SINR 最大的位置；它是强性能基准，但仍是局部贪心，不是全局最优。

## 11.3 Exhaustive optimum 一句话

> Exhaustive optimum 枚举小规模离散网格下所有满足最小间距约束的组合，是该小规模离散问题的全局最优天花板。

## 11.4 最重要的对比

```text
FADLS：低复杂度、物理启发式、信道结构驱动
SINR-greedy：高复杂度、receiver-level SINR 驱动、局部贪心
Exhaustive：最高复杂度、小规模离散全局最优
```

---

# 12. 推荐后续推导顺序

1. # 重新手推 MMSE 输出 SINR：  
    $$  
    \Gamma_{\mathrm{out}}
    
    P_s\mathbf h^H\mathbf R^{-1}\mathbf h  
    $$
    
2. # 写清楚 FADLS score：  
    $$  
    s_{\mathrm{FADLS}}(\mathcal S)
    
    \omega_H\log(\bar H_{\mathcal S}+\epsilon)  
    +  
    \omega_G\log(1-\bar G_{\mathcal S}+\epsilon)  
    +  
    \omega_\rho\log(1-\rho_{hg}(\mathcal S)+\epsilon)  
    $$
    
3. # 对比 SINR-greedy：  
    $$  
    p_n^\star
    
    \arg\max_{p\in\mathcal P}  
    \Gamma_{\mathrm{MMSE}}(\mathcal S_{n-1}\cup{p})  
    $$
    
4. 写复杂度：  
    $$  
    \text{FADLS}: O(M\log M+CN^2)  
    $$
    
    $$  
    \text{SINR-greedy}: O(MN^4)  
    $$
    
5. 最后再整理实验图：
    
    - exp7: SINR vs paths
        
    - exp8: SINR vs JSR
        
    - exp9: performance-search tradeoff
        
    - exp10: small-scale exhaustive ceiling