
tree = connected acyclic graph
spanning tree = subset of edges of Graph that form a tree &hit all vertices of G
MST: MST 不是找最短的一条边，而是找一组边，让所有点连起来，并且总权重最小。
# 💡 基于 Greedy 策略的天线选点与 MST 算法对比笔记

## 📌 核心逻辑线索

> **核心结论**：天线位置选择算法与图论中的最小生成树（MST）算法在**数学对象上并不等价**，但它们在**算法设计模式（Greedy Template）**上具有高度的相似性。它们都属于在特定约束下，通过局部最优解逼近或达到全局最优解的尝试。

## 一、 我的算法核心：SINR-Greedy 天线选点

在天线位置选择问题（`sinr_greedy_select_fullgrid.m`）中，算法的核心逻辑是**逐个点进行贪心搜索**。

### 1. 算法伪代码流向

% 初始化：已选天线集合为空，候选网格点为全部 FullGrid

for n = 1 : N_antennas

for 每一个候选网格位置 p_try

% 1. 可行性检查 (Feasibility Check)

if any(abs(p_try - p_selected) < d_min)

skip; % 距离太近，违反物理约束，跳过

end

```
    % 2. 局部性能评估 (Local Evaluation)
    计算加入 p_try 后的系统 SINR: \Gamma({p_1, ..., p_{n-1}, p_try})
end

% 3. 贪心选择 (Greedy Selection)
选择让 SINR 最大的那个位置 $p_n^\star$ 加入已选集合
```

end

### 2. 数学表达

$$p_n^\star = \arg\max_{p \in \mathcal{P}_{\text{feasible}}} \Gamma(\{p_1, \ldots, p_{n-1}, p\})$$

- **保证**：每一步在当前条件下选得最好，且完全满足最小间距 $d_{\min}$ 的可行性约束。
    
- **局限**：**不保证最终组合是全局最优的**。可能第 1 步选 A 看起来最好，但如果第 1 步选 B，后面和 C、D 组合起来总体 SINR 反而更高。
    

## 二、 拓宽视野：什么是 Spanning Tree（生成树）与 MST？

为了理解这种“贪心选择”的本质，我们引入图论中经典的 **MST（最小生成树）** 问题作为对照。

### 1. 核心定义拆解

- **Tree（树）**：
    
    $$\text{Tree} = \text{Connected（连通）} + \text{Acyclic（无环）}$$
    
    - _正例_：`A —— B —— C` （所有点连通，没有绕成圈的路径）
        
    - _反例_：`A —— B —— C —— A` （形成了一个环，不是树）
        
- **Spanning Tree（生成树）**：
    
    包含原图 $G$ 中**所有顶点**（Hit all vertices），但只保留一部分边，使其构成一棵树的子图。
    
- **MST（最小生成树）**：
    
    在所有合法的生成树中，**边权重总和最小**的那棵树。
    
    $$\text{Minimize } \sum_{e \in T} w(e)$$
    

## 三、 深度映射：天线选点与 MST Greedy 算法的互通性

虽然面对的问题不同，但 MST 的经典贪心算法（Kruskal 和 Prim）与天线选点算法在结构上如出一辙：

### 1. 算法角色对照表

|**算法组件**|**🌲 最小生成树 (MST) 问题**|**📡 天线网格选点 (SINR-Greedy)**|
|---|---|---|
|**已选集合**|已选入树的边集合 $T$|已确定的天线物理位置集合 $p_{\text{selected}}$|
|**新候选项**|原图中的一条未知边 $e$|网格中的一个待测试位置 $p_{\text{try}}$|
|**可行性约束**<br><br>  <br><br>_(Feasibility)_|**不能成环**<br><br>  <br><br>(Kruskal 靠判环，Prim 靠只连向树外)|**不能违反最小间距 $d_{\min}$**<br><br>  <br><br>(`any(abs(p_try - p_selected) < d_min)`)|
|**选择标准**<br><br>  <br><br>_(Metric)_|边的权重尽量小 ($w(e) \rightarrow \min$)|系统的 SINR 尽量大 ($\Gamma \rightarrow \max$)|

### 2. Kruskal 与 Prim 是如何“防成环”的？

- **Kruskal 算法**：把边按权重从小到大排序，每次选最小的边。如果加入后**发现会成环就直接跳过**（这对应你代码里的 `continue`）。
    
- **Prim 算法**：从一个点出发，每次只在“树内的点 $\rightarrow$ 树外的点”的候选边里选最小的。因为**一只脚在树内，一只脚在树外**，所以天生不可能在树内部形成环。
    

## 四、 关键差异：为什么 MST 能到全局最优，而我的天线算法不能？

这是评价你论文算法性能时的核心理论支撑：

> [!danger] **Greedy 策略的全局最优性分水岭**
> 
> **Greedy 本身只是一种逐步选择的“策略”，并不天然保证最终结果是全局最优。**

1. **MST 问题（Kruskal / Prim）**：
    
    - **结论**：能保证最终得到的是全局最优解。
        
    - **原因**：MST 问题具有特殊的数学结构——**割性质（Cut Property）**。它保证了“局部安全的最优边”最终一定能连成全局最优树。
        
2. **天线选点问题（SINR-Greedy）**：
    
    - **结论**：**不能保证**全局最优解。
        
    - **原因**：无线信道的容量/SINR 具有非线性和强耦合性。前面选的位置会严重影响后面位置的信道独立性，不满足割性质。
        

### 📝 论文写作定位提示

在论文中，不要把 SINR-Greedy 吹捧为完美的最优化算法。最严谨的学术定位是：

> **"A strong grid-based greedy baseline."**
> 
> （一个基于网格的强贪心基准算法：它每一步直接对物理指标进行局部优化，性能优异且能绝对满足 $d_{\min}$ 约束，但受限于贪心机制的局限性，通常作为高性能的近似解基准。）