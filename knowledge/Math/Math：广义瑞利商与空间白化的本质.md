#📥-待复习
## 📌 核心一句话记忆

> [!TIP]
> 
> 广义瑞利商就是在问：**选择一个方向 $\mathbf{w}$，如何让收益矩阵 $\mathbf{A}$给我的有用能量最大，同时让代价矩阵 $\mathbf{B}$带来的干扰代价最小？**
> 
> 在接收机问题中，最优合并权重的方向永远满足：
> 
> $$\boxed{\mathbf{w}^\star \propto \mathbf{R}_{i+n}^{-1}\mathbf{h}}$$
> 
> **物理直觉**：先按干扰噪声协方差做抑制（白化），再对有用信号通道做匹配（对准）。

## 🛠️ 概念基石：从普通瑞利商说起

### 1. 物理来源：带约束的能量最大化

我们最初的冲动是最大化输出功率（二次型形式）：$\max \mathbf{x}^H \mathbf{A} \mathbf{x}$。

为了防止无限制放大，必须约束其为单位长度 $\mathbf{x}^H \mathbf{x} = 1$。

将约束条件直接归一化融进目标函数，便诞生了**普通瑞利商**：

$$R(\mathbf{x}) = \frac{\mathbf{x}^H \mathbf{A} \mathbf{x}}{\mathbf{x}^H \mathbf{x}}$$

### 2. 几何图像：寻找椭球体的最长轴

- **矩阵 $\mathbf{A}$ 的二次型**：在几何上描绘了一个多维椭球体。
    
- **特征向量**：决定了椭球体的轴向（主轴方向）。
    
- **特征值**：决定了椭球体在各个轴向上的半径（放大倍数）。
    
    最大化普通瑞利商，本质上就是寻找这个椭球体**最长的那根主轴**。最大值即为最大特征值 $\lambda_{max}$，最优方向即为其对应的特征向量：$\mathbf{A}\mathbf{x}^* = \lambda_{max}\mathbf{x}^*$。
    

## 🪄 核心对齐：矩阵白化 $\approx$ 高维向量归一化

| **维度概念**     | **物理/数学对象**                | **归一化工具**                                                                        | **归一化后的完美状态**           |
| ------------ | -------------------------- | -------------------------------------------------------------------------------- | ----------------------- |
| **一维 (幅值维)** | 向量 $\mathbf{x}$（长度有长有短）    | 除以标量长度$$\|\mathbf{x}\| = \sqrt{\sum \|x_i\|^2}= \sqrt{\mathbf{x}^H \mathbf{x}}$$ | **单位向量**：长度恒为 1，消除长度影响。 |
| **多维 (功率维)** | 噪声协方差 $\mathbf{B}$（空间高低不平） | 乘矩阵方根的逆$\mathbf{B}^{-1/2}$                                                       | **白化空间**：各个方向噪声功率完全等高。  |


### 🔍 对应的核心公式验证

- **一维向量归一化公式**：
    
    $$\|\mathbf{x}\| = (\mathbf{x}^H\mathbf{x})^{1/2}$$
    
    $$\mathbf{x}_{\text{norm}} = \frac{\mathbf{x}}{\|\mathbf{x}\|} = (\mathbf{x}^H\mathbf{x})^{-1/2} \mathbf{x}$$
    
- **多维空间白化公式**：
    
    $$\mathbf{\tilde{n}} = \mathbf{B}^{-1/2} \mathbf{n}$$
    
    此时新噪声的协方差矩阵完美归一化为单位阵：$E[\mathbf{\tilde{n}}\mathbf{\tilde{n}}^H] = \mathbf{I}$。
    

> [!WARNING]
> 
> **为什么是 $\mathbf{B}^{-1/2}$ 而不是 $\mathbf{B}^{-1}$？**
> 
> 因为协方差矩阵 $\mathbf{B}$ 描述的是**功率（平方）**维度，而信号和权重向量在**幅值**维度。必须先开根号降维到幅值维，再求逆。

## 📐 严谨数学推导全过程

1. **引入新变量（白化转换）**：
    
    令 $\mathbf{\tilde{w}} = \mathbf{B}^{1/2} \mathbf{w} \implies \mathbf{w} = \mathbf{B}^{-1/2} \mathbf{\tilde{w}}$
    
    代入广义瑞利商的分母，分母被烫平：$\mathbf{w}^H \mathbf{B} \mathbf{w} = \mathbf{\tilde{w}}^H \mathbf{\tilde{w}}$。
    
2. **同步转换分子**：
    
    原式退化为关于 $\mathbf{\tilde{w}}$ 的普通瑞利商：
    
    $$R(\mathbf{\tilde{w}}) = \frac{\mathbf{\tilde{w}}^H \left( \mathbf{B}^{-1/2} \mathbf{A} \mathbf{B}^{-1/2} \right) \mathbf{\tilde{w}}}{\mathbf{\tilde{w}}^H \mathbf{\tilde{w}}}$$
    
3. **套用标准线代工具（谱定理保底必有解）**：
    
    令中间的新对称矩阵 $\mathbf{C} = \mathbf{B}^{-1/2} \mathbf{A} \mathbf{B}^{-1/2}$，最优解满足标准特征值方程：
    
    $$\left( \mathbf{B}^{-1/2} \mathbf{A} \mathbf{B}^{-1/2} \right) \mathbf{\tilde{w}}^* = \lambda \mathbf{\tilde{w}}^*$$
    
4. **还原原变量 $\mathbf{w}^*$**：
    
    代入 $\mathbf{\tilde{w}}^* = \mathbf{B}^{1/2} \mathbf{w}^*$：
    
    $$\mathbf{B}^{-1/2} \mathbf{A} \mathbf{w}^* = \lambda \mathbf{B}^{1/2} \mathbf{w}^*$$
    
    两边同时**左乘 $\mathbf{B}^{-1/2}$**，强行消去右边的矩阵，暴露出变身后的广义特征值方程：
    
    $$\mathbf{B}^{-1} \mathbf{A} \mathbf{w}^* = \lambda \mathbf{w}^*$$
    
5. **结合天线具体问题（秩为 1 的奇迹）**：
    
    代入 $\mathbf{A} = P_s \mathbf{h}\mathbf{h}^H$ 和 $\mathbf{B} = \mathbf{R}_{i+n}$：
    
    $$\mathbf{R}_{i+n}^{-1} \mathbf{h} \cdot \left( P_s \mathbf{h}^H \mathbf{w}^* \right) = \lambda \mathbf{w}^*$$
    
    因为我们只关心权重向量的**方向**，剥离标量系数后完美收网：
    
    $$\boxed{\mathbf{w}^* \propto \mathbf{R}_{i+n}^{-1} \mathbf{h}}$$
    

## ⚡ 遗忘急救工具箱：Norm 运算法则

如果以后公式推导卡壳，随时调用以下三条通用法则：

- **定义式**：$\|\mathbf{x}\| = \sqrt{\sum |x_i|^2}$
    
- **矩阵转换（最常用）**：
    
    $$\|\mathbf{x}\|^2 = \mathbf{x}^H \mathbf{x}$$
    
    （看到模平方，立刻拆成左边共轭转置行向量 $\times$ 右边列向量）。
    
- **齐次性**：$\|a\mathbf{x}\| = |a| \cdot \|\mathbf{x}\|$（复数常数提出来要取模）。
    

