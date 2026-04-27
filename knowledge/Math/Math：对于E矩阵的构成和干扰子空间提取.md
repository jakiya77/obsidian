### 1. 矩阵 $\mathbf{E}$ 的定义本身就是列向量的集合

特征向量矩阵 $\mathbf{E}$ 是按列排列的 。如果天线阵列有 $M$ 个阵元，那么每一个特征向量 $\mathbf{e}_i$ 都是一个 $M \times 1$ 的**列向量**（它代表了空间中的一个方向基底）。 所以，矩阵 $\mathbf{E}$ 展开后长这样：

$$\mathbf{E} = \begin{bmatrix} | & | & & | \\ \mathbf{e}_1 & \mathbf{e}_2 & \dots & \mathbf{e}_M \\ | & | & & | \end{bmatrix}$$

### 2. 把 $\mathbf{R} = \mathbf{E} \mathbf{D} \mathbf{E}^H$ 彻底拆开

- $\mathbf{D}$ 是一个对角矩阵，对角线上的元素是特征值（代表功率）$d_1, d_2, \dots, d_M$ 。假设 $d_1$ 是最大的特征值（即最强的干扰）。
    
- $\mathbf{E}^H$ 是 $\mathbf{E}$ 的共轭转置。因为 $\mathbf{E}$ 是按列排的，转置之后，$\mathbf{E}^H$ 的每一**行**就变成了原本的特征向量的共轭转置 $\mathbf{e}_i^H$。
    

如果我们把这三个矩阵相乘的动作写成完整的分块矩阵形式：

$$\mathbf{R} = \begin{bmatrix} | & | & & | \\ \mathbf{e}_1 & \mathbf{e}_2 & \dots & \mathbf{e}_M \\ | & | & & | \end{bmatrix} \begin{bmatrix} d_1 & 0 & \dots & 0 \\ 0 & d_2 & \dots & 0 \\ \vdots & \vdots & \ddots & \vdots \\ 0 & 0 & \dots & d_M \end{bmatrix} \begin{bmatrix} - & \mathbf{e}_1^H & - \\ - & \mathbf{e}_2^H & - \\ & \vdots & \\ - & \mathbf{e}_M^H & - \end{bmatrix}$$

根据分块矩阵的乘法法则，这个长长的式子可以等价展开为一系列秩为 1 的矩阵之和：

$$\mathbf{R} = d_1 (\mathbf{e}_1 \mathbf{e}_1^H) + d_2 (\mathbf{e}_2 \mathbf{e}_2^H) + \dots + d_M (\mathbf{e}_M \mathbf{e}_M^H)$$

### 3. 结论：为什么是列？

观察上面展开后的最终结果：

最大的特征值（干扰功率）$d_1$，它是跟 $\mathbf{e}_1 \mathbf{e}_1^H$ 绑定在一起的。

这里的 $\mathbf{e}_1$ 就是提取出来的**代表干扰空间方向的基底**，而这个 $\mathbf{e}_1$ 正是当初矩阵 $\mathbf{E}$ 的**第一列**。

- 矩阵 $\mathbf{E}$ 里的**行**，代表的是物理上第 $m$ 个天线阵元在不同空间基底上的投影系数，它没有完整的空间几何意义。
    
- 矩阵 $\mathbf{E}$ 里的**列**，才是一个完整的 $M \times 1$ 维度的空间导向矢量，代表着干扰或信号到底是从哪个物理角度射过来的。