### 复变量求导法则
##### 1. 原理：在求导时，把复数向量 $\mathbf{w}$ 和它的共轭转置 $\mathbf{w}^H$ 当作两个完全独立、互不相干的变量。
##### 2. 求导规则

假设 $\mathbf{w}$ 是变量向量，$\mathbf{A}$ 是常数方阵，$\mathbf{c}$ 是常数向量。我们现在要对 $\mathbf{w}^H$ 求偏导：

- **规则 1（线性项）**：
    
    如果式子是 $\mathbf{w}^H \mathbf{c}$。因为 $\mathbf{w}^H$ 是一次项，求导后 $\mathbf{w}^H$ 消失，只剩下后面的常数。
    
    $$\frac{\partial (\mathbf{w}^H \mathbf{c})}{\partial \mathbf{w}^H} = \mathbf{c}$$
    
- **规则 2（二次项）**：
    
    如果式子是 $\mathbf{w}^H \mathbf{A} \mathbf{w}$。把 $\mathbf{w}$ 当作常数，$\mathbf{w}^H$ 是一次项，求导后 $\mathbf{w}^H$ 消失，剩下后面的部分。
    
    $$\frac{\partial (\mathbf{w}^H \mathbf{A} \mathbf{w})}{\partial \mathbf{w}^H} = \mathbf{A} \mathbf{w}$$
    
- **规则 3（无关项）**：
    
    如果式子是 $\mathbf{c}^H \mathbf{w}$。注意！这里面**没有** $\mathbf{w}^H$！因为我们前面说了，$\mathbf{w}$ 和 $\mathbf{w}^H$ 是两个独立变量。既然这里没有 $\mathbf{w}^H$，那它对 $\mathbf{w}^H$ 来说就是一个常数。常数求导等于 0。
    
    $$\frac{\partial (\mathbf{c}^H \mathbf{w})}{\partial \mathbf{w}^H} = \mathbf{0}$$

##### 3. 具体运用例子
原始公式：

$$\mathcal{L} = \underbrace{\mathbf{w}^H \left( \frac{\rho}{2} \mathbf{I} - \mathbf{H} \right) \mathbf{w}}_{\text{第 1 部分}} \quad \underbrace{- \frac{\rho}{2} \mathbf{w}^H (\mathbf{v} - \mathbf{u})}_{\text{第 2 部分}} \quad \underbrace{- \frac{\rho}{2} (\mathbf{v} - \mathbf{u})^H \mathbf{w}}_{\text{第 3 部分}} + \text{常数}$$

**对 $\mathbf{w}^H$ 求导实战：**

1. **看第 1 部分**：套用**规则 2**（这里的 $\mathbf{A}$ 就是括号里那一坨常数矩阵）。
    
    求导结果：$\left( \frac{\rho}{2} \mathbf{I} - \mathbf{H} \right) \mathbf{w}$
    
2. **看第 2 部分**：套用**规则 1**（这里的 $\mathbf{c}$ 就是括号里那一坨常数向量 $\mathbf{v} - \mathbf{u}$，前面的常系数 $-\frac{\rho}{2}$ 照抄）。
    
    求导结果：$-\frac{\rho}{2} (\mathbf{v} - \mathbf{u})$
    
3. **看第 3 部分**：套用**规则 3**。这部分里根本没有 $\mathbf{w}^H$，全是常数和 $\mathbf{w}$。
    
    求导结果：$\mathbf{0}$
    
4. 后面的“常数”求导当然也是 $\mathbf{0}$。
    

把上面求出来的结果拼在一起，令它等于 0，就完美得到了代码里的那行核心推导：

$$\frac{\partial \mathcal{L}}{\partial \mathbf{w}^H} = \left( \frac{\rho}{2} \mathbf{I} - \mathbf{H} \right) \mathbf{w} - \frac{\rho}{2} (\mathbf{v} - \mathbf{u}) = \mathbf{0}$$
$$\mathbf{w} = \mathbf{A}^{-1} \mathbf{b}$$
```matlab 
 w = (rho/2 * eye(N) - H) \ (rho/2 * (exp(1i*phi) - u));
```
 > MATLAB 中的 `\`（左除）和 `/`（右除）
 > - **左除 `\` (mldivide)**：
    
    `A \ b` 在数学上等价于 $\mathbf{A}^{-1} \mathbf{b}$（即 `inv(A) * b`）。
    
    它专门用来解 $\mathbf{A}\mathbf{x} = \mathbf{b}$ 这种未知数在**右边**的方程。
    
- **右除 `/` (mrdivide)**：
    
    `b / A` 在数学上等价于 $\mathbf{b} \mathbf{A}^{-1}$（即 `b * inv(A)`）。
    
    它专门用来解 $\mathbf{x}\mathbf{A} = \mathbf{b}$ 这种未知数在**左边**的方程。
[[Math：矩阵条件数（Condition Number）]]