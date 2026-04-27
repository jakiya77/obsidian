R_jj 是协方差矩阵，所以是 Hermite（对称/共轭对称）矩阵，可以特征分解：
    
$$
    R_{jj}=UΛU
$$
    
    其中：
    
    - UU：单位正交矩阵，列是特征向量 u1,…,uN
        
    - Λ=diag(λ1,…,λN)Λ=diag(λ1​,…,λN​)：对角阵，元素是特征值
令
    
$$
    z=U^Hw
$$
    
    因为 U 是酉矩阵（正交的复数版），有
    
$$
    w=Uz,w^Hw=z^Hz
$$ Rayleigh 熵：
    
$$
\frac{w^HR_{jj}w}{w^Hw} = \frac{z^HΛz}{z^Hz}=\frac{\sum_{i=1}^{N}\lambda_i|z_i|^2}{\sum_{i=1}^{N}|z_i|^2}​
$$
观察最后这个式子能发现：  
    分子是 “特征值的加权平均”，权重是 ∣zi∣^2，分母是权重之和。  
    所以这个商一定在所有特征值的最小值和最大值之间：
$$
    λmin⁡≤\frac{\sum_{i=1}^{N}\lambda_i|z_i|^2}{\sum_{i=1}^{N}|z_i|^2}​≤λmax​
$$
    
[[LMS算法的步长]]