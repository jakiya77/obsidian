```matlab
% =========================================================================

% 真正逼出 DMA 极限的代码：N=4, 5个干扰, 全局穷举寻优

% =========================================================================

clc; clear; close all;

  

N = 4;

d = 0.5;

theta = -90:0.5:90;

theta_T = 0; % 目标在 0°

  

% 5 个强干扰 (超出 N-1 的极限)

theta_J = [-45, -20, 15, 35];

num_J = length(theta_J);

  

P_N = 1;

P_T = 10^(10/10);

P_J = 10^(40/10);

  

%% 2. 定义物理模式字典

modes_pointing = [-45, -20, 0, 20, 45];

K = length(modes_pointing);

% 提高物理天线自身的定向滤波能力 (4次方，模拟更真实的定向贴片天线)

EF_func = @(ang, phi_0) max(cosd(ang - phi_0).^4, 0.01);

  

%% 3. 传统相控阵

fixed_mode = 0;

a_space = @(ang) exp(1j * 2 * pi * d * (0:N-1)' * sind(ang));

  

v_T_trad = EF_func(theta_T, fixed_mode) * a_space(theta_T);

  

R_trad_J = zeros(N, N);

for k = 1:num_J

v_Jk = EF_func(theta_J(k), fixed_mode) * a_space(theta_J(k));

R_trad_J = R_trad_J + P_J * (v_Jk * v_Jk');

end

R_trad = R_trad_J + P_N * eye(N);

  

w_trad = (R_trad \ v_T_trad) / (v_T_trad' * (R_trad \ v_T_trad));

  

Pattern_trad = zeros(1, length(theta));

for i = 1:length(theta)

v_obs = EF_func(theta(i), fixed_mode) * a_space(theta(i));

Pattern_trad(i) = abs(w_trad' * v_obs)^2;

end

Pattern_trad_dB = 10*log10(Pattern_trad / max(Pattern_trad));

  

%% 4. DMA 阵列：全局穷举 (Exhaustive Search) 寻找最优状态

% 生成 5^4 = 625 种所有可能的物理状态组合

[A, B, C, D] = ndgrid(modes_pointing, modes_pointing, modes_pointing, modes_pointing);

all_combos = [A(:), B(:), C(:), D(:)]; % 625 x 4 矩阵

  

best_sinr = -inf;

best_modes = zeros(N, 1);

  

for idx = 1:size(all_combos, 1)

current_modes = all_combos(idx, :)';

% 当前物理状态下的目标导向矢量

v_T_temp = zeros(N, 1);

for n = 1:N

v_T_temp(n) = EF_func(theta_T, current_modes(n)) * exp(1j * 2 * pi * d * (n-1) * sind(theta_T));

end

% 当前物理状态下的干扰协方差矩阵

R_J_temp = zeros(N, N);

for k = 1:num_J

v_Jk_temp = zeros(N, 1);

for n = 1:N

v_Jk_temp(n) = EF_func(theta_J(k), current_modes(n)) * exp(1j * 2 * pi * d * (n-1) * sind(theta_J(k)));

end

R_J_temp = R_J_temp + P_J * (v_Jk_temp * v_Jk_temp');

end

R_temp = R_J_temp + P_N * eye(N);

% 计算理论最大 SINR

sinr_temp = real(v_T_temp' * (R_temp \ v_T_temp));

% 寻找全局最优

if sinr_temp > best_sinr

best_sinr = sinr_temp;

best_modes = current_modes;

end

end

  

disp('全局穷举找到的 DMA 最优异构物理状态 (度)：');

disp(best_modes');

  

% 使用最优物理状态计算最终的 DMA 权重与方向图

v_T_dma = zeros(N, 1);

for n = 1:N

v_T_dma(n) = EF_func(theta_T, best_modes(n)) * exp(1j * 2 * pi * d * (n-1) * sind(theta_T));

end

  

R_J_dma = zeros(N, N);

for k = 1:num_J

v_Jk_dma = zeros(N, 1);

for n = 1:N

v_Jk_dma(n) = EF_func(theta_J(k), best_modes(n)) * exp(1j * 2 * pi * d * (n-1) * sind(theta_J(k)));

end

R_J_dma = R_J_dma + P_J * (v_Jk_dma * v_Jk_dma');

end

R_dma = R_J_dma + P_N * eye(N);

  

w_dma = (R_dma \ v_T_dma) / (v_T_dma' * (R_dma \ v_T_dma));

  

Pattern_dma = zeros(1, length(theta));

for i = 1:length(theta)

v_obs_dma = zeros(N, 1);

for n = 1:N

v_obs_dma(n) = EF_func(theta(i), best_modes(n)) * exp(1j * 2 * pi * d * (n-1) * sind(theta(i)));

end

Pattern_dma(i) = abs(w_dma' * v_obs_dma)^2;

end

Pattern_dma_dB = 10*log10(Pattern_dma / max(Pattern_dma));

  

%% 5. 绘图对比

figure('Position', [100, 100, 900, 600]);

plot(theta, Pattern_trad_dB, 'b-', 'LineWidth', 2.5); hold on;

plot(theta, Pattern_dma_dB, 'r--', 'LineWidth', 2.5);

  

xline(theta_T, 'g-.', '目标(0^{\circ})', 'LineWidth', 2, 'LabelVerticalAlignment', 'bottom');

for k = 1:num_J

xline(theta_J(k), 'k:', sprintf('干扰%d', k), 'LineWidth', 1.5, 'LabelVerticalAlignment', 'bottom');

end

  

ylim([-40 5]); xlim([-90 90]);

grid on; set(gca, 'GridLineStyle', '--', 'GridAlpha', 0.4);

xlabel('空间角度 \theta (度)', 'FontSize', 12, 'FontWeight', 'bold');

ylabel('归一化总辐射增益 (dB)', 'FontSize', 12, 'FontWeight', 'bold');

title('N=4 5个干扰源 ', 'FontSize', 15);

legend('传统相控阵 ', 'DMA阵列 ', 'Location', 'South', 'FontSize', 12);
```
