
## 如果把 θ 定义为“**纬度**”（从赤道起算，北正南负）

参数化单位球面：  

$$
\mathbf{r}(\theta,\phi)=\big(\cos\theta\cos\phi,\ \cos\theta\sin\phi,\ \sin\theta\big).  
$$

求偏导：  

$$
\mathbf{r}_\theta=(-\sin\theta\cos\phi,\ -\sin\theta\sin\phi,\ \cos\theta),\quad  
\mathbf{r}_\phi=(-\cos\theta\sin\phi,\ \cos\theta\cos\phi,\ 0).  
$$

内积得到第一基本形式系数：  
$$
E=\mathbf{r}_\theta\cdot\mathbf{r}_\theta=1,\quad  
F=\mathbf{r}_\theta\cdot\mathbf{r}_\phi=0,\quad  
G=\mathbf{r}_\phi\cdot\mathbf{r}_\phi=\cos^2\theta.  
$$
因此  
$$
\boxed{\mathrm{d}s^{2}=\mathrm{d}\theta^{2}+\cos^{2}\theta\mathrm{d}\phi^{2}}.
$$

## 如果把 θ 定义为“**余纬/极角**”（从 +z 轴起算，0°在极点、90°在赤道）

记此角为 (\vartheta)，参数化：  
$$
\mathbf{r}(\vartheta,\phi)=\big(\sin\vartheta\cos\phi,\ \sin\vartheta\sin\phi,\ \cos\vartheta\big), 
$$
同样推导可得  
$$
\boxed{\mathrm{d}s^{2}=\mathrm{d}\theta^{2}+\sin^{2}\theta\mathrm{d}\phi^{2}}.  
]
$$

> 总结：**(\cos^2\theta)** 出现在你把 (\theta) 当作“**纬度**”（从赤道量）时；若用我们阵列常用的“**余纬/极角**”（从 z 轴量），则是 **(\sin^2\vartheta)**。两者等价，只是角度零点不同。