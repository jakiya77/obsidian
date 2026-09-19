
# 隐蔽通信与 Willie 检测模型分析

**标签：** #隐蔽通信 #物理层安全 #DEP检测错误概率 #信号处理

  

> [!note] 核心定义
> 
> Willie 的任务不是解码信号，而是判断 Alice **有没有发送信号**。
> 
> 评价这一判断能力的核心指标是：
> 
>   
> 
> $$\boxed{\mathrm{DEP}=\text{Detection Error Probability}}$$
> 
> （中文：检测错误概率）
> 
>   

## 一、 Willie 的检测假设与错误类型

Willie 在检测过程中面临两个假设（Hypothesis）：

  

|**假设**|**实际情况**|
|---|---|
|**$\mathcal{H}_0$**|Alice **没有**发送隐蔽信号|
|**$\mathcal{H}_1$**|Alice **正在**发送隐蔽信号|

Willie 观察接收到的信号并做出判断，这期间可能会犯**两种错误**：

  

1. **虚警 (False Alarm)**
    
      
    - **情境**：Alice 实际没发送 ($\mathcal{H}_0$)，Willie 却判断 Alice 在发送。
        
          
        
    - **概率表示**：$P_{\mathrm{FA}}$
        
          
        
2. **漏检 (Missed Detection)**
    
      
    - **情境**：Alice 实际正在发送 ($\mathcal{H}_1$)，Willie 却判断 Alice 没有发送。
        
          
        
    - **概率表示**：$P_{\mathrm{MD}}$
        
          
        

### 1.1 DEP ($\xi$) 的数学定义

在我们当前代码所采用的常见隐蔽通信定义下，检测错误概率 $\xi$ 被定义为这两种条件错误概率之和：

  

$$\boxed{\xi=P_{\mathrm{FA}}+P_{\mathrm{MD}}}$$

> [!warning] 注意
> 
> $\xi$ 是**两种条件错误概率之和**，并不是根据先验概率加权的平均错误率。
> 
>   

### 1.2 物理意义剖析

- **极端情况 A（完全无法区分）：**
    
    如果 Willie 总是判断 Alice 没有发信号，那么 $P_{\mathrm{FA}}=0$ 且 $P_{\mathrm{MD}}=1$，此时：
    
      
    
    $$\xi=1$$
    
    这表示 Willie 完全没有有效地区分两种通信状态。
    
      
    
- **极端情况 B（完美检测）：**
    
    如果 Willie 能够准确判断 Alice 是否在通信，那么 $P_{\mathrm{FA}}\approx0$ 且 $P_{\mathrm{MD}}\approx0$，此时：
    
      
    
    $$\xi\approx0$$
    

**结论：** 对于 Willie 的最优检测器而言，**最小 DEP 越接近 1，Alice 的通信就越难被检测出来（隐蔽性越好）**。

  

## 二、 KL 散度与 DEP 下界 (MATLAB 仿真释疑)

在 MATLAB 仿真输出中，常看到如下结果：

  

Plaintext

```
D = 0.020000
DEP lower bound = 0.9000
```

### 2.1 理论推导 (Pinsker 不等式)

这里的 $D$ 指代两种接收信号统计分布（$\mathcal{H}_0$ 与 $\mathcal{H}_1$ 状态下）之间的 **KL 散度 (Kullback-Leibler Divergence)**，用于衡量它们在统计意义上的差异。

  

代码使用 **Pinsker 不等式**建立 KL 散度与最优 DEP 下界 $\xi^\star$ 之间的关系：

  

$$\boxed{\xi^\star \geq 1-\sqrt{\frac{D}{2}}}$$

### 2.2 数据代入计算

将代码中的散度值代入公式：

  

- $D=0.02$
    
      
    
    $$\xi^\star \geq 1-\sqrt{\frac{0.02}{2}} = 1-\sqrt{0.01} = 0.9$$
    

> [!info] 为什么四种方案输出的下界都是 0.9？
> 
> 这个 $0.9$ 只说明 Willie 的最优 DEP **至少**为 $0.9$，并不是说实际 DEP 恰好等于 $0.9$（实际最优 DEP 可能是 0.93 或 0.97，仍然满足此下界）。
> 
>   
> 
> 四种方案虽然通信速率不同，但 MATLAB 都输出了接近 $0.9$ 的 DEP lower bound，是因为**它们基本都把同一个 KL 隐蔽约束（$D \leq 0.02$）用满了**。
> 
>   

## 三、 全局视角：TX-MA 的物理链路机制

将上述理论串联起来，可以通过以下物理过程链条来理解这段 MATLAB 仿真的本质：

  

> [!abstract] 物理过程因果链
> 
>   
> 
> 1. 📡 **Alice 发射**同一个信号。
>     
>       
>     
> 2. 〰️ **多径传播**：信号沿不同传播路径到达 Willie。
>     
>       
>     
> 3. ⚡ **电磁叠加**：电磁场在 Willie 所在位置发生叠加。
>     
>       
>     
> 4. 📍 **MA 位置决定相位**：移动天线 (MA) 的位置决定了各路径的相对相位。
>     
>       
>     
> 5. 📉 **信道改变**：Willie 处的合成信道 $h_W(p)$ 发生改变。
>     
>       
>     
> 6. 📊 **统计分布改变**：Willie 接收信号的统计分布（KL 散度 $D$）随之改变。
>     
>       
>     
> 7. 👁️ **区分能力改变**：Willie 区分 $\mathcal{H}_0/\mathcal{H}_1$ 的能力受到影响。
>     
>       
>     
> 8. 🎯 **最终结果**：DEP ($\xi$) 改变。
>     
>       
>     

### 总结

我们引入 **TX-MA (Transmit Moving Antenna)** 的真正目的，并不是让 Alice 的信号“发射得不相干”；而是**利用发射位置的微小改变，重塑不同接收点（Bob 和 Willie）的空间信道特性**。

这样就能在确保 Bob 能够可靠接收（高通信速率）的同时，严格限制 Willie 处接收信号的统计差异（利用 KL 散度约束），从而让 Willie 的 DEP 逼近于 1，达成高隐蔽性。