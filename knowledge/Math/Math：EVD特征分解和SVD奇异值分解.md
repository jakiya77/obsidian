
# SVD 与 EVD 的物理映射与数学等效性

在阵列信号处理（如 MUSIC 算法、空间滤波）中，我们面对的核心物理量有两个：

1. **观测数据矩阵 $\mathbf{Y}$**（维度：$N_e \times K$，即 `天线数 × 快拍数`）
    
2. **空间协方差矩阵 $\mathbf{R}$**（维度：$N_e \times N_e$，即 `天线数 × 天线数`）
    

两者分别对应 SVD 和 EVD，其物理本质是从“瞬时混合信号”向“平稳空间统计量”的升华。

---

## 一、 数学定义与物理维度

### 1. 奇异值分解 (SVD) —— 作用于原始数据 $\mathbf{Y}$

$$\mathbf{Y} = \mathbf{U} \mathbf{\Sigma} \mathbf{V}^H$$

- **$\mathbf{U}$ (左奇异矩阵，纯空域)：** 每一列代表一个独立的空间方向基向量。
    
- **$\mathbf{\Sigma}$ (奇异值对角阵，幅度)：** 物理量纲为**“累积幅度 (Amplitude)”**，代表该空间方向上接收到的电压/电流强度。
    
- **$\mathbf{V}^H$ (右奇异矩阵，纯时域)：** 每一行代表该空间方向信号在 $K$ 个快拍中的**时域波形起伏**。
    

### 2. 特征值分解 (EVD) —— 作用于协方差矩阵 $\mathbf{R}$

$$\mathbf{R} = \frac{1}{K} \mathbf{Y}\mathbf{Y}^H$$

$$\mathbf{R} = \mathbf{E} \mathbf{D} \mathbf{E}^H$$
- **$\mathbf{E}$ (特征向量矩阵，纯空域)：** 每一列代表一个独立的空间方向基向量。
    
- **$\mathbf{D}$ (特征值对角阵，平均功率)：** 物理量纲为**平均功率 (Average Power)**，代表该空间方向的信号功率或噪声方差。
    
- 需要注意 这里提取的$\mathbf{E}$ 的构成，不要弄混行和列 [[Math：对于E矩阵的构成和干扰子空间提取]]
#### 在仿真中通过模型驱动方式构建理论协方差矩阵

$$\mathbf{R}_{\text{IN}} = \sum_{k=1}^{K} P_{J} \mathbf{v}(\theta_{Jk}, \boldsymbol{\phi}) \mathbf{v}^H(\theta_{Jk}, \boldsymbol{\phi}) + P_N \mathbf{I}$$

- **干扰子空间**：第一项对应信号处理中的 $\mathbf{A} \mathbf{P} \mathbf{A}^H$ 或特征分解中的信号子空间 $\mathbf{E}_s \mathbf{D}_s \mathbf{E}_s^H$。
    
- **噪声基底**：$P_N \mathbf{I}$ 代表白噪声在空域各向同性。
    
> [!important]- 信号特征分解和模型重构的区别和联系
> 
> $\sum_{k=1}^{K} P_{J} \mathbf{a}(\theta_{Jk}) \mathbf{a}^H(\theta_{Jk})$，在数学上可以完美地写成一个大矩阵的乘法：
假设把 5 个干扰的导向矢量拼成一个 $N \times K$ 的**阵列流型矩阵（Steering Matrix）** $\mathbf{A}$：
$$\mathbf{A} = [\mathbf{a}(\theta_{J1}), \mathbf{a}(\theta_{J2}), \dots, \mathbf{a}(\theta_{J5})]$$
再把 5 个干扰的功率放在一个 $K \times K$ 的**对角阵 $\mathbf{P}$** 里：
$$\mathbf{P} = \begin{bmatrix} P_J & 0 & \dots & 0 \\ 0 & P_J & \dots & 0 \\ \vdots & \vdots & \ddots & \vdots \\ 0 & 0 & \dots & P_J \end{bmatrix} = P_J \mathbf{I}_{K}$$
物理求和公式，在代数上就绝对等于：
$$\mathbf{R}_{\text{IN (干扰部分)}} = \mathbf{A} \mathbf{P} \mathbf{A}^H$$
这个结构 $\mathbf{A} \mathbf{P} \mathbf{A}^H$，和特征值分解（EVD） $\mathbf{E} \mathbf{D} \mathbf{E}^H$ **一模一样**
>- 物理上的 **阵列流型矩阵 $\mathbf{A}$** 对应了 代数上的 **特征向量矩阵 $\mathbf{E}$**。
  >  
> - 物理上的 **干扰功率矩阵 $\mathbf{P}$** 对应了 代数上的 **特征值矩阵 $\mathbf{D}$**。





## 二、 核心等效性推导

证明为什么对 $\mathbf{R}$ 求 EVD，与对 $\mathbf{Y}$ 求 SVD 殊途同归：

将 $\mathbf{Y} = \mathbf{U} \mathbf{\Sigma} \mathbf{V}^H$ 代入协方差矩阵的定义中：

$$\mathbf{R} = \frac{1}{K} (\mathbf{U} \mathbf{\Sigma} \mathbf{V}^H) (\mathbf{U} \mathbf{\Sigma} \mathbf{V}^H)^H$$

根据矩阵转置共轭律 $(\mathbf{A}\mathbf{B})^H = \mathbf{B}^H\mathbf{A}^H$：

$$\mathbf{R} = \frac{1}{K} \mathbf{U} \mathbf{\Sigma} (\mathbf{V}^H \mathbf{V}) \mathbf{\Sigma}^H \mathbf{U}^H$$

**【物理质变点】**：因为时域矩阵 $\mathbf{V}$ 是酉矩阵（Unitary），其列向量相互正交且归一化，必然满足 $\mathbf{V}^H \mathbf{V} = \mathbf{I}$。

这意味着：**在乘以 $\mathbf{Y}^H$ 并求统计平均的过程中，随时间随机波动的瞬时波形被彻底抵消了。**

公式坍缩为最终形态：

$$\mathbf{R} = \mathbf{U} \left( \frac{1}{K} \mathbf{\Sigma} \mathbf{\Sigma}^H \right) \mathbf{U}^H$$

---

## 三、 绝对映射关系字典 (Mapping Table)

将坍缩后的公式与 EVD 标准形态 $\mathbf{R} = \mathbf{E} \mathbf{D} \mathbf{E}^H$ 严格对齐，得出以下映射真理：

| **物理维度**  | **SVD (作用于数据 Y)**       | **EVD (作用于协方差 R)** | **严格等式关系**                                       | **核心物理意义**                                              |
| --------- | ----------------------- | ------------------ | ------------------------------------------------ | ------------------------------------------------------- |
| **空间方向**  | 左奇异矩阵 $\mathbf{U}$      | 特征向量 $\mathbf{E}$  | **$\mathbf{E} = \mathbf{U}$**                    | 提取出的空间基向量**完全等价**。MUSIC 算法中的噪声子空间 $\mathbf{E}_n$ 即来源于此。 |
| **能量/功率** | 奇异值矩阵 $\mathbf{\Sigma}$ | 特征值矩阵 $\mathbf{D}$ | **$\mathbf{D} = \frac{1}{K} \mathbf{\Sigma}^2$** | 这是**量纲跃迁**的关键。                                          |
| **时间波形**  | 右奇异矩阵 $\mathbf{V}$      | 无                  | $\mathbf{V}^H \mathbf{V} = \mathbf{I}$ (已湮灭)     | EVD 放弃了恢复具体波形，只保留纯粹的空间与功率统计特征。                          |

---

## 四、 量纲演进的三个物理层级

在硬件仿真与门限设置时，必须严格区分以下三个量纲：

1. **$\mathbf{\Sigma}$ (奇异值)：** 量纲为 **幅度 (Amplitude)**。数值随时间 $K$ 累积。
    
2. **$\mathbf{\Sigma}^2$ (奇异值平方)：** 量纲为 **总能量 (Total Energy)**。数值随采样时间 $K$ 趋于无穷大而发散。
    
3. **$\mathbf{D} = \frac{1}{K} \mathbf{\Sigma}^2$ (特征值)：** 量纲为 **平均功率 (Average Power)**。**极其重要：** 只有除以 $K$ 后，矩阵底部的噪声特征值才会死死钉在物理底噪（如 $\sigma_n^2$ 或 $-97\text{dBm}$）上。此时才能设定一个绝对常数阈值，将信号子空间与噪声子空间切分开来。
    

---

## 五、 工程应用准则

- **何时用 `[E, D] = eig(R)`？**
    
    - **经典理论推导**与学术论文书写。
        
    - 当快拍数 $K$ 远大于天线数 $N_e$ 时（如 $K=500, N_e=16$），计算 $16 \times 16$ 的 $\mathbf{R}$ 的 EVD 计算量极小，速度更快。
        
- **何时用 `[U, S, V] = svd(Y)`？**
    
    - **直接数据域处理 (Direct Data Domain)。**
        
    - 当处于低精度 ADC 或极端强干扰下，计算 $\mathbf{Y}\mathbf{Y}^H$ 会使矩阵的**条件数平方**（导致微弱信号的数值精度被截断掩盖）。直接对 $\mathbf{Y}$ 做 SVD 提取 $\mathbf{U}$，数值稳定性极高，能最大程度保全微弱信号的空间特征。
    -


 [[Math：奇异值分解]]
 [[Math：EVD和SVD的手算过程]]
 