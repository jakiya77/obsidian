---
tags:
  - Academic/Research_Notes
  - Wireless_Communication/Movable_Antenna
  - Anti_Jamming/FADLS
date: 2026-06-18
---

# Multi-path Spatial Fluctuation & Grid Density Derivation

## 📌 Core Conclusion
> [!abstract] **最核心的物理本质**
> **2-ray 模型是该推导的基石，而多径（Multipath）本质上是多组“2-ray 交叉项”的空间叠加。**
> 多径数量 $L$ 的增加会使位置域（Position Domain）的起伏结构变得更丰富、更复杂，但其**最快空间变化周期**（上限）完全由**最大路径角度差**（$\max |\sin\theta_\ell - \sin\theta_m|$）决定，而不会凭空产生无限快的空间变化。

---

## 1. 基础模型：2-ray 空间拍频（Spatial Beating）

假设期望信道仅包含 2 条路径：
$$h(p) = \alpha_1 e^{-j k_0 p \sin\theta_1} + \alpha_2 e^{-j k_0 p \sin\theta_2}$$
其中波数 $k_0 = \frac{2\pi}{\lambda}$。其位置域功率分布为：
$$H(p) = |h(p)|^2 = |\alpha_1|^2 + |\alpha_2|^2 + 2\operatorname{Re}\left[ \alpha_1\alpha_2^* e^{-j k_0 p(\sin\theta_1 - \sin\theta_2)} \right]$$

* **空间起伏源：** 最后一项交叉项 $e^{-j\frac{2\pi}{\lambda}p(\sin\theta_1 - \sin\theta_2)}$ 导致了位置域的功率起伏。
* **空间周期（Spatial Period）：**
    $$T_{12} = \frac{\lambda}{|\sin\theta_1 - \sin\theta_2|}$$
    在 2-ray 情况下，功率分布呈现出规则的**“空间拍频”**现象。

---

## 2. 推广模型：多径（Multipath）多重交叉项叠加

当路径数增加至 $L$ 条时：
$$h(p) = \sum_{\ell=1}^{L} \alpha_\ell e^{-j k_0 p \sin\theta_\ell}$$
展开其位置域功率：
$$H(p) = |h(p)|^2 = \sum_{\ell=1}^{L}|\alpha_\ell|^2 + \sum_{\ell \neq m} \alpha_\ell\alpha_m^* e^{-j k_0 p(\sin\theta_\ell - \sin\theta_m)}$$

* **多径本质：** 展开式后半部分包含了 $\frac{L(L-1)}{2}$ 组不同的路径对。
* **空间波形：** 多径下的位置域起伏，实质上是**诸多空间周期不同的正弦波线性叠加**的结果，波形更复杂，但没有引入新的频域分量。

---

## 3. 边界分析：最快变化周期上限

任意两条路径 $(\ell, m)$ 产生的空间周期为：
$$T_{\ell m} = \frac{\lambda}{|\sin\theta_\ell - \sin\theta_m|}$$
要估计位置域变化的最快极限（即周期的下限 $T_{\min}$），只需寻找分母的最大值。

### 📐 结合具体仿真参数计算
若路径角度范围限制在 $\theta \in [-60^\circ, 60^\circ]$，则最大正弦差为：
$$|\sin\theta_\ell - \sin\theta_m|_{\max} = \sin 60^\circ - \sin(-60^\circ) = 0.866 - (-0.866) = 1.732$$
因此，信道功率分布 $H(p)$ 的**最快起伏周期**为：
$$T_{\min} \approx \frac{\lambda}{1.732} \approx 0.577\lambda$$

---

## 4. 衍生拓展：Dual Score（双重得分）的最快周期

在 FADLS 算法中，选点依据通常基于 Dual Score：
$$F(p) = H(p)\left(1 - G(p)\right)$$
其中 $H(p)$ 为期望信道功率，$G(p)$ 为干扰信道功率。

> [!warning] **组合频率效应**
> 由于 $F(p)$ 是两个各自包含空间频率项的剖面（Profile）相乘，在频域上相当于**卷积**，理论上会产生更细碎的**组合高频项**（即和频）。

* **保守估计下的最快周期：**
    $$T_{\min, F} \approx \frac{\lambda}{2 \times 1.732} \approx 0.289\lambda$$
* **采样密度验证（以网格粒度 $\Delta p = \lambda/100$ 为例）：**
    一个最快周期内包含的采样点数为：
    $$\text{Points} = \frac{0.289\lambda}{0.01\lambda} \approx 29 \text{ 个点}$$
    **结论：** 即使考虑了 Dual Score 的组合高频，$\lambda/100$ 的网格精度依然能够提供极其细腻且不易失真的空间采样（每周期近 30 个点，远超奈奎斯特采样定律要求）。

---

## 5. 论文表述与实验设计（Paper Writing & Exp Design）

### 📝 论文英文学术表述（可直接用于 Manuscript）
> "The spatial variation of the channel power profile is governed by the pairwise path-difference terms. For an $L$-path channel, the power profile contains cross terms with spatial periods $T_{\ell m}=\lambda/|\sin\theta_\ell-\sin\theta_m|$. Therefore, the fastest position-domain fluctuation is determined by the maximum angular separation among the paths rather than by the number of paths alone."
> 
> *（**中文释义：** 多径信道功率在位置域中的变化由路径两两之间的交叉项决定。对于 $L$ 条路径，功率分布包含周期为 $T_{\ell m}=\lambda/|\sin\theta_\ell-\sin\theta_m|$ 的交叉项。因此，最快的位置域起伏主要由路径最大角度间隔决定，而不是单纯由路径数量决定。）*

### 📊 实验设计支撑（网格敏感性分析 Grid Sensitivity）
该推导完美解释了为什么不需要无限细化网格。在进行网格敏感性仿真时，可预期如下收敛趋势：
* $\lambda/5 \to \lambda/20$：一个周期内点数过少（1~3点），性能随网格变细而**显著提升**。
* $\lambda/20 \to \lambda/50$：基本捕捉到主要空间结构，性能**继续提升但趋于平缓**。
* $\lambda/50 \to \lambda/100$：空间采样已足够稠密，性能**达到饱和**。
* $\lambda/100 \to \lambda/200$：超出实际物理波动需求，性能增益微乎其微，唯有**计算复杂度陡增**。