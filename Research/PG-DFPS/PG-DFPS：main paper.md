

**作者**：IEEE 出版技术部（职员，IEEE）

**致谢**：本文由 IEEE 出版技术组制作。他们位于新泽西州皮斯卡塔韦。

**稿件状态**：收到日期：2021年4月19日；修订日期：2021年8月16日。

> [!abstract] **摘要**
> 
> 本文档介绍了最常见的文章要素，以及如何使用 IEEEtran 类配合 $\LaTeX$ 排版出适合提交给 IEEE 的文件。通过合理选择类选项，IEEEtran 可以生成会议、期刊和技术简报（通信）论文。

**关键词**：文章提交，IEEE，IEEEtran，期刊，$\LaTeX$，论文，模板，排版。

## 1. 引言

本文旨在作为使用 IEEEtran.cls 1.8b 及更高版本在 $\LaTeX$ 下排版的 IEEE 期刊论文的“示例文章文件”。最常用的要素已包含在简化且更新的说明文档 `New_IEEEtran_how-to.pdf` 中。对于较少使用的要素，您可以参考原始的 `IEEEtran_HOWTO.pdf`。

本文假设读者具备 $\LaTeX$ 的基本操作知识。鼓励初学者阅读 Tobias Oetiker 的 _《一份不太简短的 $\LaTeX$ 介绍》_（The Not So Short Introduction to $\LaTeX$），该文件可在以下网址获取：[CTAN 链接](http://tug.ctan.org/info/lshort/english/lshort.pdf)。

## 2. 系统模型与位置域动因

^sec-system-model

### 2.1 流体天线几何结构与信道模型

^subsec-ma-channel-model

我们考虑一个窄带远场上行抗干扰通信系统，其中单天线目标发射机与配备有 $N$ 元流体天线（Movable Antenna, MA）阵列的接收机进行通信。单天线干扰机发射强干扰信号以降低目标信号的接收质量。MA 阵元部署在长度为 $A$ 的一维有限孔径上。在不失一般性的情况下，MA 的位置按以下顺序排列：

$$-\frac{A}{2} \le p_1 < p_2 < \cdots < p_N \le \frac{A}{2} \tag{1}$$

且满足最小阵元间距约束：

$$p_{n+1}-p_n \ge d_{\min}, \quad n=1,\ldots,N-1 \tag{2}$$

公式 (1) 中的排序消除了 MA 阵元之间的排列模糊性，因为所考虑的性能指标在重新标记天线位置时保持不变。一个充要的可行性条件为：

$$A \ge (N-1)d_{\min} \tag{3}$$

对于从角度 $\theta$ 到达的平面波，位于位置 $p$ 处的 MA 阵元的响应建模为：

$$a(p,\theta) = e^{-j\frac{2\pi}{\lambda}p\sin\theta} \tag{4}$$

其中 $\lambda$ 为载波波长，$\theta$ 是相对于阵列法线方向（Broadside）测量得到的。

在位置 $p$ 处观测到的目标信道建模为：

$$h(p) = \sum_{\ell=0}^{L_s-1} \alpha_\ell e^{-j\frac{2\pi}{\lambda}p\sin\theta_{s,\ell}} \tag{5}$$

其中 $L_s$ 为目标传播路径的数量，$\alpha_\ell$ 为第 $\ell$ 条目标路径的复增益，$\theta_{s,\ell}$ 为其到达角（AoA）。类似地，在位置 $p$ 处观测到的干扰信道建模为：

$$g(p) = \sum_{\ell=0}^{L_j-1} \beta_\ell e^{-j\frac{2\pi}{\lambda}p\sin\theta_{j,\ell}} \tag{6}$$

其中 $L_j$、$\beta_\ell$ 和 $\theta_{j,\ell}$ 分别表示干扰路径的数量、第 $\ell$ 条干扰路径的复增益和到达角。

收集各阵元的信道响应，目标信道向量和干扰信道向量分别由下式给出：

$$\mathbf h(\mathbf p) = [h(p_1),h(p_2),\ldots,h(p_N)]^T \in\mathbb C^{N\times 1} \tag{7}$$

以及

$$\mathbf g(\mathbf p) = [g(p_1),g(p_2),\ldots,g(p_N)]^T \in\mathbb C^{N\times 1} \tag{8}$$

### 2.2 接收信号模型与位置优化

^subsec-received-signal-model

MA 阵列处的接收信号为：

$$\mathbf y = \sqrt{P_s}\mathbf h(\mathbf p)x_s + \sqrt{P_j}\mathbf g(\mathbf p)x_j + \mathbf n \tag{9}$$

其中 $P_s$ 和 $P_j$ 分别表示目标信号和干扰信号的发射功率。目标符号 $x_s$ 和干扰信号 $x_j$ 归一化为：

$$\mathbb E\{|x_s|^2\}=1, \qquad \mathbb E\{|x_j|^2\}=1$$

我们假设 $x_s$、$x_j$ 以及接收机噪声相互独立。噪声向量服从：

$$\mathbf n\sim\mathcal{CN}(\mathbf 0,\sigma^2\mathbf I_N)$$

其中 $\sigma^2$ 为噪声功率。假设接收机可以获取或估计候选 MA 位置处的目标和干扰信道向量 $\mathbf h(\mathbf p)$ 与 $\mathbf g(\mathbf p)$。

对于给定的位置向量 $\mathbf p$，线性接收合并器 $\mathbf w\in\mathbb C^{N\times 1}$ 得到的输出信干噪比（SINR）为：

$$\Gamma(\mathbf p,\mathbf w) = \frac{ P_s|\mathbf w^H\mathbf h(\mathbf p)|^2 }{ P_j|\mathbf w^H\mathbf g(\mathbf p)|^2 + \sigma^2\|\mathbf w\|^2 } \tag{10}$$

对于固定的 $\mathbf p$，使 SINR 最大化的合并器方向为：

$$\mathbf w^\star(\mathbf p) \propto \left( P_j\mathbf g(\mathbf p)\mathbf g^H(\mathbf p) + \sigma^2\mathbf I_N \right)^{-1} \mathbf h(\mathbf p) \tag{11}$$

该方向在标量归一化意义下等价于最小均方误差（MMSE）合并器。由于输出 SINR 对 $\mathbf w$ 的缩放具有不变性，将公式 (11) 代入公式 (10) 可得最大输出 SINR：

$$\Gamma^\star(\mathbf p) = P_s \mathbf h^H(\mathbf p) \left( P_j\mathbf g(\mathbf p)\mathbf g^H(\mathbf p) + \sigma^2\mathbf I_N \right)^{-1} \mathbf h(\mathbf p) \tag{12}$$

应用矩阵求逆引理（Matrix Inversion Lemma），公式 (12) 可以重写为：

$$\Gamma^\star(\mathbf p) = \frac{P_s}{\sigma^2} \left( \|\mathbf h(\mathbf p)\|^2 - \frac{ P_j |\mathbf g^H(\mathbf p)\mathbf h(\mathbf p)|^2 }{ \sigma^2+P_j\|\mathbf g(\mathbf p)\|^2 } \right) \tag{13}$$

这一表达式表明，最大输出 SINR 受三个耦合量的控制：**目标信道能量** $\|\mathbf h(\mathbf p)\|^2$、**干扰信道能量** $\|\mathbf g(\mathbf p)\|^2$ 以及**目标-干扰空间相关性** $|\mathbf g^H(\mathbf p)\mathbf h(\mathbf p)|^2$。因此，一个良好的 MA 位置集合不仅应当提供强大的目标信道增益，还应当使目标信道向量和干扰信道向量在接收端具有足够的空间区分度。

为了后续使用，我们将归一化的目标-干扰相关系数定义为：

$$\rho_{hg}(\mathbf p) = \frac{ |\mathbf h^H(\mathbf p)\mathbf g(\mathbf p)|^2 }{ \|\mathbf h(\mathbf p)\|^2 \|\mathbf g(\mathbf p)\|^2+\epsilon } \tag{14}$$

其中 $\epsilon$ 是一个为了数值稳定性而引入的微小正常数。较小的 $\rho_{hg}(\mathbf p)$ 表明目标信道向量与干扰信道向量之间具有更好的空间可分性。

优化问题 $\mathbf P_0$ 定义为：

$$\begin{aligned} \mathbf P_0:\quad \max_{\mathbf p}\quad & \Gamma^\star(\mathbf p) \\ \text{s.t.}\quad & -\frac{A}{2}\le p_1 < p_2 < \cdots < p_N \le \frac{A}{2}, \\ & p_{n+1}-p_n\ge d_{\min},\quad n=1,\ldots,N-1. \end{aligned} \tag{P0}$$

优化问题 $\mathbf P_0$ 通常难以求解。多径信道响应是 MA 位置的高度振荡函数，这导致 $\Gamma^\star(\mathbf p)$ 具有非凸性，并在有限孔径内产生多个局部最优解。此外，最小间距约束将各个 MA 的位置耦合在一起，因此无法总是同时选出单点表现优异的位置。因此，直接在位置域进行黑盒搜索可能需要大量的目标函数评估次数，尤其是在孔径较大、MA 数量较多或数值实现采用精细空间采样分辨率的情况下。这些挑战促使我们提出一种物理引导的位置域选择策略，该策略利用多径诱导的空间结构，在务实的搜索预算下识别出高质量的 MA 位置。

### 2.3 双径双点阵洞察

^subsec-two-path-dual-lattice

由于多个空间频率的叠加，公式 (5) 和 (6) 中的通用多径模型可能会产生复杂的位置域剖面。为了获得清晰的物理洞察，我们首先考虑占主导地位的双径情况。由此产生的周期性结构为所提 MA 设计中使用的通用位置域选择规则提供了依据。

对于具有两条主导路径的目标信道，我们写成：

$$h(p) = \alpha_0 e^{-j\frac{2\pi}{\lambda}p\sin\theta_{s,0}} + \alpha_1 e^{-j\frac{2\pi}{\lambda}p\sin\theta_{s,1}} \tag{15}$$

其中对于 $\ell\in\{0,1\}$，有 $\alpha_\ell=|\alpha_\ell|e^{j\phi_{s,\ell}}$。定义：

$$\Delta k_s = \frac{2\pi}{\lambda} \left( \sin\theta_{s,1}-\sin\theta_{s,0} \right), \qquad \Delta\phi_s = \phi_{s,1}-\phi_{s,0}$$

则位置 $p$ 处的目标信道功率为：

$$|h(p)|^2 = |\alpha_0|^2 + |\alpha_1|^2 + 2|\alpha_0||\alpha_1| \cos\left(\Delta\phi_s-\Delta k_s p\right) \tag{16}$$

当 $\Delta k_s\neq0$ 时，目标信号相长相干（Constructive）的位置满足：

$$\Delta\phi_s-\Delta k_s p = 2m\pi, \quad m\in\mathbb Z \tag{17}$$

并构成一个周期性集合，其周期为：

$$T_s=\frac{2\pi}{|\Delta k_s|} \tag{18}$$

类似地，对于占主导地位的双径干扰信道：

$$g(p) = \beta_0 e^{-j\frac{2\pi}{\lambda}p\sin\theta_{j,0}} + \beta_1 e^{-j\frac{2\pi}{\lambda}p\sin\theta_{j,1}} \tag{19}$$

其中 $\beta_\ell=|\beta_\ell|e^{j\phi_{j,\ell}}$。定义：

$$\Delta k_j = \frac{2\pi}{\lambda} \left( \sin\theta_{j,1}-\sin\theta_{j,0} \right), \qquad \Delta\phi_j = \phi_{j,1}-\phi_{j,0}$$

位置 $p$ 处的干扰功率为：

$$|g(p)|^2 = |\beta_0|^2 + |\beta_1|^2 + 2|\beta_0||\beta_1| \cos\left(\Delta\phi_j-\Delta k_j p\right) \tag{20}$$

当 $\Delta k_j\neq0$ 时，干扰相消相干（Destructive）的位置对应于公式 (20) 的局部极小值，即：

$$\Delta\phi_j-\Delta k_j p = (2n+1)\pi, \quad n\in\mathbb Z \tag{21}$$

在这些位置上，干扰功率被最小化为：

$$|g(p)|^2_{\min} = \left(|\beta_0|-|\beta_1|\right)^2 \tag{22}$$

因此，仅在 $|\beta_0|=|\beta_1|$ 的特殊情况下才会出现完美的干扰零陷。通常情况下，相消位置应当被理解为干扰功率的局部极小值，而非完美的零陷。它们的周期为：

$$T_j=\frac{2\pi}{|\Delta k_j|} \tag{23}$$

双径表达式为“路径参数如何决定有限孔径内良好 MA 位置的可获得性”提供了有用的洞察。特别是，相位偏移 $\Delta\phi_s$ 和 $\Delta\phi_j$ 分别决定了目标相长图案和干扰相消图案的空间平移。对于固定的空间周期 $T_s$ 和 $T_j$，改变相位偏移不会改变相邻相长或相消区域之间的间距，但会改变它们在位置域沿线的具体位置。因此，即使两个信道实现具有相同的空间周期，它们在有限孔径内的良好区域也可能呈现出不同程度的对齐。

相比之下，由相应路径的 AoA 间隔决定的空间频率差，控制着目标剖面和干扰剖面的空间周期。两条路径的投影 AoA 差异越大，引起的空间频率差就越大，从而空间周期越短。结果是，在给定的孔径内会出现更多的相长或相消区域。可用目标相长机会和干扰相消机会的数量主要分别由归一化孔径大小 $A/T_s$ 和 $A/T_j$ 决定，而它们的准确位置则进一步取决于相位偏移和孔径边界。

这些观察突显了有限孔径的重要性。当空间周期大于或与孔径相当时，只能利用空间振荡的有限部分，MA 接收机可能几乎没有良好的位置可供选择。当空间周期较短时，孔径内可能会出现更多良好的区域。然而，对于多天线选择，最小间距约束阻止了同时使用任意多个邻近的良好位置。因此，实际可用的良好位置数量是由多径诱导的空间周期、孔径大小、相位偏移和天线间距约束共同决定的。

上述双径分析引出了一种双重良好（dual-favorable）的空间解释：一个良好的 MA 位置预期应当既接近目标相长区域，又同时接近干扰相消区域。然而，这种点阵（lattice）解释仅对双径情况是精确的，因为此时每个信道剖面均由单个空间频率差决定。

对于 $L_s>2$ 或 $L_j>2$ 的通用多径信道，多个空间频率分量相互叠加，产生的位置域剖面可能不再形成单一的周期性点阵。因此，双径情况仅作为一种可分析的物理基准，而在通用多径环境中，良好区域是直接从归一化的目标和干扰功率剖面中识别出来的。这促使我们提出一种物理引导的位置域选择策略，该策略利用多径诱导的位置域结构，而不依赖于精确的点阵交集。

### 2.4 有限孔径双重良好区域

^subsec-dual-favorable-region

受到上述洞察的启发，我们通过归一化的目标和干扰功率剖面来表征有限孔径的空间机会。具体而言，在孔径 $\mathcal A=[-A/2,A/2]$ 上，定义：

$$h_{\max}^2 = \max_{p\in\mathcal A}|h(p)|^2, \qquad g_{\max}^2 = \max_{p\in\mathcal A}|g(p)|^2 \tag{24}$$

归一化的目标和干扰功率剖面分别由下式给出：

$$H(p) = \frac{|h(p)|^2}{h_{\max}^2}, \qquad G(p) = \frac{|g(p)|^2}{g_{\max}^2}, \quad p\in\mathcal A \tag{25}$$

在此，$H(p)$ 和 $G(p)$ 仅用作位置域的筛选剖面。它们并不取代原始的最大输出 SINR 目标。相反，它们指示了在有限孔径内哪里更有可能出现强大的目标信道响应和微弱的干扰信道响应。最终的 MA 位置向量仍为 $\mathbf p=[p_1,\ldots,p_N]^T$，其性能由 $\Gamma^\star(\mathbf p)$ 进行评估。

给定目标强度阈值 $\tau_s$ 和干扰抑制阈值 $\tau_j$，近似的**双重良好区域**定义为：

$$\Omega_{\rm dual}(A;\tau_s,\tau_j) = \left\{ p\in\mathcal A: H(p)\ge \tau_s,\; G(p)\le \tau_j \right\} \tag{26}$$

位于 $\Omega_{\rm dual}(A;\tau_s,\tau_j)$ 内的位置在局部是良好的，因为它提供了相对较强的目标响应和相对较弱的干扰响应。然而，$\Omega_{\rm dual}(A;\tau_s,\tau_j)$ 只是一个物理引导的候选区域，而不是原始优化问题的最终可行集。选定 MA 集合的性能不仅取决于单点剖面 $H(p)$ 和 $G(p)$，还取决于 $\mathbf h(\mathbf p)$ 与 $\mathbf g(\mathbf p)$ 之间的向量级关系。

为了表征严格的双重良好区域是否足够大以支持多个 MA 阵元，我们将双重良好装箱数（packing number）定义为：

$$N_{\rm dual}^{\rm pack} = \max_{\mathcal S\subseteq \Omega_{\rm dual}(A;\tau_s,\tau_j)} |\mathcal S| \tag{27}$$

$$\text{约束条件为：} \quad |u-v|\ge d_{\min}, \quad \forall u,v\in\mathcal S,\; u\neq v \tag{28}$$

指标 $N_{\rm dual}^{\rm pack}$ 衡量了在最小间距约束下，严格的双重良好区域内物理上允许的最大 MA 位置数量。如果 $N_{\rm dual}^{\rm pack}<N$，则仅凭严格的双重良好区域无法容纳所有的 MA 阵元，此时需要放宽阈值 $\tau_s$ 和 $\tau_j$，或通过额外的选择标准进行补充。

然而，$N_{\rm dual}^{\rm pack}$ 仅考虑了最小物理间距 $d_{\min}$。它无法区分装箱后的位置是否提供了足够不同的信道采样。在双径情况下，目标和干扰功率剖面分别大约以空间周期 $T_s$ 和 $T_j$ 变化。因此，相距远小于这些周期的两个位置可能仍会经历高度相似的目标和干扰响应。

值得注意的是，$N_{\rm dual}^{\rm pack}$ 是一个几何可行性指标，而非可达最大输出 SINR 的直接度量。它仅统计了在最小间距约束下，物理上可以放置在 $\Omega_{\rm dual}$ 内的 MA 位置数量。然而，如果物理上允许的位置处于位置域剖面变化缓慢的部分，它们仍可能观测到相似的目标和干扰信道响应。在双径情况下，这种效应与空间周期 $T_s$ 和 $T_j$ 密切相关：当 these 周期与孔径或阵元间距相比结构较大时，信道响应在空间上的变化较为缓慢，多个允许的位置可能只能提供有限的额外空间可区分性。在通用多径信道中，叠加信道剖面的空间变化尺度也起着相同的作用。

因此，$N_{\rm dual}^{\rm pack}$ 应当仅被解释为有限孔径的几何可用性度量。最终的 MA 位置选择不应完全依赖于单点良好性，而应进一步考虑集合层面的目标信道强度、干扰抑制以及目标-干扰信道相关性。

## 3. 所提基于 FADLS 的位置选择

^sec-proposed-fadls

上一节表明，多径诱导的位置域剖面为 MA 位置选择提供了有用的物理引导。然而，公式 (26) 中的严格双重良好区域不应作为放置所有 MA 阵元的硬性约束，因为在有限孔径和最小间距约束下，它可能无法包含足够数量的物理允许位置。受到这一观察的启发，我们提出了一种有限孔径双重良好位置选择（FADLS）方法。其核心思想是将双重良好结构作为一种软性的候选排序原则，然后根据集合级指标选择一个可行的 MA 位置集。

### 3.1 双剖面候选排序

^subsec-dual-profile-ranking

为了数值实现，有限孔径 $\mathcal A$ 被离散采样为 $\mathcal A_\Delta=\{q_1,q_2,\ldots,q_K\}$，其中 $K$ 为候选网格点的数量。基于公式 (25) 中定义的归一化剖面 $H(p)$ 和 $G(p)$，我们首先为每个候选位置分配一个单点**双剖面得分**：

$$s_{\rm dual}(q_k) = H(q_k)\bigl(1-G(q_k)\bigr), \quad q_k\in\mathcal A_\Delta \tag{29}$$

较大的 $s_{\rm dual}(q_k)$ 表明位置 $q_k$ 具有更强的目标响应和更弱的干扰响应。与公式 (26) 中的严格双重良好区域不同，公式 (29) 没有对候选位置施加硬性阈值。相反，它根据候选位置的局部双重良好程度对整个有限孔径上的所有位置进行排序。这种软排序机制避免了当严格的双重良好区域无法容纳所有 $N$ 个 MA 阵元时可能出现的不可行问题。

### 3.2 集合级 FADLS 指标

^subsec-set-level-metric

双剖面得分仅单独评估每个位置。然而，公式 (12) 中的输出 SINR 取决于所选 MA 位置向量的整体。因此，需要一个集合级指标来评估临时选出的位置集的质量。

令 $\mathcal S=\{u_1,u_2,\ldots,u_M\}\subseteq\mathcal A_\Delta$ 表示一个包含 $M\le N$ 个位置的临时选定位置集。对应的目标信道向量和干扰信道向量定义为：

$$\begin{aligned} \mathbf h_{\mathcal S} &= [h(u_1),h(u_2),\ldots,h(u_M)]^T, \\ \mathbf g_{\mathcal S} &= [g(u_1),g(u_2),\ldots,g(u_M)]^T. \end{aligned} \tag{30}$$

在 $\mathcal S$ 上的平均目标剖面值和平均干扰剖面值定义为：

$$\overline H(\mathcal S) = \frac{1}{M}\sum_{u\in\mathcal S}H(u), \qquad \overline G(\mathcal S) = \frac{1}{M}\sum_{u\in\mathcal S}G(u) \tag{31}$$

为了表征所选位置集上目标信道向量与干扰信道向量之间的空间可分性，我们定义：

$$\rho_{hg}(\mathcal S) = \frac{ |\mathbf h_{\mathcal S}^H\mathbf g_{\mathcal S}|^2 }{ \|\mathbf h_{\mathcal S}\|^2 \|\mathbf g_{\mathcal S}\|^2+\epsilon } \tag{32}$$

其中 $\epsilon$ 是用于数值稳定性的微小正常数。较小的 $\rho_{hg}(\mathcal S)$ 表明在选定的 MA 位置上，目标信道向量与干扰信道向量之间的相关性较低。

由此，所提**集合级 FADLS 指标**由下式给出：

$$\mathcal J(\mathcal S) = \overline H(\mathcal S) \bigl(1-\overline G(\mathcal S)\bigr) \bigl(1-\rho_{hg}(\mathcal S)\bigr) \tag{33}$$

公式 (33) 中的三个因子具有明确的物理意义。第一项鼓励所选位置提供强大的目标响应。第二项降低了具有强干扰响应的位置集的优先级。第三项鼓励更低的目标-干扰信道相关性，这有助于接收机从干扰中区分出目标信号。需要说明的是，$\mathcal J(\mathcal S)$ 仅用作低复杂度的物理引导选择指标。最终的性能仍由最大输出 SINR $\Gamma^\star(\mathbf p)$ 进行评估。

### 3.3 贪婪 FADLS 位置选择

^subsec-greedy-fadls-algorithm

基于双剖面排序和集合级指标，所提 FADLS 方法以贪婪的方式选择 MA位置。在每一步中，一个候选位置被试探性地添加到已选集合中。该候选位置必须与已选位置满足最小间距约束，并且剩余孔径必须仍能容纳未选的 MA 阵元。在所有符合条件的候选位置中，选择使公式 (33) 中集合级指标最大化的那一个。

> [!info] **算法 1：基于 FADLS 的贪婪 MA 位置选择**
> 
> **输入**：候选网格 $\mathcal A_\Delta=\{q_1,\ldots,q_K\}$；剖面 $H(q_k)$，$G(q_k)$；信道采样 $h(q_k)$，$g(q_k)$；MA 数量 $N$；最小间距 $d_{\min}$。
> 
> **输出**：选定的 MA 位置集 $\mathcal S$。
> 
> 1. 对所有 $q_k\in\mathcal A_\Delta$ 计算 $s_{\rm dual}(q_k)=H(q_k)(1-G(q_k))$。
>     
> 2. 将所有候选位置按 $s_{\rm dual}(q_k)$ 降序排列，排序后的列表记为 $\mathcal C$。
>     
> 3. 初始化 $\mathcal S \leftarrow \emptyset$。
>     
> 4. **FOR** $n = 1, \ldots, N$ **DO**
>     
> 5. $\quad$ 设置 $J_{\max} \leftarrow -\infty$ 且 $q^\star \leftarrow \emptyset$。
>     
> 6. $\quad$ **FOR** 列表 $\mathcal C$ 中的每个 $q$ **DO**
>     
> 7. $\quad\quad$ **IF** $q \in \mathcal S$ **THEN** continue
>     
> 8. $\quad\quad$ 形成临时集合 $\mathcal S_q = \mathcal S \cup \{q\}$。
>     
> 9. $\quad\quad$ **IF** $\mathcal S_q$ 违反了最小间距约束 **THEN** continue
>     
> 10. $\quad\quad$ **IF** $\mathcal S_q$ 无法在 $\mathcal A_\Delta$ 内补全为 $N$ 个可行位置 **THEN** continue
>     
> 11. $\quad\quad$ 根据公式 (33) 计算 $\mathcal J(\mathcal S_q)$。
>     
> 12. $\quad\quad$ **IF** $\mathcal J(\mathcal S_q) > J_{\max}$ **THEN**
>     
> 13. $\quad\quad\quad$ $J_{\max} \leftarrow \mathcal J(\mathcal S_q)$
>     
> 14. $\quad\quad\quad$ $q^\star \leftarrow q$
>     
> 15. $\quad\quad$ **END IF**
>     
> 16. $\quad$ **END FOR**
>     
> 17. $\quad$ 更新 $\mathcal S \leftarrow \mathcal S \cup \{q^\star\}$。
>     
> 18. **END FOR**
>     
> 19. 将 $\mathcal S$ 按升序排序并输出。
>     

算法 1 的贪婪特性使所提方法在计算上非常有吸引力。FADLS 不是直接在所有可行的位置组合上进行搜索，而是首先根据双剖面得分对候选位置进行排序，然后执行集合级贪婪选择。因此，它能够以极少的目标函数评估次数提供高质量的初始 MA 位置集。

### 3.4 FADLS 初始化的 BCD/AO 微调

^subsec-fadls-initialized-bcd

FADLS 解可以直接用作最终的 MA 位置向量。它也可以作为局部微调的物理引导初始化。在本文中，我们考虑将有限预算的 BCD/AO（块坐标下降/交替优化）方法作为可选的第二阶段。该方法从初始位置向量开始，每次更新一个 MA 位置，同时保持其他 $N-1$ 个位置固定。对于当前更新的 MA 阵元，测试满足最小间距约束的可行网格点，并保留提供最大 $\Gamma^\star(\mathbf p)$ 的位置。当预设的目标函数评估预算耗尽时，微调停止。

在数值结果中，对比了两种初始化策略：

- **随机初始化的 BCD/AO**：从随机生成的临时可行位置向量开始。
    
- **FADLS 初始化的 BCD/AO**：从算法 1 获得的位置向量开始。
    

该对比用于评估所提物理引导的 FADLS 方法能否为有限预算下的局部 SINR 微调提供更好的初始点。基础 FADLS 方法并不强制要求进行 BCD/AO 微调，但这证明了 FADLS 可以与现有的局部搜索方法相结合以提高搜索效率。

### 3.5 复杂度讨论

^subsec-complexity-discussion

令 $K=|\mathcal A_\Delta| $ 表示候选网格点的数量。对所有可行 MA 位置集进行穷举搜索的复杂度对于 $K$ 和 $N$ 呈组合级增长。相比指下，所提 FADLS 方法首先对 $K$ 个候选点进行排序，这需要 $\mathcal O(K\log K)$次操作。

在贪婪选择阶段，在 $N$ 个选择步骤的每一步中，最多检查 $K$ 个候选点。由于集合级指标是在临时选定的集合上计算的，因此整体复杂度随 $K$ 和 $N$ 呈**多项式级增长**，而非组合级增长。

此外，公式 (33) 中的 FADLS 指标避免了在初始选择阶段对所有候选位置组合重复评估原始的最大输出 SINR。由于可选的 BCD/AO 微调受到目标函数评估预算的限制，因此其复杂度是可控的。总之，所提方法在物理可解释性、计算复杂度和输出 SINR 性能之间提供了一种务实的折衷。

## 4. 数值结果

^sec-numerical-results

在本节中，提供了数值结果以验证所提有限孔径双重良好位置选择（FADLS）方法。仿真旨在验证以下几个方面：

1. 由多径传播引起的位置域双重良好结构。
    
2. 严格双重良好位置在有限孔径下的可用性。
    
3. 面向抗干扰的 MA 位置选择在通用多径信道下的性能。
    
4. 将 FADLS 用作物理引导初始化时在有限预算下的搜索效率。
    

除非另有说明，我们考虑一个包含 $N=6$ 个阵元且最小间距为 $d_{\min}=0.5\lambda$ 的一维 MA 阵列。为了数值实现，连续孔径以 $\Delta p=\lambda/100$ 的分辨率进行采样。目标信号的发射功率设置为 $P_s=1$。所有位置选择方案的性能均由公式 (12) 中相同的最大输出 SINR $\Gamma^\star(\mathbf p)$ 评估。

对比方案总结如下：

- **固定 ULA**：将天线均匀放置在孔径内。
    
- **朴素双剖面（Naive Dual-Profile）**：仅根据单点得分 $H(p)(1-G(p))$ 进行贪婪选择。
    
- **所提 FADLS**：在选择 MA 位置向量时，进一步考虑了集合层面的目标-干扰信道相关性。
    
- **随机搜索（Random Search）**：直接在给定预算下评估随机可行解的 $\Gamma^\star(\mathbf p)$。
    
- **随机初始化 BCD/AO** / **FADLS 初始化 BCD/AO**。
    

### 4.1 位置域双重良好结构

我们首先考虑一个典型的双径场景。`Fig. 1` 显示了在有限孔径上的归一化目标剖面 $H(p)$、干扰微弱响应剖面 $1-G(p)$ 以及双剖面得分 $H(p)(1-G(p))$。从结果可以看出，目标信道和干扰信道在孔径上表现出不同的空间变化。FADLS 位置主要位于双剖面得分的高值区域周围，这证实了所提方法利用了多径诱导的位置域结构，而不是在孔径上盲目搜索。

### 4.2 有限孔径双重良好可用性

我们检查了严格的双重良好区域是否能容纳多个 MA 阵元。`Fig. 2` 展示了平均双重良好装箱数 $N_{\rm dual}^{\rm pack}$ 随孔径大小的变化情况。结果表明，在许多情况下（特别是当孔径有限时）$N_{\rm dual}^{\rm pack}$ 仍低于所需的 MA 数量 $N$。这表明仅凭严格的双重良好区域不足以放置所有天线，从而验证了 FADLS 采用**软排序评分**加**集合级指标**的必要性。

### 4.3 通用多径信道下的性能

在通用多径信道下（路径数 $L_s=L_j=4$），`Fig. 3` 展示了平均最大输出 SINR 随孔径大小的变化情况。

- 所有基于 MA 的方案相比于固定 ULA 均带来了显著的性能提升。
    
- 随着孔径的增大，可利用的位置域机会增多，输出 SINR 相应提高。
    
- 所提 FADLS 显著优于朴素双剖面方案，而结合了 BCD/AO 微调后能持续逼近最优表现。
    

### 4.4 有限预算下的搜索效率

`Fig. 4` 展示了平均最大输出 SINR 随目标函数评估预算 $B$ 的变化情况。

- 随机搜索与随机初始化的 BCD/AO 对初始点较为敏感，在小预算范围内表现受限。
    
- **FADLS 初始化的 BCD/AO** 方案在相同评估预算下大幅超越了随机初始化的对比方案。这证实了所提 FADLS 方法通过利用多径诱导的空间结构，显著提高了有限预算下的搜索效率。
    

## 5. 作者简介部分

> [!tip] **排版提示**
> 
> 如果您有 EPS/PDF 格式的照片，在 biography 的可选参数内容周围需要额外的花括号，以防止 $\LaTeX$ 解析器混淆。

**如果您附带照片**：

_Michael Shell_：使用 `\begin{IEEEbiography}` 并链接照片，后接简介文本。

**如果您不附带照片**：

_John Doe_：使用 `\begin{IEEEbiographynophoto}`，后接简介文本。