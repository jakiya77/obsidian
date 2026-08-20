## TL;DR

我们要证明的其实只有这一件事：

  

$$\boxed{ \text{Path beam} \rightarrow \text{delay/phase compensation} \rightarrow \text{combine} }$$

最后从整体输入 $\mathbf y[k]$ 到输出 $z[k]$ 看，等效成了一个普通线性接收器：

  

$$z[k]=\mathbf w_{\rm Path}^H[k]\mathbf y[k].$$

然后证明：

  

$$\boxed{ \mathbf w_{\rm Path}[k] = \frac{1}{\sqrt N}\mathbf h[k] }$$

而 MRC：

  

$$\boxed{ \mathbf w_{\rm MRC}[k]\propto\mathbf h[k]. }$$

所以：

  

$$\boxed{ \mathbf w_{\rm Path}[k] \propto \mathbf w_{\rm MRC}[k]. }$$

整个证明不需要假设不同 path 正交。

  

## 问题结构树

1. 从最原始的两径信号开始
    
      
    - 1.1 每条 path 在第 $k$ 个子载波上的系数
        
          
        
    - 1.2 两条 path 如何形成总信道
        
          
        
    - 1.3 接收信号 $y[k]$
        
          
        
2. 先做 Path beam
    
      
    - 2.1 两个 matched beam 怎么定义
        
          
        
    - 2.2 两个 branch 分别得到什么
        
          
        
    - 2.3 为什么这里不要求 path 正交
        
          
        
3. 再做 delay/phase compensation 和 combining
    
      
    - 3.1 每条 branch 要补偿什么
        
          
        
    - 3.2 两个 branch 怎么加起来
        
          
        
    - 3.3 把整个操作写成矩阵形式
        
          
        
4. 找到等效接收权重 $w_{Path}$
    
      
    - 4.1 从输出式提取 $w_{Path}$
        
          
        
    - 4.2 代入 $V=A/\sqrt{N}$
        
          
        
    - 4.3 证明 $AD^H\alpha=h[k]$
        
          
        
5. 与 MRC 比较
    
      
    - 5.1 MRC 的最优权重
        
          
        
    - 5.2 为什么差 $1/\sqrt{N}$ 不影响 SNR
        
          
        
    - 5.3 最终等价结论
        
          
        
6. 最关键的物理理解
    
      
    - 6.1 为什么“branch 并未真正分干净”也不影响证明
        
          
        
    - 6.2 为什么这说明先分再合在当前系统里是绕了一圈
        
          
        

## 1. 从最原始的两径信号开始

### 1.1 每条 path 在第 $k$ 个子载波上的系数

假设有两条 desired path。

第 $l$ 条 path：

  

$$(\alpha_l,\theta_l,\tau_l).$$

定义它在第 $k$ 个子载波上的复系数：

  

$$\boxed{ d_l[k] = \alpha_l e^{-j2\pi f_k\tau_l}. }$$

这里：

  

$$\alpha_l$$

包含 path 的幅度和初始相位；

而：

  

$$e^{-j2\pi f_k\tau_l}$$

是 delay 引起的频域相位旋转。

  

### 1.2 两条 path 如何形成总信道

Path 1 的空间响应：

  

$$\mathbf a_1.$$

Path 2：

  

$$\mathbf a_2.$$

所以：

  

$$\boxed{ \mathbf h[k] = d_1[k]\mathbf a_1 + d_2[k]\mathbf a_2. }$$

把 $d_l[k]$ 展开：

  

$$\boxed{ \mathbf h[k] = \alpha_1e^{-j2\pi f_k\tau_1}\mathbf a_1 + \alpha_2e^{-j2\pi f_k\tau_2}\mathbf a_2. }$$

### 1.3 接收信号

发送的是同一个 symbol：

  

$$x[k].$$

所以：

  

$$\boxed{ \mathbf y[k] = \mathbf h[k]x[k]+\mathbf n[k]. }$$

展开就是：

  

$$\mathbf y[k] = d_1[k]\mathbf a_1x[k] + d_2[k]\mathbf a_2x[k] + \mathbf n[k].$$

到这里：

  

Plaintext

```
y[k]
│
├─ d1[k] a1 x[k]
├─ d2[k] a2 x[k]
└─ noise
```

两个 desired path 都混在 $\mathbf y[k]$ 里。

  

## 2. 先做 Path beam

### 2.1 两个 matched beam

针对 Path 1：

  

$$\boxed{ \mathbf v_1 = \frac{\mathbf a_1}{\vert{}\mathbf a_1\vert{}} = \frac{\mathbf a_1}{\sqrt N}. }$$

针对 Path 2：

  

$$\boxed{ \mathbf v_2 = \frac{\mathbf a_2}{\sqrt N}. }$$

组成：

  

$$\mathbf V = [\mathbf v_1,\mathbf v_2].$$

由于：

  

$$\mathbf A = [\mathbf a_1,\mathbf a_2],$$

所以：

  

$$\boxed{ \mathbf V = \frac{1}{\sqrt N}\mathbf A. }$$

### 2.2 Branch 1 得到什么？

做：

  

$$z_1[k] = \mathbf v_1^H\mathbf y[k].$$

代入 $\mathbf y[k]$：

  

$$\begin{aligned} z_1[k] =& d_1[k]\mathbf v_1^H\mathbf a_1x[k] \\ &+ d_2[k]\mathbf v_1^H\mathbf a_2x[k] \\ &+ \mathbf v_1^H\mathbf n[k]. \end{aligned}$$

因为：

  

$$\mathbf v_1^H\mathbf a_1 = \sqrt N,$$

所以：

  

$$\boxed{ z_1[k] = \sqrt N d_1[k]x[k] + d_2[k]\mathbf v_1^H\mathbf a_2x[k] + n_1[k]. }$$

### 2.3 Branch 2 同理

$$\boxed{ z_2[k] = d_1[k]\mathbf v_2^H\mathbf a_1x[k] + \sqrt N d_2[k]x[k] + n_2[k]. }$$

注意这里非常重要：

  

$$\boxed{ z_1\neq\text{纯 Path 1}, \qquad z_2\neq\text{纯 Path 2} }$$

一般会有 leakage。

所以这里并没有假设：

  

$$\mathbf a_1^H\mathbf a_2=0.$$

## 3. 接下来做 delay/phase compensation

这里最好不要只说“补 delay”，实际上最终是要把完整 path complex coefficient 的相位匹配掉。

  

### 3.1 定义 delay compensation matrix

定义：

  

$$\boxed{ \mathbf D[k] = \begin{bmatrix} e^{+j2\pi f_k\tau_1}&0 \\ 0&e^{+j2\pi f_k\tau_2} \end{bmatrix}. }$$

因为原来的 delay phase 是：

  

$$e^{-j2\pi f_k\tau_l}.$$

所以这里使用相反方向的 phase。

  

### 3.2 Branch 写成一个向量

定义：

  

$$\mathbf z_b[k] = \begin{bmatrix} z_1[k] \\ z_2[k] \end{bmatrix}.$$

由于：

  

$$z_1=\mathbf v_1^H\mathbf y, \qquad z_2=\mathbf v_2^H\mathbf y,$$

所以：

  

$$\boxed{ \mathbf z_b[k] = \mathbf V^H\mathbf y[k]. }$$

### 3.3 对两个 branch 做 delay compensation

$$\boxed{ \widetilde{\mathbf z}_b[k] = \mathbf D[k]\mathbf V^H\mathbf y[k]. }$$

现在再考虑 $\alpha_l$ 自身的复相位。

定义：

  

$$\boldsymbol\alpha = \begin{bmatrix} \alpha_1 \\ \alpha_2 \end{bmatrix}.$$

最终输出：

  

$$\boxed{ z[k] = \boldsymbol\alpha^H \mathbf D[k] \mathbf V^H \mathbf y[k]. }$$

## 4. 现在找它的等效 $\mathbf w_{\rm Path}$

这是整个证明最关键的一步。

  

### 4.1 普通线性接收器长什么样？

所有线性 combiner 都可以写成：

  

$$\boxed{ z[k] = \mathbf w^H[k]\mathbf y[k]. }$$

而我们现在得到：

  

$$z[k] = \boldsymbol\alpha^H \mathbf D[k] \mathbf V^H \mathbf y[k].$$

把前面的东西看成一个整体：

  

$$\underbrace{ \boldsymbol\alpha^H \mathbf D[k] \mathbf V^H }_{\mathbf w_{\rm Path}^H[k]}.$$

因此：

  

$$\boxed{ \mathbf w_{\rm Path}^H[k] = \boldsymbol\alpha^H \mathbf D[k] \mathbf V^H. }$$

两边取 Hermitian：

  

$$\boxed{ \mathbf w_{\rm Path}[k] = \mathbf V \mathbf D^H[k] \boldsymbol\alpha. }$$

这就是你代码：

  

Matlab

```
w_eq = V * Dk' * alpha;
```

的来源。

  

## 5. 现在开始证明它其实就是 $\mathbf h[k]$

### 5.1 先代入 matched beam

已经知道：

  

$$\mathbf V = \frac1{\sqrt N}\mathbf A.$$

所以：

  

$$\mathbf w_{\rm Path}[k] = \frac1{\sqrt N} \mathbf A \mathbf D^H[k] \boldsymbol\alpha.$$

### 5.2 再看 $\mathbf D^H[k]$

因为：

  

$$\mathbf D[k] = \begin{bmatrix} e^{+j2\pi f_k\tau_1}&0 \\ 0&e^{+j2\pi f_k\tau_2} \end{bmatrix},$$

所以：

  

$$\boxed{ \mathbf D^H[k] = \begin{bmatrix} e^{-j2\pi f_k\tau_1}&0 \\ 0&e^{-j2\pi f_k\tau_2} \end{bmatrix}. }$$

于是：

  

$$\mathbf D^H[k]\boldsymbol\alpha = \begin{bmatrix} \alpha_1e^{-j2\pi f_k\tau_1} \\ \alpha_2e^{-j2\pi f_k\tau_2} \end{bmatrix}.$$

注意！

这个东西其实就是：

  

$$\boxed{ \begin{bmatrix} d_1[k] \\ d_2[k] \end{bmatrix}. }$$

### 5.3 再左乘 $\mathbf A$

$$\mathbf A = [\mathbf a_1,\mathbf a_2].$$

所以：

  

$$\begin{aligned} \mathbf A \mathbf D^H[k]\boldsymbol\alpha &= [\mathbf a_1,\mathbf a_2] \begin{bmatrix} d_1[k] \\ d_2[k] \end{bmatrix} \\ &= d_1[k]\mathbf a_1 + d_2[k]\mathbf a_2. \end{aligned}$$

但是一开始我们就定义：

  

$$\boxed{ \mathbf h[k] = d_1[k]\mathbf a_1+ d_2[k]\mathbf a_2. }$$

所以：

  

$$\boxed{ \mathbf A \mathbf D^H[k] \boldsymbol\alpha = \mathbf h[k]. }$$

最终：

  

$$\boxed{ \mathbf w_{\rm Path}[k] = \frac1{\sqrt N}\mathbf h[k]. }$$

## 6. 再和 MRC 比较

### 6.1 MRC 的结果

白噪声条件下：

  

$$\boxed{ \mathbf w_{\rm MRC}[k] \propto \mathbf h[k]. }$$

例如直接取：

  

$$\mathbf w_{\rm MRC}[k] = \mathbf h[k].$$

### 6.2 Path receiver

刚才证明：

  

$$\mathbf w_{\rm Path}[k] = \frac1{\sqrt N}\mathbf h[k].$$

所以：

  

$$\boxed{ \mathbf w_{\rm Path}[k] = \frac1{\sqrt N} \mathbf w_{\rm MRC}[k]. }$$

因此：

  

$$\boxed{ \mathbf w_{\rm Path}[k] \propto \mathbf w_{\rm MRC}[k]. }$$

### 6.3 为什么 $1/\sqrt N$ 无所谓？

输出 SNR：

  

$$\gamma = \frac{ P_s\vert{}\mathbf w^H\mathbf h\vert{}^2 }{ \sigma^2\vert{}\mathbf w\vert{}^2 }.$$

如果：

  

$$\widetilde{\mathbf w}=c\mathbf w,$$

那么：

  

$$\gamma = \frac{ P_s\vert{}c\vert{}^2\vert{}\mathbf w^H\mathbf h\vert{}^2 }{ \sigma^2\vert{}c\vert{}^2\vert{}\mathbf w\vert{}^2 }.$$

$\vert{}c\vert{}^2$ 抵消。

所以：

  

$$\boxed{ \mathbf h \quad\text{和}\quad \frac{\mathbf h}{\sqrt N} }$$

输出 SNR 完全相同。

  

## 7. 最值得注意：这个证明根本没用到“path 正交”

你前面已经发现了：

  

Branch 1 其实还包含 Path 2，Branch 2 也可能包含 Path 1。

完全正确。

但看刚才整个证明：

  

$$\mathbf w_{\rm Path} = \frac1{\sqrt N} \mathbf A\mathbf D^H\boldsymbol\alpha = \frac1{\sqrt N}\mathbf h.$$

我们从来没有写过：

  

$$\mathbf a_1^H\mathbf a_2=0.$$

也没有要求：

  

$$\mathbf A^H\mathbf A=N\mathbf I.$$

所以：

  

$$\boxed{ \text{即使两个 matched branch 互相串扰，整体 Path receiver 仍然可以恰好等效于 MRC。} }$$

这个地方其实很有意思。

  

## 8. 为什么这会让“先分 branch 再合并”显得多余？

现在你就可以从数学上看到：

原来的信道本来已经是：

  

$$\boxed{ \mathbf h[k] = \mathbf A\mathbf D^H[k]\boldsymbol\alpha. }$$

MRC 直接说：

  

$$\boxed{ \mathbf w=\mathbf h. }$$

而 Path 方法做的是：

  

Plaintext

```
A
│
├─ 每列归一化 → V
│
↓
形成多个 matched branches
│
↓
乘 D 做 delay phase compensation
│
↓
乘 α 做 path weighting
│
↓
重新合成
│
↓
w_Path
```

结果：

  

$$\boxed{ \mathbf w_{\rm Path} = \frac1{\sqrt N} \mathbf A\mathbf D^H\boldsymbol\alpha = \frac1{\sqrt N}\mathbf h. }$$

所以它实际上就是：

  

已经有：

$h = A D^H \alpha$

  

MRC：

直接用 $h$

  

Path 方法：

把 $h$ 拆成 $A$、$D$、$\alpha$

↓

处理一圈

↓

再重新乘成 $A D^H \alpha$

↓

又得到 $h$

  

这就是为什么我们现在越来越明确：

  

$$\boxed{ \text{在当前 perfect-CSI full-digital OFDM 场景中，为了最终接收而先“分 desired paths”并不是必要步骤。} }$$

如果以后我们要证明 path-wise processing 有研究价值，就必须引入一个让“不能直接用完整 $\mathbf h[k]$”或“不能自由实现 $\mathbf w[k]$”成立的条件。否则数学上很容易再次退化回 MRC。