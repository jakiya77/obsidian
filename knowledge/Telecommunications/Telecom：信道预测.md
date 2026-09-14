
## TL;DR

论文的几种方法可以分成：

```text
1. 真正的信道估计
   └─ MMSE channel estimation

2. 基于历史 RF 的信道预测
   ├─ Transformer-based prediction
   ├─ Kalman filter prediction
   └─ PAD / Prony prediction

3. 利用其他先验的预测
   ├─ Reciprocity-based prediction
   └─ Vision-aided prediction

4. 作者的方法
   └─ MSCP = RF + Vision
```

论文 Simulation Setup 明确列出了这些 baseline，并且说明 **输入历史信道本身是用 MMSE channel estimation 得到的**。

---

# 1. 整体知识地图

```text
Channel acquisition
│
├─ 1.1 MMSE estimation
│     当前 pilot → 当前 CSI
│
├─ 1.2 Kalman prediction
│     历史 CSI → 状态空间预测
│
├─ 1.3 Transformer prediction
│     历史 CSI → 学时间相关性
│
├─ 1.4 PAD / Prony
│     历史 CSI → angle-delay-Doppler → 外推
│
├─ 1.5 Reciprocity
│     利用上下行 angle/delay 共性
│
├─ 1.6 Vision-only
│     图像 → 用户/LoS geometry
│
└─ 1.7 MSCP
      历史 RF → 粗预测
             +
      当前 image → refinement
```

---

# 2. [1.1] MMSE channel estimation

这个是唯一最典型的：

估计当前信道 $\boxed{\text{估计当前信道}}$

## 它用什么？

用户发送已知 pilot：

$$
 y=\mathbf X\mathbf h+\mathbf n
$$

BS 知道：

- 发的 pilot X
    
- 收到的 y

要估：

$$
h
$$

MMSE 的思想就是找：

$$
\hat{\mathbf h}
$$

使：

$$
E\left[\|\mathbf h-\hat{\mathbf h}\|^2\right]
$$

最小。

你可以理解成：

> **结合收到的 pilot、噪声强度、信道统计信息，找均方误差最小的 CSI。**

这篇论文明确说：

> “we use the MMSE channel estimation method to obtain the input channels.”

所以：

```text
uplink SRS / pilot
      ↓
MMSE estimator
      ↓
h_{-9T}, ..., h_0
```

这些才成为后面各种 prediction 方法的输入。

---

# 3. [1.2] Kalman filter-based prediction

这个就不是重新发 pilot 估未来信道了。

它假设：

$\boxed{\text{channel 是一个随时间演化的动态状态}}$

比如简单写：

$\mathbf h_{t+1} = \mathbf A\mathbf h_t+\mathbf w_t$

也就是：

> 下一时刻信道和当前信道有一定关系。

Kalman 做两件事：

```text
过去信道
   ↓
根据动态模型预测下一时刻
   ↓
如果又有新观测，再校正
```

所以它的核心是：

$\boxed{\text{state-space model + recursive prediction}}$

论文明确把它描述为：

> using Kalman filter with RF-based sequential channel information.

### 最大弱点

如果突然：

```text
LoS blocked
用户突然转弯
path突然出现/消失
```

原来的状态演化模型就可能失效。

---

# 4. [1.3] Transformer-based channel prediction

这个就是：

$\boxed{\text{不用人工规定动态方程，让神经网络自己学时间规律}}$

输入：

$\mathbf h_{t-M+1},\ldots,\mathbf h_t$

输出：

$\hat{\mathbf h}_{t+1}$

Transformer 的 self-attention 会学习：

> 哪些历史时刻和未来信道更相关？

所以：

```text
h_-9
h_-8
...
h_0
   ↓
Transformer
   ↓
future h
```

论文把它描述成：

> extracts temporal correlations in historical channels with DNNs.

它和 Kalman 的区别最简单记：

$\boxed{\text{Kalman：人为给模型}}$ 
$\boxed{\text{Transformer：数据自己学模型}}$

---

# 5. [1.4] PAD / Prony prediction
[[Telecom：Prony]]


PAD 不直接预测整个：

$\mathbf h$

而是先认为 channel 是少数 path 叠加：

```text
Path 1:
angle θ1
delay τ1
Doppler ν1

Path 2:
angle θ2
delay τ2
Doppler ν2
```

先进入：

$\boxed{\text{angle-delay domain}}$

把 path 分离出来。

然后用 Prony 从历史相位变化估：

$\nu_\ell$

也就是每条 path 的 Doppler / complex exponential 变化速度。

最后：

过去 path→估 Doppler→外推未来\text{过去 path} \rightarrow \text{估 Doppler} \rightarrow \text{外推未来}

论文对 PAD 的描述就是：

> exploits the angle-delay-Doppler structure of the multi-path channel.

所以它属于：

物理参数化预测\boxed{\text{物理参数化预测}}

---

# 6. [1.5] Reciprocity-based prediction

这个稍微特殊。

论文引用的是：

> Tracking FDD massive MIMO downlink channels by exploiting delay and angular reciprocity.

核心思想：

虽然 FDD 上下行频率不同，所以：

$h_{\rm UL}\neq h_{\rm DL}$

但很多传播几何参数还是接近的，比如：

$\boxed{\text{angle}}$

和：

$\boxed{\text{delay}}$ 

因为墙、人、路径几何没有因为上下行频率变了就换位置。

所以：

```text
uplink channel
   ↓
提取 angle / delay
   ↓
利用角度、时延 reciprocity
   ↓
推断 downlink channel
```

这篇 MSCP 论文只把它列为 benchmark，并没有详细推导这个方法。

---

# 7. [1.6] Vision-only channel prediction

这个就完全不依赖历史 RF 做主要预测。

它用：

$\boxed{\text{camera image}}$

识别：

- mobile device 位置
    
- LoS 方向
    

然后根据 geometry 推 channel / beam。

论文指出它的主要问题是：

> single-modal vision method mainly identifies the LoS component based on mobile position.

所以：

```text
Image
 ↓
找到用户
 ↓
估几何方向
 ↓
估 LoS channel
```

但是如果：

```text
LoS被挡住
+
NLoS reflection 很重要
```

就容易出问题。

论文的 simulation 也明确说，single-modal vision 主要识别 LoS，而 MSCP 可以进一步识别 reflecting points 和 obstacles。

---

# 8. [1.7] 最后才是作者的 MSCP

它其实是把：

### RF-only prediction

和：

### Vision-only prediction

结合。

先：

$\mathbf h_{\rm past}$

经过 CPPN：

$\tilde\beta,\tilde\alpha,\tilde\phi,\tilde\theta$

也就是：

> RF 根据历史趋势给出一个粗未来预测。

然后 camera：

$I_t$

告诉它：

> 当前环境到底有没有突然变化。

最后 VCRN：

$(\tilde P_t,I_t) \rightarrow \hat P_t$

再重构：

$\hat{\mathbf h}_t.$

这就是整篇论文。

---

## 最后，你可以这样记这几条曲线

|方法|看什么|核心思想|
|---|---|---|
|MMSE|当前 pilot|**现在测**|
|Kalman|历史 RF|**按动态模型往后推**|
|Transformer|历史 RF|**学时间规律往后推**|
|PAD/Prony|历史 RF|**拆 path + Doppler 外推**|
|Reciprocity|UL/RF|**用角度/时延共性推 DL**|
|Vision-only|图像|**看用户/LoS geometry**|
|MSCP|RF + image|**RF 粗预测 + 图像纠错**|

最关键的区别其实就是：

$\boxed{ \text{Estimation：有当前时刻 pilot} }$

vs.

$\boxed{ \text{Prediction：没有未来 pilot，只能往未来猜} }$

你如果想继续，下一步最值得单独讲的是 **MMSE channel estimation 到底怎么算**，因为这是这篇论文所有 RF 输入的起点。