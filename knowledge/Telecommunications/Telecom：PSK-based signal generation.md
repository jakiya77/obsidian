
## 1. QPSK，Quadrature Phase Shift Keying

### 一、 数学原理

QPSK 信号在时间域上的标准数学表达式为：

$$s_i(t) = \sqrt{\frac{2E_s}{T_s}} \cos(2\pi f_c t + \theta_i)$$

其中：

- $E_s$ 是每个符号的能量。
    
- $T_s$ 是符号周期。
    
- $f_c$ 是载波频率。
    
- $\theta_i$ 是调制相位。通常采用的四个相位为 $\theta_i \in \{\frac{\pi}{4}, \frac{3\pi}{4}, \frac{5\pi}{4}, \frac{7\pi}{4}\}$。
    

利用三角函数展开式 $\cos(A + B) = \cos A \cos B - \sin A \sin B$，上述公式可以重写为同相（In-phase, I）和正交（Quadrature, Q）两个相互正交的载波的线性组合：

$$s_i(t) = \sqrt{\frac{E_s}{T_s}} I \cos(2\pi f_c t) - \sqrt{\frac{E_s}{T_s}} Q \sin(2\pi f_c t)$$

这里，$I$ 和 $Q$ 分别代表映射后的双极性基带电平，其取值为 $\{+1, -1\}$ 或 $\{+\frac{1}{\sqrt{2}}, -\frac{1}{\sqrt{2}}\}$。

由于每个 QPSK 符号代表 2 个比特（例如 $b_0$ 和 $b_1$），我们通常使用格雷码（Gray Code）映射来降低误比特率：

- $I$ 分量由偶数位比特决定。
    
- $Q$ 分量由奇数位比特决定。
    

这种正交结构使得我们可以将 QPSK 视为两个相互独立的 BPSK（二进制相移键控）信号的叠加，一个在余弦载波上，另一个在正弦载波上。

---

### 二、 MATLAB 代码实现

下面的 MATLAB 代码展示了从随机比特生成、基带映射到频带调制（乘以高频载波）的完整过程。代码不依赖于 Communications Toolbox 的黑盒函数，以便于对应数学原理进行理解。


``` matlab
% QPSK 信号生成仿真
clear; clc; close all;

%% 1. 参数设置
N = 1000;            % 符号数量
fc = 1000;           % 载波频率 (Hz)
fs = 10000;          % 采样频率 (Hz) (需满足 Nyquist 定理 fs > 2*fc)
Ts = 1/100;          % 符号周期 (s)
t = 0:1/fs:Ts-1/fs;  % 单个符号的时间向量
Ns = length(t);      % 单个符号的采样点数

%% 2. 随机比特生成
% QPSK 每个符号携带 2 个 bit，因此总 bit 数为 2*N
bits = randi([0 1], 1, 2*N); 

% 将比特流分为 I 路和 Q 路 (串并转换)
bits_I = bits(1:2:end); 
bits_Q = bits(2:2:end); 

%% 3. 基带符号映射 
(采用非归零极性映射: 0 -> -1/sqrt(2), 1 -> 1/sqrt(2))
% 加上 1/sqrt(2) 是为了保证符号总能量为 1
I = (2 * bits_I - 1) / sqrt(2); 
Q = (2 * bits_Q - 1) / sqrt(2); 

% 生成复基带信号 (星座图上的点)
baseband_sym = I + 1i * Q;

%% 4. 频带信号生成 (乘以载波)
passband_signal = zeros(1, N * Ns);
time_vector = 0:1/fs:(N*Ts)-1/fs;

for k = 1:N
    % 生成 I 路和 Q 路的载波
    carrier_I = cos(2 * pi * fc * t);
    carrier_Q = -sin(2 * pi * fc * t); % 注意这里的负号对应数学推导
    
    % 调制单个符号
    symbol_waveform = I(k) * carrier_I + Q(k) * carrier_Q;
    
    % 将当前符号波形拼接到总信号中
    passband_signal((k-1)*Ns + 1 : k*Ns) = symbol_waveform;
end

%% 5. 结果可视化
% (a) 绘制基带星座图
figure;
scatter(real(baseband_sym), imag(baseband_sym), 50, 'filled');
title('QPSK 基带星座图');
xlabel('同相分量 (I)');
ylabel('正交分量 (Q)');
grid on; axis square;
axis([-1.5 1.5 -1.5 1.5]);

% (b) 绘制前 5 个符号的频带波形
num_plot_sym = 5;
figure;
plot(time_vector(1:num_plot_sym*Ns), passband_signal(1:num_plot_sym*Ns), 'LineWidth', 1.5);
title('QPSK 频带信号时域波形 (前 5 个符号)');
xlabel('时间 (s)');
ylabel('幅度');
grid on;
```

### 代码逻辑说明：

1. **参数设置**：定义了符号率、载波频率 `fc` 和采样率 `fs`。
    
2. **串并转换**：将生成的二进制数据流分为偶数位和奇数位，分别对应 I 支路和 Q 支路。
    
3. **极性映射**：将逻辑 `0` 映射为 $-\frac{1}{\sqrt{2}}$，逻辑 `1` 映射为 $+\frac{1}{\sqrt{2}}$，从而构建复数基带符号。
    
4. **载波调制**：使用 `for` 循环为每个符号生成对应的连续载波片段，并将其与 I/Q 幅度相乘，完成上变频（Upconversion）。
    
