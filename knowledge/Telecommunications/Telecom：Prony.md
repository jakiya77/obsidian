## TL;DR

**Prony** 本质上是一种“从一串观测数据里，反推出里面藏着哪些复指数成分”的参数估计方法。
在通信里，复指数通常对应：

* 角度产生的空间相位变化
* 时延产生的频域相位变化
* 以及每条 path 的复增益

所以你看到的 **Prony-based angular-delay domain (PAD)**，本质就是：
**利用信道在天线维和频率维上的“指数结构”，直接估计各条路径的 AoA / delay / gain。**

---

## 1. 整体知识地图

```text
Prony
 ├─ 1.1 它解决什么问题
 ├─ 1.2 为什么信道能用 Prony
 ├─ 1.3 Prony 怎么从数据里找出 path
 └─ 1.4 PAD 为什么叫 angular-delay domain

```

*本次先展开最核心的 1.1 + 1.2。*

---

## 2. [1.1-A] Prony 到底在识别什么

假设你观察到一串数：


$$x[n]$$

它其实是几条复指数叠加：


$$x[n] = \sum_{\ell=1}^{L} \alpha_\ell z_\ell^n$$

这里：

* $L$：有几条 path
* $\alpha_\ell$：第 $\ell$ 条 path 的复增益
* $z_\ell$：这条 path 的“相位变化规律”

Prony 做的就是：
已知 $x[0], x[1], x[2], \dots$，反推出 $z_1, z_2, \dots, z_L$ 和 $\alpha_1, \dots, \alpha_L$。

---

## 3. [1.2-A] 为什么无线信道刚好适合 Prony

因为无线信道本身就是这种形式。

比如只看频率维：


$$H[k] = \sum_{\ell=1}^{L} \alpha_\ell e^{-j2\pi f_k\tau_\ell}$$

如果：


$$f_k = f_0 + k\Delta f$$

那么：


$$H[k] = \sum_{\ell=1}^{L} \underbrace{\alpha_\ell e^{-j2\pi f_0\tau_\ell}}_{\tilde\alpha_\ell} \left( e^{-j2\pi \Delta f\tau_\ell} \right)^k$$

你看，这就完全变成了：


$$H[k] = \sum_{\ell=1}^{L} \tilde\alpha_\ell z_\ell^k$$

其中：


$$z_\ell = e^{-j2\pi \Delta f\tau_\ell}$$

所以一旦 Prony 求出了 $z_\ell$，就能反推出：


$$\tau_\ell$$

也就是说：
**频率方向的相位旋转速度，告诉你 delay。**

---

## 4. [1.2-B] 空间维也一样

ULA（均匀线阵）下：


$$h[m] = \sum_{\ell=1}^{L} \alpha_\ell e^{-j2\pi \frac{d}{\lambda}m\sin\theta_\ell}$$

同样可以写成：


$$h[m] = \sum_{\ell=1}^{L} \alpha_\ell z_\ell^m$$

其中：


$$z_\ell = e^{-j2\pi\frac{d}{\lambda}\sin\theta_\ell}$$

于是 Prony 求出 $z_\ell$，就能求：


$$\theta_\ell$$

所以：

```text
沿天线阵元看相位变化
        ↓
     得到 AoA

沿子载波看相位变化
        ↓
     得到 delay

```

这就是为什么它可以做 angular-delay estimation。

---

## 5. 你先把 PAD 理解成这个

```text
接收到的 CSI / channel samples
          ↓
发现它是“多个复指数之和”
          ↓
Prony 分解
          ↓
找出每个复指数
          ↓
空间指数 → angle
频率指数 → delay
          ↓
得到每条 path 的
{angle, delay, gain}

```

所以 Prony-based angular-delay domain 并不是什么神秘神经网络。
它更像一个：
**经典的基于信号结构的参数估计算法。**

最关键的一句话：

> Prony 是在“拆复指数”。而多径信道恰好就是很多由 angle 和 delay 产生的复指数叠加。

如果你下一步想继续，我建议直接讲 **[1.3] Prony 到底怎么算出这些 $z_\ell$**，我可以用“两条 path 的具体数字例子”给你算一遍。