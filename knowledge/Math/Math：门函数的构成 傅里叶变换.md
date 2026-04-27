![[png求和.png]]
# $\sum_{n=-\infty}^{+\infty}\epsilon(t-nT_p)-\epsilon(t-T_s-nT_p)$ 
### 第一步：认识基础砖块 $\epsilon(t)$（拨动开关）

在信号系统里，$\epsilon(t)$ 叫做**单位阶跃函数**。

别管名字，你就把它想象成一个极度死板的**“灯泡开关”**：

- 在时间 $t < 0$ 之前，灯是关的（值为 $0$）。
    
- 到了 $t = 0$ 这一瞬间，“啪”的一下灯亮了，并且**永远**保持亮着的状态（值为 $1$）。
    

### 第二步：认识 $\epsilon(t - T_s)$（迟到的开关）

括号里减去一个时间，在数学图像上就是**“动作延迟”**。 所以，$\epsilon(t - T_s)$ 就是一个**迟到了 $T_s$ 秒**的开关：

- 它在 $0$ 秒的时候没反应，一直苟到 $t = T_s$ 这一刻，才“啪”的一下打开，然后永远亮着。
    

### 第三步：制造一个方块 $\epsilon(t) - \epsilon(t - T_s)$（定时器）

现在高潮来了，也是你克服数学恐惧的关键。把刚才两个动作相减，会发生什么？

- $t = 0$ 时：你按下了开启开关（总值变成 $1$）。
    
- $t = T_s$ 时：迟到的那个开关也打开了。你用第一个开关的 $1$，减去第二个开关的 $1$（$1 - 1 = 0$），相当于你又把它**关掉**了。
    

这一下一上，就在平坦的时间轴上，隆起了一个**高度为 $1$、宽度为 $T_s$ 的小方块**！

这就相当于一个定时开启了 $T_s$ 秒的微波炉。这就是门函数 $g(t)$ 的真面目。

### 第四步：看懂终极形态 $\sum_{n=-\infty}^{+\infty} \dots$（无限复制粘贴）

最求和符号 $\sum$ 还有里面多出来的 $nT_p$。

在数学里，带有 $n$ 从负无穷大到正无穷大的求和符号，翻译成人话只有一个意思：**“按固定间距，无限复制粘贴”**。

- 这里的 $T_p$ 就是复制粘贴的间距也就是大周期）。
    
- 整个公式的意思就是：宽度是$T_s$ 的方波重复在$nT_p$的位置出现
    

仿真代码
---

```matlab
% --- 1. 参数与时间轴准备 ---

Ts = 1; % 每个小方块的宽度 (持续时间)

Tp = 3; % 两个小方块起点之间的距离 (周期)

dt = 0.001 ； 

t = -2:dt :10; % 建立一根从 -2 秒到 10 秒，步长非常细的时间轴

  

% --- 2. 得到阶跃函数 epsilon(t) ---

% 在 MATLAB 里，逻辑判断 t >= 0 会返回一串 0 和 1 的数组。这完美等效于阶跃函数。

step_0 = t >= 0;

  

% --- 3. 得到延迟的阶跃函数 epsilon(t - Ts) ---

% 把开启的时间向后推迟 Ts 秒

step_delayed = t >= Ts;

  

% --- 4. 得到单个“时间切片”小方块 ---

% 两者相减：0秒时开启，Ts秒时关闭

single_pulse = step_0 - step_delayed;

  

% --- 5. 得到求和的一串子 (模拟无穷级数) ---

% 计算机算不了无穷大，我们用 n = -2 到 n = 3 来模拟前后几个周期

periodic_pulse = zeros(size(t)); % 先铺一张全 0 的空白画布

  

for n = -2:3

% 这里直接翻译了核心公式：epsilon(t - n*Tp) - epsilon(t - Ts - n*Tp)

pulse_n = (t >= n*Tp) - (t >= Ts + n*Tp);

% 将每个周期的方块累加到画布上

periodic_pulse = periodic_pulse + pulse_n;

end

% --- 6. 傅里叶变换 ---


Fs = 1 / dt; % 计算采样频率 (这里是 10000 Hz)

L = length(t); % 获取信号的总长度

  

% 执行快速傅里叶变换 (FFT)

Y = fft(periodic_pulse);

  

% FFT 算出来的是复数，我们要看幅度，并且要把它居中对齐

  

P_magnitude = abs(Y / L); % 求绝对值并归一化幅度

P_centered = fftshift(P_magnitude); % 用 fftshift 把 0 Hz (零频) 移到横坐标正中间

  

% 构造对应的“频率轴” (取代原来的时间轴 t)

f = Fs * (-(L/2):(L/2)-1) / L;

  

% --- 7. 绘图展示对比 ---

figure;

  

% 画出时域图像 (也就是你脑子里的“排班表”)

subplot(2,1,1);

plot(t, periodic_pulse, 'LineWidth', 2);

title('时域图像：周期性门函数序列 g(t)');

xlabel('时间 (s)'); ylabel('幅度');

ylim([-0.2 1.2]); grid on;

  

% 画出频域图像 (也就是论文公式里的那些“谐波”)

subplot(2,1,2);

plot(f, P_centered, 'LineWidth', 1.5);

title('频域图像：门函数的傅里叶变换 (看 Sinc 包络与离散谐波)');

xlabel('频率 (Hz)'); ylabel('幅度');

% xlim([-3 3]); % 因为高频能量很小，我们截取 -3 到 3 Hz 放大看最核心的部分

grid on;

zp = BaseZoom();

zp.run;
  

% --- 8. 绘图展示门函数的生成 ---

figure;

subplot(4,1,1); plot(t, step_0, 'LineWidth', 2);

title('\epsilon(t) 阶跃函数'); ylim([-0.2 1.2]); grid on;

  

subplot(4,1,2); plot(t, step_delayed, 'LineWidth', 2);

title('\epsilon(t-T_s) 延迟的阶跃函数'); ylim([-0.2 1.2]); grid on;

  

subplot(4,1,3); plot(t, single_pulse, 'LineWidth', 2);

title('单个脉冲 \epsilon(t) - \epsilon(t-T_s)'); ylim([-0.2 1.2]); grid on;

  

subplot(4,1,4); plot(t, periodic_pulse, 'LineWidth', 2);

title('周期性脉冲序列 (模拟 \Sigma 求和)'); ylim([-0.2 1.2]); grid on;
```

仿真结果
---
![[门函数频域 1.png]]
![[门函数频域 采样率 10 1.png]]