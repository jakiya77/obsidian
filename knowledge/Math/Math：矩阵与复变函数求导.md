
---


> [!tip] 推导前必问的两个问题
> 
> 1. **对谁求导？** 是对单个实数（如物理位置）、实数向量，还是完整的复数向量（如波束权重）求导？
>     
> 2. **上层结构是什么？** 是标量乘积，还是二次型矩阵相乘？先确定维度，再查阅对应表格！
>     

## 1. 实变量对复信号求导 (标量微积分)

**适用场景**：自变量 $x$ 是实数（如：天线位置 $u_n$、相位角 $\theta$），但函数 $z(x)$ 是复数。

**核心思想**：不需要矩阵法则，依靠一元导数的链式法则和复数代数性质。

| **运算目标**    | **运算法则公式**                                                                                                          | **核心提示与物理意义**                      |
| ----------- | ------------------------------------------------------------------------------------------------------------------- | ---------------------------------- |
| **复数乘积求导**  | $\frac{\partial}{\partial x} (z_1 z_2) = \frac{\partial z_1}{\partial x} z_2 + z_1 \frac{\partial z_2}{\partial x}$ | 与实数完全一致，直接应用乘积法则。                  |
| **共轭求导交换律** | $\frac{\partial (z^*)}{\partial x} = \left( \frac{\partial z}{\partial x} \right)^*$                                | **前提 $x$ 必须是实数！** 先求导再共轭 = 先共轭再求导。 |
| **复数加共轭**   | $z + z^* = 2\text{Re}\{z\}$                                                                                         | 凑出系数 2 的核心来源，推导中常用于化简。             |
| **复数模平方求导** | $$\frac{\partial}{\partial x} (\|z\|^2) = 2\text{Re} \left\{ z^* \frac{\partial z}{\partial x} \right\}$$           | 处理复信号功率或模长时的核心公式。                  |

---

## 2. 实数向量/矩阵求导 (实变梯度)

**适用场景**：自变量 $\mathbf{x}$ 是实数向量或矩阵，函数也是实数。

**核心思想**：经典机器学习与基础优化中的梯度，注意多维空间的偏导数排列和矩阵转置。

|**运算目标**|**运算法则公式**|**核心提示与注意事项**|
|---|---|---|
|**线性项 (内积)**|$\frac{\partial (\mathbf{a}^T \mathbf{x})}{\partial \mathbf{x}} = \frac{\partial (\mathbf{x}^T \mathbf{a})}{\partial \mathbf{x}} = \mathbf{a}$|$\mathbf{a}$ 是常数向量，结果为列向量。|
|**二次型 (一般方阵)**|$\frac{\partial (\mathbf{x}^T \mathbf{A} \mathbf{x})}{\partial \mathbf{x}} = (\mathbf{A} + \mathbf{A}^T)\mathbf{x}$|$\mathbf{A}$ 为任意方阵。|
|**二次型 (对称矩阵)**|$\frac{\partial (\mathbf{x}^T \mathbf{A} \mathbf{x})}{\partial \mathbf{x}} = 2\mathbf{A}\mathbf{x}$|最常见情况，系数 2 来源于 $\mathbf{A} = \mathbf{A}^T$。|
|**多变量链式法则**|$\frac{\partial f(\mathbf{g}(\mathbf{x}))}{\partial \mathbf{x}} = \left( \frac{\partial \mathbf{g}}{\partial \mathbf{x}} \right)^T \frac{\partial f}{\partial \mathbf{g}}$|严格注意矩阵转置位置，确保维度匹配。|

---

## 3. 复数向量/矩阵求导 (Wirtinger 微积分)

**适用场景**：自变量 $\mathbf{w}$ 是复数向量（如：波束赋形向量），函数是实数（如：信干噪比、发射功率）。

**核心思想**：为了获取最速上升的梯度方向，通常对**共轭向量 $\mathbf{w}^*$** 求导，求导时将 $\mathbf{w}$ 视作独立常数。

|**运算目标**|**运算法则公式**|**核心提示与注意事项**|
|---|---|---|
|**复线性项**|$\frac{\partial (\mathbf{w}^H \mathbf{a})}{\partial \mathbf{w}^*} = \mathbf{a}$|对 $\mathbf{w}^*$ 求导，$\mathbf{w}^H$ 是共轭转置。|
|**复线性项 (转置)**|$\frac{\partial (\mathbf{a}^H \mathbf{w})}{\partial \mathbf{w}^*} = \mathbf{0}$|$\mathbf{a}^H \mathbf{w}$ 中不包含 $\mathbf{w}^*$，故偏导为 0。|
|**复二次型 (Hermitian)**|$\frac{\partial (\mathbf{w}^H \mathbf{R} \mathbf{w})}{\partial \mathbf{w}^*} = \mathbf{R}\mathbf{w}$|**极易错点！** 这里**没有系数 2**，前提是 $\mathbf{R} = \mathbf{R}^H$。|
|**最速上升梯度**|$\nabla_{\mathbf{w}} f = \frac{\partial f}{\partial \mathbf{w}^*}$|优化复数权重时，更新方向应沿着对 $\mathbf{w}^*$ 的偏导方向。|

> [!warning] 避坑指南：系数 "2" 到底在哪？
> 
> - **在实数对复信号求导中**：系数 2 来自代数恒等式 $z + z^* = 2\text{Re}\{z\}$。
>     
> - **在实向量二次型求导中**：系数 2 来自对称矩阵的偏导组合 $\mathbf{x}^T \mathbf{A} \mathbf{x} \rightarrow 2\mathbf{A}\mathbf{x}$。
>     
> - **在复向量二次型求导中**：**没有系数 2！** $\mathbf{w}^H \mathbf{R} \mathbf{w}$ 对 $\mathbf{w}^*$ 求导直接等于 $\mathbf{R}\mathbf{w}$。
>