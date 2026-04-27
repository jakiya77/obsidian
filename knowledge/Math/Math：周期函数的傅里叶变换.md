重复门函数序列的**标准傅里叶级数（Fourier Series）计算公式**。
---

### 第一步：请出傅里叶级数的祖传公式

对于一个周期为 $T_p$ 的周期信号，它的第 $k$ 次谐波的系数 $c_k$ 的标准计算公式是：

$$c_k = \frac{1}{T_p} \int_{\text{一个周期}} \text{信号}(t) \cdot e^{-j k \omega_p t} dt$$

- $\omega_p$ 是基频，$\omega_p = \frac{2\pi}{T_p}$。
    
- **注意看最前面那个 $\frac{1}{T_p}$**，这就是后面所有占空比缩放的万恶之源！
    

### 第二步：代入我们特定的切片方块

我们的大周期是 $T_p$，它被切成了 $L_s$ 份，每份宽度是 $T_s$。所以：

- $T_p = L_s T_s$
    
- $\omega_p = \frac{2\pi}{L_s T_s}$
    

我们的信号 $g(t-lT_s)$ 在这一个周期里，只在 $[lT_s, (l+1)T_s]$ 这段时间开门（值为 1），其他时间都是 0。所以积分范围被压缩到了这个小方块内。我们把这些全代入标准公式：

$$c_k = \frac{1}{L_s T_s} \int_{lT_s}^{(l+1)T_s} 1 \cdot e^{-j k \frac{2\pi}{L_s T_s} t} dt$$

### 第三步：解微积分

对指数函数求积分，代入上下限：

$$c_k = \frac{1}{L_s T_s} \left[ \frac{e^{-j k \frac{2\pi}{L_s T_s} t}}{-j k \frac{2\pi}{L_s T_s}} \right]_{lT_s}^{(l+1)T_s}$$

接下来，强行把这个方块的中心时间点 $t_c = \frac{2l+1}{2}T_s$ 提出来作为公因式。

提取中心相位后，公式裂开成两部分：

$$c_k = \frac{1}{L_s T_s} \cdot e^{-j k \frac{2\pi}{L_s T_s} \frac{2l+1}{2}T_s} \cdot \left[ \frac{e^{-j k \frac{\pi}{L_s}} - e^{j k \frac{\pi}{L_s}}}{-j k \frac{2\pi}{L_s T_s}} \right]$$

### 第四步：各自归位，大功告成

现在，我们把这串长长的式子分别化简。

**1. 处理中间的指数部分（相位延迟）：**

把 $e$ 头顶上的 $T_s$ 和 2 约掉，正好变成：

$$e^{-j\frac{(2l+1)k\pi}{L_s}}$$

_(这就是你原文里代表时间延迟的那一半！)_

**2. 处理右边的方括号（凑 $Sa$ 函数）：**

利用欧拉公式 $\frac{e^{jx} - e^{-jx}}{2j} = \sin(x)$，方括号里的部分可以化简为：

$$T_s \frac{\sin\left(\frac{k\pi}{L_s}\right)}{\frac{k\pi}{L_s}}$$

**3. 合并最前面的系数：**

把最前面的 $\frac{1}{L_s T_s}$ 乘上刚刚凑出来的 $T_s$：

$$\frac{1}{L_s T_s} \cdot T_s = \frac{1}{L_s}$$

_(看！$T_s$ 被完美消掉了，只剩下了纯纯的占空比 $\frac{1}{L_s}$！)_

最后，把代表能量形状的 $\frac{\sin(x)}{x}$ 写成 $Sa(x)$，把这三部分拼在一起，就得到了你文章里的最终标准形态：

$$c_k = \frac{1}{L_s} Sa\left(\frac{k\pi}{L_s}\right) e^{-j\frac{(2l+1)k\pi}{L_s}}$$

---

这就如同拼图一样：**最前面的 $\frac{1}{T_p}$ 变成了占空比系数，积分提取的中点变成了相位偏移，而积分本身化作了 $Sa$ 包络线。
[[Math：傅里叶变换 - 周期调制信号的频域分解法]]