---
tags:
  - Wireless-Communication
  - Movable-Antenna
  - Algorithm-Complexity
  - FADLS
date: 2026-06-14
---

# 动天线选择算法复杂度分析笔记 (FADLS vs. SINR-Greedy)

## 1. 基础参数定义

设离散化和筛选相关的基本物理量如下：
* **$M$**：全部离散网格点数量（Full Grid Size）。
* **$B$**：FADLS 预筛选候选池大小（Candidate Pool Size）。
* **$N$**：待选取的动天线数量（Number of Selected MAs）。

> [!NOTE] 典型仿真参数 (Typical Setup)
> * 天线线阵长度：$A = 8\lambda$
> * 网格步长：$\text{grid\_step} = \lambda/100$
> * 全网格点数：$M = \frac{A}{\text{grid\_step}} + 1 \approx 801$
> * FADLS 候选池大小：$B = 300$
> * 动天线数量：$N = 6$

---

## 2. 算法复杂度理论推导

### 核心计算开销对比

| 算法 / 步骤 | 搜索范围 | 单次评分复杂度 | 总体时间复杂度 |
| :--- | :--- | :--- | :--- |
| **Desired-only / Jammer-only** | 全网格 ($M$) | $O(1)$ 或 $O(n)$ (间距检查) | $O(MN)$ |
| **FADLS-heuristic** | 候选池 ($B$) | $O(n)$ | $\mathbf{O(M\log M + BN^2)}$ |
| **FADLS-screened SINR** | 候选池 ($B$) | $O(n^3)$ | $O(M\log M + BN^4)$ |
| **Full-grid SINR-greedy** | 全网格 ($M$) | $O(n^3)$ | $\mathbf{O(MN^4)}$ |

---

### 2.1 FADLS-heuristic 复杂度

FADLS-heuristic 的启发式评分函数为：
$$\text{score\_fadls} = \alpha_w\log(\text{mean}(H)) + \beta_w\log(1 - \text{mean}(G)) + \gamma_w\log(1 - \rho_{hg})$$
其中最主要的计算项为空间相关性：
$$\rho_{hg} = \frac{|\mathbf{h}_{\text{tmp}}^H\mathbf{g}_{\text{tmp}}|^2}{\|\mathbf{h}_{\text{tmp}}\|^2\|\mathbf{g}_{\text{tmp}}\|^2 + \epsilon}$$

* **单次评分复杂度**：当已选定 $n-1$ 根天线，试探第 $n$ 根时，向量长度为 $n$。向量内积与范数计算的复杂度均为 $O(n)$。
* **贪心搜索开销**：每一轮最多扫描 $B$ 个候选点，选满 $N$ 根天线：
  $$\sum_{n=1}^{N} B \cdot O(n) = O(BN^2)$$
* **排序开销**：FADLS 初始阶段需要对全网格进行双重有利度排序：
  $$\text{score\_naive} = H \cdot (1 - G) \implies O(M\log M)$$

> [!SUCCESS] 结论
> FADLS-heuristic 的总体复杂度为：
> $$\boxed{O(M\log M + BN^2)}$$
> *注：若候选池中因最小间距约束无法选满天线，最坏情况会退化到全网格搜索，复杂度变为 $O(M\log M + MN^2)$，但实际仿真中 $B=300$ 足够大，基本在第一轮即可完成。*

---

### 2.2 Full-grid SINR-greedy 复杂度

作为公平对比的标准基准（Benchmark），传统的 SINR-greedy 每一轮在全网格 $M$ 上扫描，并计算 post-MMSE SINR：
$$\mathbf{R}_{\text{tmp}} = P_j(\mathbf{g}_{\text{tmp}}\mathbf{g}_{\text{tmp}}^H) + \sigma^2\mathbf{I}_n$$
$$\text{score\_tmp} = \Re\left(P_s\left(\mathbf{h}_{\text{tmp}}^H (\mathbf{R}_{\text{tmp}}\backslash\mathbf{h}_{\text{tmp}})\right)\right)$$

* **单次评分复杂度**：使用 MATLAB 默认的线性方程组求解器（`\` 算子）求解 $n \times n$ 的系统 $\mathbf{R}_{\text{tmp}}\backslash\mathbf{h}_{\text{tmp}}$，复杂度为 $O(n^3)$。
* **贪心搜索开销**：每一步扫描 $M$ 个点，选满 $N$ 根天线：
  $$\sum_{n=1}^{N} M \cdot O(n^3) = O(MN^4)$$

> [!SUCCESS] 结论
> 传统的公平全网格 SINR-greedy 复杂度为：
> $$\boxed{O(MN^4)}$$

---

## 3. 数值量级直观估计 (Quantitative Estimation)

带入典型参数 $M \approx 801$, $B = 300$, $N = 6$，仅对比核心评分步骤的计算次数量级：

* **FADLS-heuristic 核心评分开销**：
  $$B \sum_{n=1}^{6} n = 300 \times 21 = 6,300$$
* **Full-grid SINR-greedy 核心评分开销**：
  $$M \sum_{n=1}^{6} n^3 = 801 \times 441 = 353,241$$

> [!INFO] 量级对比结果
> 在当前参数下，仅从理论计算步数来看，Full-grid SINR-greedy 已经是 FADLS-heuristic 的 **$\approx 56$ 倍**。
> 在实际 MATLAB 运行中，由于矩阵求逆/线性求解带来的复杂内存开销和硬件指令常数，实际运行耗时差距会更加显著。

---

## 4. 论文防御性论点：关于 Rank-one 优化的说明

> [!WARNING] 审稿人潜在提问 (Reviewer Defense)
> 审稿人可能会提出：“由于干扰协方差矩阵 $\mathbf{R} = P_j\mathbf{g}\mathbf{g}^H+\sigma^2\mathbf{I}$ 具有典型的 **Rank-one 加上白噪声** 的结构，理论上可以通过**矩阵求逆引理 (Woodbury Matrix Identity)** 将求逆复杂度降至 $O(n)$，从而使 SINR-greedy 的整体复杂度降为 $O(MN^2)$。”

**论文对应的回应/写作策略：**
1. **承认理论上限**：承认该结构可以被利用进行接收机层面的优化。
2. **强调搜索范围的本质区别**：即便单个点的 SINR 计算被优化到了 $O(n)$，SINR-greedy 仍然不可避免地需要**在每一步扫描全部 $M$ 个网格点**。而 FADLS 的核心优势在于通过双重有利度预筛选，将搜索空间缩减到了 $B$（且 $B \ll M$）。
3. **立足当前实现**：在论文中明确指出，本文作为 Benchmark 采用的是直接的矩阵线性求解实现。

---

## 5. 论文标准英文表述 (Draft for Paper)

可在论文的 *Complexity Analysis* 章节直接使用以下学术文本：

```text
Let M denote the number of discretized antenna positions, B the size of the 
FADLS candidate pool, and N the number of selected movable antennas. The 
proposed FADLS first ranks the grid points according to the dual-favorable 
score, which requires O(M log M) operations. During greedy selection, only B 
candidate positions are evaluated at each antenna-selection step. Since the 
heuristic score only involves vector inner products and norm calculations over 
the currently selected antennas, its total greedy-selection complexity is 
O(BN^2). Therefore, the overall complexity of FADLS is O(M log M + BN^2).

In contrast, the full-grid SINR-greedy benchmark evaluates the post-MMSE 
output SINR over all M candidate positions at each selection step. With direct 
matrix solving, the SINR evaluation for an n-element temporary array requires 
O(n^3), resulting in an overall complexity of O(MN^4). For a direct MATLAB 
implementation using matrix inversion or linear solving, the SINR-greedy 
benchmark requires significantly higher computational overhead. Although the 
rank-one structure of the single-jammer covariance can theoretically be exploited 
via the Woodbury matrix identity to reduce the per-point SINR evaluation to O(n), 
the SINR-greedy approach still mandates scanning the full grid at each step, 
whereas the proposed FADLS operates on a significantly reduced candidate pool.