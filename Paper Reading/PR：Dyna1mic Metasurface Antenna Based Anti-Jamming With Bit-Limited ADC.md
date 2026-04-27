>**🐻：**
>1️⃣、信号接收部分： 
>1. 我当下可以使用的接收设备是一个块可以拥有2bit相位调节自由度的ris，这块ris后面只连接了一根RF chain。
>2. 当我接收到信号的时候，信号会被我超表面上的$N_e$ 根天线分别接收，也就是说我的接收信号是一个 $N_e\times K$ 维度的矩阵，这里的$K$是我的信号长度。
>3. 我可以通过1中提到的相位调节对2中的信号进行接收，由于我的调节速度很快，在一个symbol里面我可以调节很多次相位，可以把这种天线相位的快速变化记作一个矩阵$\phi\in C^{N_e\times 1}$ ，使用这个矩阵和2中的接收信号想乘就可以得到一次变化的接收信号，尺寸为$1\times K$   
>4. 经过多次相位的改变进行观测，可以得到一个观测的矩阵在cpu中，$Q\in C^{M\times N_e}$ ,M是观测的次数，以及一个大的相位矩阵$\Phi\in C^{N_e\times M}$ ,然后通过计算可以得到估计出来的$\tilde y$ 
>2️⃣、干扰抑制部分：
>5. 通过对接收信号进行SVD分解可以得到干扰信号的nulling space，直接把nulling取相位作为$w_{opt}$乘回去进行干扰抑制 $V \approx \Phi_{null} h_{true} x_T^T + \Phi_{null} Z$ 
>6. 然后使用$LS$算法算出期望信号的信道，$\tilde{h} = \arg \min_h ||V - \Phi_{null} h x_T^T||_2^2$  ,此时的信道还黏着噪声的影响。
>3️⃣、 设计一组相位用于nulling的同时接收信号
>7. 让天线接收完信号并加权输出后，得到的结果，跟我们理想中完美无瑕的原始信号（导频 $x_T$）越像越好，误差越小越好。$\phi_{opt} = \arg \min_{\Phi_{opt}} E\{||x_T^T - \phi_{opt}^T(\tilde{h}x_T^T + \tilde{Y})||^2\}$

[[MA+思路整理]]
[[Reproduction Debug]]
[[Reproduction 时遇到的问题]]
##### 代码
``` matlab
## 1 .system modle

clc;clear all;close all;

Ne = 8 ; % elemnt of DMA

L = 3 ;

K = 1000 ; % pilot

lambda = 1 ;

d = lambda/3 ;

  

  

P_jam = 30 ; % dbm

P_signal = 10;% dbm

P_noise = -97;% dbm

  

P_jam_li = 10^((P_jam-30)/10) ; % dbm

P_signal_li = 10^((P_signal-30)/10);% dbm

P_noise_li = 10^((P_noise-30)/10);% dbm

  

r_ADC = 4; % ADC 量化位数

  

  

rng(526);

theta_jam_list = (rand(L,1) - 0.5) * 90 ;

theta_signal_list = (rand(L,1) - 0.5) * 90 ;

  

sv = @(theta)exp(1i*2*pi*d/lambda*(0:Ne-1)'*cosd(theta));

  

% jam channel and signal channle

  

alpha_jam = (randn(L,1)+1i*randn(L,1))/sqrt(2) ; % L*1

alpha_signal = (randn(L,1)+1i*randn(L,1))/sqrt(2) ;

  

h_jam = zeros(Ne,1);

  

h_signal = zeros(Ne,1);

for l = 1 : L

h_jam = h_jam + alpha_jam(l) * sv(theta_jam_list(l)) ;

h_signal = h_signal + alpha_signal(l) * sv(theta_signal_list(l)) ;

end

  

  

x_signal = sqrt(P_signal_li) * (randn(K,1) + 1i*randn(K,1))/sqrt(2);

x_jam = sqrt(P_jam_li) * (randn(K,1) + 1i*randn(K,1))/sqrt(2);

z = sqrt(P_noise_li) * (randn(Ne,K) + 1i*randn(Ne,K))/sqrt(2);

  

Y = h_signal * x_signal.' + h_jam * x_jam.' + z;

  

  

## 2. DMA Dynamic Sampling

M = 2*Ne ; % sampling

b_DMA = 2 ; % 2 bits adc 4 possibilities

% phase_sel = linspace(0,2*pi,2^b_DMA) 0和2pi是一个点

phase_sel = (0 : 2^b_DMA - 1) * (2*pi / 2^b_DMA);

% 1. consturct the codebook phi

codebook = exp(1i*phase_sel);

  

% 2. random ordering

index = randi([1,2^b_DMA],M,Ne)

% 3. generate phi

phi = codebook(index) ;

  

  

## 3 . Collect all the observations into a matrix Q

Q_analog = phi * Y ;

gamma_ADC_jam = max(abs([real(Q_analog(:)); imag(Q_analog(:))]));

Q_digital = quantize_ADC(Q_analog, r_ADC, gamma_ADC_jam); % 经过低精度 ADC

  

Y_tilde = (phi' * phi) \ phi' * Q_digital;

  

  

error = norm(Y_tilde - Y, 'fro') / norm(Y, 'fro'); %

disp(['还原误差: ', num2str(error)])

[U_Y,S_Y,V_Y] = svd(Y_tilde.') ;

  

V_null = V_Y(:,2:Ne).' ;% extract the jam-nulling space

V_null_angle = angle(V_null) ;% find the near nulling space

V_null_angle = mod(V_null_angle,2*pi);

step = 2*pi/(2^b_DMA) ;

quantized_angle = round(V_null_angle/step) *step;

phi_null = exp(1i * quantized_angle)

Phi_null_mat = phi_null ;

  

V = Phi_null_mat * Y; % 使用0空间的角度进行投影 忽略幅度

[A,B,C] = svd(V);

%

% V_temp = V_null * Y; % 直接用0空间进行投影 复数

% [D,E,F] = svd(V_temp);

%

% V_direct = ones(Ne,Ne)*Y;%使用全通的模式 看一下功率

% [U_dir,S_dir,V_dir] = svd(V_direct);

%

% V_PGD = Phi_PGD * Y;

% [U_PGD,S_PGD,V_PGD] = svd(V_PGD);

  

  

% disp(diag(S_Y(1:4, 1:4)));

% disp(diag(B(1:4, 1:4)));

% disp(diag(E(1:4, 1:4)));

% disp(diag(S_dir(1:4,1:4)));

% disp(diag(S_PGD(1:4,1:4)));

denom = norm(x_signal)^2;

Phi_inv = pinv(Phi_null_mat' * Phi_null_mat);

h_tilde = Phi_inv * (Phi_null_mat' * V * conj(x_signal)) / denom;

nmse_h = norm(h_tilde - h_signal)^2 / norm(h_signal)^2;

disp(['信道估计NMSE：',num2str(nmse_h)]);

信道估计NMSE：0.79938

  

% 可视化对比

figure;

plot(abs(h_signal), 'b-o', 'LineWidth', 1.5); hold on;

plot(abs(h_tilde), 'r--x', 'LineWidth', 1.5);

legend('True Channel', 'Estimated Channel');

title('Channel Estimation Performance');

grid on;

  

legend(["True Channel", "Estimated Channel"], "Position", [0.5109 0.6038 0.3596, 0.1262])

  

legend(["True Channel", "Estimated Channel"], "Position", [0.5359 0.7693 0.3641, 0.1320])

function y_q = quantize_ADC(y, r, gamma)

if r == inf

y_q = y; % 理想 ADC，不量化

return;

end

L = 2^r; % 量化阶数

step = 2 * gamma / L; % 量化步长

% 对实部进行量化和截断

y_real = real(y);

y_real(y_real > gamma) = gamma - step/2; % 正向饱和截断

y_real(y_real < -gamma) = -gamma + step/2; % 负向饱和截断

idx_real = floor((y_real + gamma) / step); % 映射到量化区间

idx_real(idx_real >= L) = L - 1;

idx_real(idx_real < 0) = 0;

y_real_q = -gamma + step * (idx_real + 0.5); % 重新赋予离散电平

% 对虚部进行量化和截断

y_imag = imag(y);

y_imag(y_imag > gamma) = gamma - step/2;

y_imag(y_imag < -gamma) = -gamma + step/2;

idx_imag = floor((y_imag + gamma) / step);

idx_imag(idx_imag >= L) = L - 1;

idx_imag(idx_imag < 0) = 0;

y_imag_q = -gamma + step * (idx_imag + 0.5);

% 组合输出数字基带信号

y_q = y_real_q + 1i * y_imag_q;

end

```