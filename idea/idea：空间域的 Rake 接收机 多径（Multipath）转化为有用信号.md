导师这个思路非常有前瞻性。本质上，这是一个**空间域的 Rake 接收机（Spatial Rake Receiver）**，也就是把原本可能造成频率选择性衰落或破坏性干涉的“多径（Multipath）”，转化为可以提供空间分集和能量增益的“有用信号”。

引入可移动天线（Movable Antennas, MA）作为辅佐，是这个方案的破局点。传统的固定阵列（如 ULA 或 UPA）在形成双波束并进行相位对齐时，受限于固定的天线间距（通常为 $\lambda/2$），其空间分辨率和波束赋形能力是受限的。而 MA 可以通过灵活改变天线位置，重塑阵列流型（Array Manifold），从而完美捕获并合并这两条路径的能量。


### 1. 系统建模与核心机制

假设发射端为单天线（或已固定波束的多天线），接收端配备 $N$ 个可移动天线元素。

信道可以建模为包含直达径（LoS，即主信号）和一条强反射径（NLoS，即 Multipath）的双径几何信道模型：

$$ \mathbf{h}(\mathbf{p}) = \alpha_0 e^{-j \phi_0} \mathbf{a}(\theta_0, \mathbf{p}) + \alpha_1 e^{-j \phi_1} \mathbf{a}(\theta_1, \mathbf{p}) $$

- $\alpha_0, \alpha_1$：主径和多径的复增益（包含路径损耗）。
    
- $\theta_0, \theta_1$：主径和多径的到达角（AoA）。
    
- $\mathbf{p} = [p_1, p_2, \dots, p_N]^T$：MA 的位置矢量。
    
- $\mathbf{a}(\theta, \mathbf{p})$：依赖于 MA 位置的阵列响应矢量。
    

**叠加和（Coherent Combining）的本质：**

接收端的数字波束赋形向量为 $\mathbf{w}$。接收到的信号为 $y = \mathbf{w}^H \mathbf{h}(\mathbf{p}) s + n$。

为了同时接收这两个方向的信号并进行最大比合并（MRC），最优的接收波束其实就是信道本身的共轭（归一化）：

$$ \mathbf{w}_{\text{opt}} = \frac{\mathbf{h}(\mathbf{p})}{\|\mathbf{h}(\mathbf{p})\|} $$

此时，接收到的信号功率达到最大，等于 $\|\mathbf{h}(\mathbf{p})\|^2$。

### 2. MA 在这里的核心作用

如果使用固定阵列，$\|\mathbf{h}_{\text{fixed}}\|^2$ 的值是固定的，由于两条路径的相位差，它们在某些天线上可能会发生**破坏性干涉（Destructive Interference）**。

加入 MA 后，我们可以通过优化位置 $\mathbf{p}$ 来实现两个极其强大的效果：

1. **波束级匹配（空间功率最大化）：** MA 可以移动到特定的位置，使得两条路径的相位在各个天线阵元上实现**建设性干涉（Constructive Interference）**，从而最大化等效信道增益 $\|\mathbf{h}(\mathbf{p})\|^2$。
    
2. **双主瓣重塑：** 相比于固定 $\lambda/2$ 间距的阵列，MA 可以通过稀疏阵列分布扩大等效孔径，形成两个非常尖锐的接收波束，分别精准对准 $\theta_0$ 和 $\theta_1$，同时压低其他方向的旁瓣（降低环境噪声或干扰）。
    

### 3. 优化问题构建

这个思路可以直接落地为一个联合优化问题。目标是最大化合并后的接收信噪比（SNR）：

$$ \max_{\mathbf{w}, \mathbf{p}} \quad |\mathbf{w}^H \left( \alpha_0 \mathbf{a}(\theta_0, \mathbf{p}) + \alpha_1 \mathbf{a}(\theta_1, \mathbf{p}) \right)|^2 $$

$$ \text{s.t.} \quad \|\mathbf{w}\|^2 \leq 1 $$

$$ \quad \quad \mathbf{p} \in \mathcal{C} \quad (\text{MA的移动区域限制}) $$

$$ \quad \quad |p_i - p_j| \geq d_{\min}, \forall i \neq j \quad (\text{避免天线互耦的安全距离}) $$

### 4. 算法求解框架

这是一个典型的非凸优化问题，建议采用交替优化（Alternating Optimization, AO）的方法：

- **步骤 1：固定 MA 位置 $\mathbf{p}$，优化波束 $\mathbf{w}$**
    
    这个问题有闭式解，即标准的 MRC 接收：$\mathbf{w}^* = \frac{\mathbf{h}(\mathbf{p})}{\|\mathbf{h}(\mathbf{p})\|}$。
    
- **步骤 2：固定波束 $\mathbf{w}$（或者将 $\mathbf{w}^*$ 代入原目标函数），优化 MA 位置 $\mathbf{p}$**
    
    由于 $\mathbf{a}(\theta, \mathbf{p})$ 中包含关于位置的高频非线性相位项 $e^{j \frac{2\pi}{\lambda} p_i \sin\theta}$，这部分相对棘手。
    
    - _解法 A：_ 使用连续凸近似（SCA），对相位项进行泰勒展开线性化。
        
    - _解法 B：_ 既然你之前也在看流形优化（Manifold Optimization），由于阵列流型本身的非线性，可以尝试基于梯度的算法（如 PGD）或者流形上的搜索来寻找最优位置。
        
    - _解法 C：_ 粒子群算法（PSO）等启发式算法（用于基准对比）。
        

### 5. 进阶探讨（与抗干扰结合）

如果你想把这个思路和你一直在做的**抗干扰**结合起来，故事会非常漂亮：

“在存在强干扰机（Jammer）的环境中，主干 LoS 径可能正好处于干扰机的波束范围内或被阻挡。此时，我们利用 MA 构造出不规则的双波束，不仅把 LoS 和 Multipath 叠加起来增强信号，**同时还在干扰机的方向 $\theta_J$ 处生成一个极深的零陷（Nulling）**。”

这就把“信号增强”和“干扰抑制”完美统合在了一起。