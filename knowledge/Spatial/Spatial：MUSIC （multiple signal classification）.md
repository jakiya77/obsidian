---
tags:
  - 空域
---
MUSIC算法的原理
---
分母确实是在求**当前搜索角度的导向矢量（Steering Vector）在噪声子空间上投影的模的平方**。你的推导完全正确：

$||a^H(\theta)U_{null}||^2=a^H(\theta)U_{null}(a^H(\theta)U_{null})^H=a^H(\theta)U_{null}U_{null}^Ha(\theta)$

当搜索角度 $\theta$ 正好等于真实信号的到达角（DOA）时，导向矢量会与噪声子空间正交，投影接近于 $0$（即分母极小），取倒数后就会在空间谱上形成一个极大的尖峰。

为了更系统地帮你梳理，以下是这个公式的完整推导过程和物理原理：

### 1. 物理原理：子空间正交性

MUSIC 算法的根基是**信号子空间与噪声子空间的正交性**。

阵列接收到的数据中包含信号和噪声。通过对接收信号的协方差矩阵进行特征值分解，可以将整个空间划分为两个相互正交的部分：

- **信号子空间**：由真实信号的导向矢量张成的空间。
    
- **噪声子空间**：与信号子空间完全正交的空间。
    

因为真实信号的导向矢量必然存在于“信号子空间”中，所以**真实信号的导向矢量必然与“噪声子空间”正交**。



### 2. 严谨的数学推导

**第一步：建立信号模型与协方差矩阵**

假设有 $K$ 个互不相干的信号入射到 $M$ 个阵元的阵列上，接收信号模型为：

$$x(t)=As(t)+n(t)$$

其中，$A=[a(\theta_1),a(\theta_2),...,a(\theta_K)]$ 是阵列流型矩阵（包含了所有真实信号的导向矢量）。

接收信号的理想协方差矩阵 $R_x$ 为：

$$R_x=E[x(t)x^H(t)]=AR_sA^H+\sigma^2I$$

其中，$R_s$ 是信号的协方差矩阵，$\sigma^2$ 是噪声功率，$I$ 是单位阵。

**第二步：特征值分解 (EVD)**

对协方差矩阵 $R_x$ 进行特征值分解：

$$R_x=U_s\Sigma_sU_s^H+U_{null}\Sigma_nU_{null}^H$$

- $U_s$ 是由 $K$ 个最大特征值对应的特征向量组成的矩阵，张成**信号子空间**。
    
- $U_{null}$ 是由剩下的 $M-K$ 个较小特征值（等于噪声方差 $\sigma^2$）对应的特征向量组成的矩阵，张成**噪声子空间**。
    

**第三步：利用正交性构建空间谱**

理论上，信号子空间和噪声子空间正交，即 $U_s \perp U_{null}$。

由于真实的导向矢量 $a(\theta_k)$ 是信号子空间 $U_s$ 的线性组合，因此真实的导向矢量也必须与噪声子空间正交：

$$a^H(\theta_k)U_{null}=0\quad(k=1,2,...,K)$$

**第四步：谱函数寻优**

为了找出这些真实的角度 $\theta_k$，我们在空间中遍历所有可能的角度 $\theta$。对于任意角度 $\theta$，我们计算它的导向矢量 $a(\theta)$ 在噪声子空间 $U_{null}$ 上的投影：

$$D(\theta)=||a^H(\theta)U_{null}||^2=a^H(\theta)U_{null}U_{null}^Ha(\theta)$$

- 如果 $\theta$ 是真实信号方向，$D(\theta) \approx 0$。
    
- 如果 $\theta$ 不是真实信号方向，$D(\theta) > 0$。
    

为了让人类更直观地观察到“峰值”而不是“极小值”，MUSIC 算法将被投影的范数取倒数，这就得到了你看到的伪谱（Pseudospectrum）公式：

$$P_{MUSIC}(\theta)=\frac{1}{a^H(\theta)U_{null}U_{null}^Ha(\theta)}$$

---

MUSIC算法的局限性 - 针对多径（相干信源）失效
---

在现实场景中（尤其是城市或室内通信），信号到达接收天线时往往不仅有直达波，还有经过墙壁、建筑物反射后的多径信号。这些多径信号本质上是同一个源信号的延迟和衰减版本，它们在数学上被称为**相干信源（Coherent Sources）**。

当信源完全相干时，假设有两个信号 $s_1(t)$ 和 $s_2(t)$，它们满足线性关系：

$$s_2(t)=\alpha s_1(t)$$

其中 $\alpha$ 是复衰减系数。

**致命后果：信号协方差矩阵秩亏（Rank Deficiency）**

此时，信号的协方差矩阵 $R_s$ 会发生变化：

$$R_s=E[s(t)s^H(t)]=E\left[\begin{pmatrix} s_1(t) \\ \alpha s_1(t) \end{pmatrix} \begin{pmatrix} s_1^*(t) & \alpha^* s_1^*(t) \end{pmatrix}\right]=E[|s_1(t)|^2]\begin{pmatrix} 1 & \alpha^* \\ \alpha & |\alpha|^2 \end{pmatrix}$$

这个 $2 \times 2$ 的矩阵 $R_s$ 的行列式为 $0$，它的**秩（Rank）变成了 1**，而不是理想情况下的 2（即互不相干的信号个数 $K$）。[[Math：奇异 == 行列式值为0 == 行线性相关 == 列线性相关 == 不可逆 == 秩亏]]

**对空间谱的破坏：**

由于 $R_s$ 秩亏减小，接收数据协方差矩阵 $R_x$ 进行特征值分解后：

- **信号子空间的维度变小了**（本来应该有 $K$ 个大特征值，现在只有 $1$ 个）。
    
- **噪声子空间的维度变大了**，它“侵占”了原本属于信号子空间的部分。
    

结果就是，真实的导向矢量 $a(\theta_1)$ 和 $a(\theta_2)$ 不再完全包含于残缺的信号子空间中。因此，它们与扩大后的噪声子空间 $U_{null}$ **不再正交**。

$$a^H(\theta_k)U_{null}\neq0$$

分母不再趋近于 0，MUSIC 谱上的尖峰就会消失，导致无法分辨出这两个相干信号的角度。

---

### 2. 空间平滑技术（Spatial Smoothing）的破局原理

为了解决秩亏问题，我们需要一种方法来打乱相干信号之间的固定相位关系，使其解相关。这就是空间平滑技术的核心思想。

**核心操作：子阵划分与平均**

空间平滑通常针对均匀线阵（ULA）。假设我们有一个包含 $M$ 个阵元的天线阵列：

1. **划分子阵**：将这 $M$ 个阵元划分成 $L$ 个相互重叠的子阵。每个子阵包含 $m$ 个阵元。
    
    - 子阵 1：包含阵元 $1, 2, ..., m$
        
    - 子阵 2：包含阵元 $2, 3, ..., m+1$
        
    - ...
        
    - 子阵 L：包含阵元 $L, L+1, ..., M$ (此时 $M = m + L - 1$)
        
2. **求各子阵的协方差矩阵**：计算每个子阵的接收数据协方差矩阵 $R_1, R_2, ..., R_L$。
    
3. **空间平均**：将所有子阵的协方差矩阵加起来求平均，得到平滑后的协方差矩阵 $R_{smooth}$：
    
    $$\bar{R}=\frac{1}{L}\sum_{i=1}^L R_i$$
    

**为什么这样能解相关？**

不同子阵在空间上有一个物理位移，这个位移会给导向矢量带来一个额外的相位偏转。当把这些带有不同相位偏转的协方差矩阵加在一起时，相干信号之间的交叉互相关项会被平均掉（或者说发生破坏性干涉）。

经过这样的数学处理，平滑后的协方差矩阵 $\bar{R}$ 的秩被奇迹般地**恢复到了满秩 $K$**。

此时，再对 $\bar{R}$ 进行特征值分解，应用 MUSIC 算法，就能重新找到正确的正交噪声子空间，从而准确测向。[[]]

---

### 3. 空间平滑的代价

空间平滑是以**牺牲阵列孔径（Array Aperture）**（即自由度）为代价的。

- 原本 $M$ 个阵元的阵列，最多可以分辨 $M-1$ 个独立信号。
    
- 使用空间平滑后，我们实际使用的是只有 $m$ 个阵元的子阵矩阵 $\bar{R}$。因此，它最多只能分辨 $m-1$ 个信号。为了能够解相关 $K$ 个相干信号，子阵的数量 $L$ 必须大于等于 $K$。
    

这就意味着，同样的硬件天线数量，使用空间平滑后能同时测向的信号数量变少了，空间分辨率也会有所下降。

代码实现
---
```matlab
  

clear; clc; close all;

  

lambda = 1;

d = lambda / 2;

N = 8;

K = 200;

k = 2 * pi / lambda;

L = 3;

signal_angle = [-20, 10, 40];

sv = @(theta, num_ant) exp(1i * k * d * (0:num_ant-1)' * sind(theta));

  

  

s_source = (randn(K,1) + 1i*randn(K,1)) / sqrt(2);

  

  

% alpha = [1; 0.8*exp(1i*pi/4); 0.6*exp(-1i*pi/3)];

alpha_signal = (randn(L,1)+1i*randn(L,1))/sqrt(2);

  

h_signal = zeros(N,1);

for l = 1:L

h_signal = h_signal + alpha_signal(l) * sv(signal_angle(l), N);

end

  

noise = 1 * (randn(N,K) + 1i*randn(N,K)) / sqrt(2);

Y = h_signal * s_source.' + noise;

  

  

R_y = (Y * Y') / K;

  

theta_scan = -90:0.1:90;

  

% 算法 A: 标准 MUSIC

  

[U_std, ~, ~] = svd(R_y);

  

Un_std = U_std(:, L+1:N); % 直接提取除了信号外的其余子空间作为正交空间

  

P_std = zeros(size(theta_scan));

for i = 1:length(theta_scan)

a = sv(theta_scan(i), N);

P_std(i) = 1 / abs(a' * Un_std * Un_std' * a);

end

P_std_dB = 10 * log10(P_std / max(P_std)); % 归一化转 dB

  

  

% 算法 B: 前向空间平滑 MUSIC

  

m = 6; % 子阵阵元数 (m > L)设计的 子阵列的个数要大于和干扰个数-1

P = N - m + 1; % 子阵个数

R_smooth = zeros(m, m);

  

for p = 1:P

R_smooth = R_smooth + R_y(p : p+m-1, p : p+m-1);

end

R_smooth = R_smooth / P; % 平均恢复满秩

  

[U_sm, ~, ~] = svd(R_smooth);

Un_sm = U_sm(:, L+1:m); % 维度变成了 m

  

P_sm = zeros(size(theta_scan));

for i = 1:length(theta_scan)

a = sv(theta_scan(i), m); % 注意导向矢量维度也变成了 m

P_sm(i) = 1 / abs(a' * Un_sm * Un_sm' * a);

end

P_sm_dB = 10 * log10(P_sm / max(P_sm));

  


% 绘图对比


figure('Color', 'w');

plot(theta_scan, P_std_dB, 'k-.', 'LineWidth', 1.5); hold on;

plot(theta_scan, P_sm_dB, 'b-', 'LineWidth', 2);

  

% 画出真实的信号角度参考线

for i = 1:L

xline(signal_angle(i), 'r--', 'LineWidth', 1.5);

end

  

grid on;

xlabel('Angle (degrees)');

ylabel('Spatial Spectrum (dB)');

title('MUSIC vs Spatial Smoothing MUSIC (Coherent Sources)');

legend('Standard MUSIC (Failed)', 'Spatial Smoothing MUSIC', 'True Angles');
```
仿真结果
---

| ![[Pasted image 20260309211237.png]] | ![[Pasted image 20260309211355.png]] |     |
| ------------------------------------ | ------------------------------------ | --- |

