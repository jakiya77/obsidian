*20260401*

``` matlab
clc;clear all;close all;
Ne = 8 ; % elemnt of DMA
L = 3 ;
K = 300 ; % pilot
lambda = 1 ;
d = lambda/3 ;

P_jam = 30 ;  % dbm
P_signal = 10;% dbm
P_noise = -97;% dbm

P_jam_li = 10^((P_jam-30)/10) ;  % dbm
P_signal_li = 10^((P_signal-30)/10);% dbm
P_noise_li = 10^((P_noise-30)/10);% dbm

r_ADC = 6; % ADC 量化位数

rng(5);
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

% 2. DMA Dynamic Sampling
M = 2*Ne ; % sampling
b_DMA = 2 ; % 2 bits adc 4 possibilities

phase_sel = (0 : 2^b_DMA - 1) * (2*pi / 2^b_DMA);
codebook = exp(1i*phase_sel);
index = randi([1,2^b_DMA],M,Ne) ;
phi = codebook(index) ;

% 3 . Collect all the observations into a matrix Q
Q_analog = phi * Y ; % 模拟dma接到信号的过程 analog的信号
gamma_ADC_jam = max(abs([real(Q_analog(:)); imag(Q_analog(:))])); % AGC 增益调节
Q_digital = quantize_ADC(Q_analog, r_ADC, gamma_ADC_jam); % 经过低精度 ADC

Y_tilde = (phi' * phi) \ phi' * Q_digital;


error = norm(Y_tilde - Y, 'fro') / norm(Y, 'fro'); %
disp(['还原误差: ', num2str(error)])
```
>[!success]  [line 16 ] 改变r_ADC 从2-4-6，还原误差从0.78795-0.19015-0.048232


## Part 2 . Anti-jamming

![155](data:image/png;base64,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) (1)

![149](data:image/png;base64,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) (2)

``` matlab
% 4. SVD decomposition formualr (1)

[U_Y,S_Y,V_Y] = svd(Y_tilde.') ;
V_null = V_Y(:,2:Ne).' ;% extract the jam-nulling space
V_null_angle = angle(V_null) ;% find the near nulling space

% 5. discretiztaion mapping function formualr (2)

V_null_angle = mod(V_null_angle,2*pi);
step = 2*pi/(2^b_DMA) ;
quantized_angle = round(V_null_angle/step) *step;
phi_null = exp(1i * quantized_angle);
% 6. Near Nulling Space Projection

Phi_null_mat = phi_null ;
V = Phi_null_mat * Y; % 使用0空间的角度进行投影 忽略幅度
[A,B,C] = svd(V);
 
V_temp = V_null * Y; % 直接用0空间进行投影 复数
[D,E,F] = svd(V_temp);

V_direct = ones(Ne,Ne)*Y;%使用全通的模式 看一下功率
[U_dir,S_dir,V_dir] = svd(V_direct);

V_no_quan= V_null_angle*Y;
[AA,BB,CC] = svd(V_no_quan);



disp(diag(S_Y(1:4, 1:4))); % 原始干扰情况
disp(diag(B(1:4, 1:4))); % 使用量化后的近0空间的角度进行投影 忽略幅度
disp(diag(E(1:4, 1:4))); %直接用0空间进行投影 复数
disp(diag(BB(1:4, 1:4))); 

```
>[!hint] 比较了1-论文中忽略幅度的近零空间 2-直接使用零空间投影 3-使用全通模式不作投影 情况下的干扰抑制情况-通过乘上$\mathbf{\Phi}$矩阵后的信号进行SVD分解比较对角矩阵的最大值是否得到抑制。

| 原始干扰情况                                            | 零空间                                              | 近零空间                                               | 量化后近零空间                                            |
| ------------------------------------------------- | ------------------------------------------------ | -------------------------------------------------- | -------------------------------------------------- |
| 70.0771<br>    3.3975<br>    1.7744<br>    1.2893 | 3.0286<br>    0.1474<br>    0.0000<br>    0.0000 | 351.5384<br>   12.5867<br>    0.0001<br>    0.0000 | 109.9299<br>    7.6871<br>    0.0000<br>    0.0000 |



