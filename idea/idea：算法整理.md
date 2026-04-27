最终的接收信号：$\hat{X} = \beta \mathbf{w}^H \mathbf{\Phi} \mathbf{Y}$

第一步迭代过程：固定$\beta$ 和$\mathbf{w}$   
目标函数：$\min_{\tilde{\mathbf{w}}} \| \mathbf{X}_{target} - \tilde{\mathbf{w}}^H \mathbf{Y}_{eq} \|_F^2$
优化手段：闭式最小二乘解 (Closed-form Least Squares via Pseudoinverse)
> $\tilde{\mathbf{w}} = (\mathbf{Y}_{eq} \mathbf{Y}_{eq}^H)^{-1} \mathbf{Y}_{eq} \mathbf{X}_{target}^H$
> $\tilde{\mathbf{w}} = (\mathbf{Y}_{eq}^H)^{\dagger} \mathbf{X}_{target}^H$

第二步迭代过程： 固定 $\beta$ 和 $\mathbf{w}$，优化模拟投影矩阵 $\mathbf{\Phi}$
目标函数：$\min_{\mathbf{\Phi}} \| \mathbf{X}_{target} - \beta \mathbf{w}^H \mathbf{\Phi} \mathbf{Y} \|_F^2$
约束条件：$|\Phi_{m,n}| \le 1, \quad \forall m,n$
优化手段： 投影梯度下降法 (Projected Gradient Descent, PGD)
具体迭代公式：

>1. 计算当前误差矩阵 (Error Computation)：$\mathbf{E} = \beta \mathbf{w}^H \mathbf{\Phi}^{(t)} \mathbf{Y} - \mathbf{X}_{target}$
>2. 计算目标函数关于 $\mathbf{\Phi}$ 的复数梯度 (Gradient Computation)：$\nabla_{\mathbf{\Phi}} J = \beta \mathbf{w} \mathbf{E} \mathbf{Y}^H$
>3. 执行梯度下降：$\tilde{\mathbf{\Phi}} = \mathbf{\Phi}^{(t)} - \mu \frac{\nabla_{\mathbf{\Phi}} J}{\| \nabla_{\mathbf{\Phi}} J \|_F + \epsilon}$
>4. 可行域投影 (Projection onto Feasible Set)：$\mathbf{\Phi}^{(t+1)} = \mathcal{P}_{\Omega}(\tilde{\mathbf{\Phi}})$
>投影算子 $\mathcal{P}_{\Omega}$ 的逐元素解析解为：**$\Phi^{(t+1)}_{m,n} = \frac{\tilde{\Phi}_{m,n}}{\max(1, |\tilde{\Phi}_{m,n}|)}$