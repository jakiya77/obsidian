**因为信道 $h(p)$ 是复高斯分布（Complex Gaussian），而复高斯变量的模的平方（即功率）在数学上严格服从指数分布。**

我们可以通过严谨的数学推导来一步步证明，这个过程非常优美。

### 第一步：拆解信道 $h(p)$ 为实部和虚部

在你的代码中，你使用了 `randn + 1i*randn` 来生成多径信道。根据中心极限定理，当多条独立路径的信号叠加时，接收到的信道增益 $h(p)$ 可以表示为一个复高斯随机变量：

$$h(p) = X + jY$$

其中：

- $X$ 是实部（同相分量 In-phase），服从零均值的高斯分布 $X \sim \mathcal{N}(0, \sigma^2)$。
    
- $Y$ 是虚部（正交分量 Quadrature），也服从零均值的高斯分布 $Y \sim \mathcal{N}(0, \sigma^2)$。
    
- $X$ 和 $Y$ 是相互独立的。
    

### 第二步：定义接收功率 $P$

功率是复信号模的平方，因此：

$$P = |h(p)|^2 = X^2 + Y^2$$

现在我们的数学问题变成了：**已知 $X$ 和 $Y$ 都是独立的高斯分布，求 $Z = X^2 + Y^2$ 的概率分布。**

### 第三步：推导 $X^2 + Y^2$ 的分布（极坐标法）

由于 $X$ 和 $Y$ 是独立的，它们的联合概率密度函数（Joint PDF）等于各自 PDF 的乘积：

$$f_{X,Y}(x, y) = f_X(x) \cdot f_Y(y) = \left( \frac{1}{\sqrt{2\pi}\sigma} e^{-\frac{x^2}{2\sigma^2}} \right) \cdot \left( \frac{1}{\sqrt{2\pi}\sigma} e^{-\frac{y^2}{2\sigma^2}} \right)$$

$$f_{X,Y}(x, y) = \frac{1}{2\pi\sigma^2} e^{-\frac{x^2 + y^2}{2\sigma^2}}$$

我们引入极坐标变换，令 $r^2 = x^2 + y^2$（注意，这里的 $r^2$ 就是我们的功率 $P$）。在微积分中，平面上的一个微小面积微元可以写为 $dxdy = r dr d\theta$。

我们先求包络 $r$ 的分布。对角度 $\theta$ 从 $0$ 到 $2\pi$ 积分：

$$f_R(r) = \int_{0}^{2\pi} f_{X,Y}(r\cos\theta, r\sin\theta) \, r \, d\theta$$

$$f_R(r) = \int_{0}^{2\pi} \frac{1}{2\pi\sigma^2} e^{-\frac{r^2}{2\sigma^2}} \, r \, d\theta = \frac{r}{\sigma^2} e^{-\frac{r^2}{2\sigma^2}}, \quad r \ge 0$$

_(注：这就是大名鼎鼎的**瑞利分布 Rayleigh Distribution**，描述的是信号幅度 $|h(p)|$ 的分布)_。

### 第四步：从幅度推导到功率（变量代换）

我们已经知道幅度 $r$ 的分布，但我们需要的是功率 $P = r^2$ 的分布。

根据概率论中的随机变量代换公式：$f_P(p) = f_R(r) \left| \frac{dr}{dp} \right|$。

因为 $p = r^2$，所以 $r = \sqrt{p}$，对其求导得到：$\frac{dr}{dp} = \frac{1}{2\sqrt{p}}$。

将 $r = \sqrt{p}$ 和导数代入刚才的瑞利分布 $f_R(r)$ 中：

$$f_P(p) = f_R(\sqrt{p}) \cdot \frac{1}{2\sqrt{p}}$$

$$f_P(p) = \left( \frac{\sqrt{p}}{\sigma^2} e^{-\frac{p}{2\sigma^2}} \right) \cdot \frac{1}{2\sqrt{p}}$$

化简后，$\sqrt{p}$ 完美对消：

$$f_P(p) = \frac{1}{2\sigma^2} e^{-\frac{p}{2\sigma^2}}, \quad p \ge 0$$

### 结论：它就是指数分布！

我们来观察一下推导出的这个公式。如果我们将**平均功率**定义为 $\mu$。

因为 $P = X^2 + Y^2$，所以期望（均值）为：

$$\mu = \mathbb{E}[P] = \mathbb{E}[X^2] + \mathbb{E}[Y^2] = \sigma^2 + \sigma^2 = 2\sigma^2$$

把 $2\sigma^2$ 替换为 $\mu$，上面的式子立刻变成了：

$$f_P(p) = \frac{1}{\mu} e^{-\frac{p}{\mu}}, \quad p \ge 0$$

这与**指数分布（Exponential Distribution）**的标准定义完全一致！

**总结一下：**

1. 信号的实部和虚部是**高斯分布**（正态分布）。
    
2. 信号的包络（幅度）是**瑞利分布**。
    
3. 信号的模平方（功率）是**指数分布**（这在统计学上也叫自由度为2的卡方分布 $\chi^2_2$）。