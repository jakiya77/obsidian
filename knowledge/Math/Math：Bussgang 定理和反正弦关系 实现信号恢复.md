
### MATLAB 代码实现


Matlab

```
%% 5. 进阶演示：基于 Bussgang 定理与反正弦定律的信号恢复
% =========================================================================
% 场景说明：
% 假设我们有两个麦克风/天线，接收到一个宽带信号（高斯随机过程）。
% 信号到达第二个天线比第一个天线晚了一些（模拟波束方向/AoA）。
% 我们只有 1-bit ADC 的数据，我们试图恢复它们的相关性并找到时延。
% =========================================================================

disp('正在运行 Bussgang 定理演示...');

% 5.1 生成带限高斯噪声信号 (模拟真实世界的宽带信号)
N = 5000;                   % 采样点数
t_noise = (0:N-1)/Fs;
% 生成高斯白噪声并通过低通滤波器使其变得"平滑"（产生相关性）
original_noise = randn(1, N); 
b = fir1(50, 0.2);          % 低通滤波器系数
src_sig = filter(b, 1, original_noise); 

% 5.2 制造两个通道的信号 (Channel 2 有时延)
delay_samples = 25;         % 设置真实时延为 25 个采样点
sig_ch1 = src_sig(1:end-delay_samples);
sig_ch2 = src_sig(1+delay_samples:end); % sig_ch2 滞后 sig_ch1

% 5.3 1-bit 量化 (丢失幅度信息)
y_1bit_ch1 = sign(sig_ch1);
y_1bit_ch2 = sign(sig_ch2);

% 5.4 计算互相关函数 (Cross-Correlation)
max_lags = 100;
% A. 原始模拟信号的互相关 (Ground Truth - 理想情况)
[R_xx, lags] = xcorr(sig_ch1, sig_ch2, max_lags, 'coeff');

% B. 1-bit 信号的互相关 (直接计算)
% 注意：xcorr 只是做乘加运算，不包含反正弦校正
[R_yy, ~] = xcorr(y_1bit_ch1, y_1bit_ch2, max_lags, 'coeff');

% 5.5 【关键步骤】利用反正弦定律恢复原始相关性
% 公式：R_xx_estimated = sin( (pi/2) * R_yy )
R_xx_recovered = sin((pi/2) * R_yy);

% 5.6 绘图验证
figure('Color', 'w', 'Position', [100, 100, 1000, 600]);

% --- 子图1: 原始信号 vs 1-bit 信号 (时域片段) ---
subplot(2,1,1);
plot(sig_ch1(1:200), 'b', 'LineWidth', 1.5); hold on;
stairs(y_1bit_ch1(1:200), 'r', 'LineWidth', 1.2);
title('时域对比：原始高斯信号 vs 1-bit 量化信号 (前200点)');
legend('原始信号 (Ch1)', '1-bit 量化 (Ch1)');
grid on; ylim([-4 4]);

% --- 子图2: 相关函数恢复效果 (频域/统计域) ---
subplot(2,1,2);
% 1. 画出理想的原始相关曲线
plot(lags, R_xx, 'b', 'LineWidth', 4); hold on;
% 2. 画出 1-bit 直接算出的相关曲线 (幅度是错的，成比例缩小)
plot(lags, R_yy, 'k--', 'LineWidth', 1.5); 
% 3. 画出利用反正弦定律恢复的曲线
plot(lags, R_xx_recovered, 'g--', 'LineWidth', 2.5);

% 标记峰值位置 (用于波束形成/测向)
[~, idx_true] = max(abs(R_xx));
[~, idx_1bit] = max(abs(R_yy));
xline(lags(idx_true), 'b--', ['真实时延: ' num2str(lags(idx_true))]);
if lags(idx_true) == lags(idx_1bit)
    text(lags(idx_1bit)+2, 0.5, '1-bit 峰值位置未变!', 'Color', 'r', 'FontWeight', 'bold');
end

title('Bussgang 定理验证：互相关函数恢复');
xlabel('时延 (Lags)'); ylabel('归一化相关系数');
legend('原始信号互相关 (真值)', ...
       '1-bit 直接互相关 (线性近似)', ...
       '反正弦定律恢复后 (R_{recovered})', ...
       'Location', 'best');
grid on;

% 输出结论
fprintf('真实时延: %d samples\n', lags(idx_true));
fprintf('1-bit估算时延: %d samples\n', lags(idx_1bit));
```

### 结果解析：发生了什么？

#### 1. 为什么用高斯噪声？

代码中 `randn` 产生的是高斯白噪声。

- **Bussgang 定理**的核心前提是输入信号服从高斯分布。
    
- 如果是纯正弦波，1-bit 量化后的互相关是三角形的，而正弦波的互相关是余弦形的，它们之间的关系不是反正弦，比较复杂。但在实际通信和雷达中，信号通常看起来像高斯噪声（OFDM信号、CDMA信号或热噪声）。
    

#### 2. 三条曲线的对比

观察下方的图：
![[Figure_3.png]]
- **蓝色实线 (真值)**：这是如果我们有完美的、昂贵的高精度 ADC 算出来的互相关 $R_{xx}$。
    
- **黑色虚线 (1-bit 直接算)**：这是 $R_{yy}$。你会发现它的**形状**和蓝色很像，但是**幅度偏低**（略显尖锐，或者更接近线性）。
    
    - _重要观察：_ 尽管幅度变了，但是**峰值的位置（Peak Location）没有变！** 这就是为什么 1-bit ADC 依然可以做波束形成（Beamforming）的原因——我们要的是时延差（相位差），而不是绝对幅度。
        
- **绿色虚线 (恢复后)**：这是经过 $R = \sin(\frac{\pi}{2} R_{1bit})$ 计算后的结果。
    
    - 你会发现**绿色虚线几乎完美覆盖了蓝色实线**。
        
    - 这证明了即使信号被切成了 $\pm 1$，只要我们知道它是高斯过程，我们就能**数学上无损地**找回原本的信号相关性矩阵。
        

#### 3. 这里的恢复指什么？

我们无法恢复 $x(t)$ 的每一个具体的波形数值（那是做不到的，因为幅度信息丢了）。

但是，我们可以完美恢复 **协方差矩阵（Covariance Matrix）** 或 **相关函数**。

对于 **阵列信号处理（DOA估计、波束形成）** 来说，有了准确的协方差矩阵，就等于拥有了一切。

### 总结

1-bit 量化虽然丢了幅度，但保留了**过零点（Zero-crossings）**。对于高斯信号，过零点的位置极其精确地编码了信号的相位和相关性信息。通过 `sin(pi/2 * ...)` 这个简单的非线性映射，我们就能把便宜的 1-bit 数据“洗”回高精度的统计特征。