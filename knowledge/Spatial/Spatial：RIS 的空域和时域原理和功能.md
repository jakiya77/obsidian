# 1. 空域
## RIS 中涉及到的相位分为三个部分：
### 1. 来自入射方向的相位 $\theta_{incident}$
这个部分就是简单的天线阵列，比如一个线阵。接收了来自$\theta_{incident}$方向的信号，按照阵列的形状和结构，导向矢量按照$a=e^{jkdsin}$
![Image of 3D spherical coordinate system](https://encrypted-tbn2.gstatic.com/licensed-image?q=tbn:ANd9GcSv_ye1LnCYVUCeOR1NENEq_tdb2JUQncp6glRV5WqqWAkr9y7wl7-SgGpuwXkAirn-VykON1TXyWOKxB9UcsryRErG6up6ZP3kmEvV-awv6le5YVE)

**波矢量（Wave Vector）与天线位置矢量（Position Vector）的点积（Dot Product）**。

**相位差 = 波数 $\times$ (入射方向 $\cdot$ 天线坐标)**。

我们可以抛弃掉 `kron` 这种针对特定形状（矩形）的写法，直接写一个**万能通式**。这个通式不仅适用于矩形阵列，以后你做圆形阵列、L型阵列、甚至随机撒点的阵列，全部通用。

### 1. 数学通式推导

假设空间中有一个坐标系：

1. **波数 $k$**：$k = \frac{2\pi}{\lambda}$。
    
2. **入射方向单位矢量 $\mathbf{u}$**（由 $\theta, \phi$ 决定）：
    
    假设 $\theta$ 是与 $Z$ 轴夹角（俯仰），$\phi$ 是在 $XY$ 平面的方位角：
    
    $$\mathbf{u} = \begin{bmatrix} u_x \\ u_y \\ u_z \end{bmatrix} = \begin{bmatrix} \sin\theta \cos\phi \\ \sin\theta \sin\phi \\ \cos\theta \end{bmatrix}$$
    
3. **第 $n$ 个天线的坐标 $\mathbf{p}_n$**：
    
    $$\mathbf{p}_n = \begin{bmatrix} x_n \\ y_n \\ z_n \end{bmatrix}$$
    

**通用相位公式**就是两者的点积：

$$\text{Phase}_n = k \cdot (\mathbf{u} \cdot \mathbf{p}_n) = \frac{2\pi}{\lambda} (x_n \sin\theta \cos\phi + y_n \sin\theta \sin\phi + z_n \cos\theta)$$

导向矢量 $a(\theta, \phi)$ 的第 $n$ 个元素就是：

$$a_n = e^{j \cdot \text{Phase}_n}$$

### 2. 来自RIS本身可以提供的相位$\phi_{r}$
### 3. 反射后的相位$\theta_{reflect}$
