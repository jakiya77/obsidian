### 原理

**在普通的DMA上，可以做的是提供一个波束成形的权重：$\mathbf{\Phi}_{null}$，这个权重乘上信道和信号是最终接收到的东西：$Y_{jam}=\Phi_{null}h_{jam}x_{jam}$** 

**而在可移动天线中，可以直接改变信道$h_{jam}$，由于信道$\mathbf{h}_{jam}(p) = \sum_{l=1}^{L} \alpha_l e^{j \frac{2\pi}{\lambda} p \cos(\theta_l)}$ ，通过改变p，就可以直接改变h，从而实现对于多径干扰信号的叠加消除，让干扰信号在进入ADC之前就受到抑制。**

##### 加入可移动天线可以起作用的原因：
 1. 空间相关性（Spatial Correlation）的解耦

这是最根本的数学原因。对于给定的信号角度 $\theta_s$ 和干扰角度 $\theta_j$，它们在阵列上的导向矢量（Steering Vector）分别为 $\mathbf{a}(\theta_s, \mathbf{p})$ 和 $\mathbf{a}(\theta_j, \mathbf{p})$。

在固定阵列中，如果这两个矢量**高度相关**（即夹角很小），数学上表示为：

$$\rho = \frac{|\mathbf{a}^H(\theta_s, \mathbf{p}) \mathbf{a}(\theta_j, \mathbf{p})|}{\|\mathbf{a}_s\| \|\mathbf{a}_j\|} \approx 1$$

当你使用 PGD 或 SVD 算法生成一个投影矩阵 $\mathbf{\Phi}$ 来抑制干扰时，因为信号和干扰在空间上几乎“重合”，抑制干扰的操作会**不可避免地把信号也投影掉**。

**MA 的数学魔力：** 通过改变 $\mathbf{p}$，你实际上是在调整这两个高维矢量的每一维的相位。由于 $\cos \theta_s \neq \cos \theta_j$，改变 $p$ 对两者的相位改变量是不一样的。优化 $p$ 的本质是**最小化这种空间相关性 $\rho$**，让信号矢量和干扰矢量在 16 维空间里趋于**正交**。

---

2. 有效秩（Effective Rank）与条件数

在你的代码中，接收信号矩阵可以近似表示为 $Y = \mathbf{H}_s X_s + \mathbf{H}_j X_j + N$。

当你进行 SVD 分解时，你实际上是在提取信号空间的特征值。

- **固定阵列：** 如果位置不理想，干扰信道 $\mathbf{h}_j$ 的增益可能极大，掩盖了信号。
    
- **MA 优化：** 你通过物理避让减小了 $\|\mathbf{h}_j\|^2$。在数学上，这降低了干扰子空间的“能量权重”。当干扰的特征值减小时，信号特征值在总能量中的占比提升，这直接改善了**信道矩阵的条件数（Condition Number）**。
    
- **结果：** 矩阵变得更健康，PGD 算法在迭代求解权重 $\mathbf{\Phi}$ 时，数值稳定性更高，不会因为干扰过强而陷入局部最优。