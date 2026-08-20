
$$\boxed{\text{真正需要多个波束} \neq \text{存在多个 path}}$$

而更准确地说是：

$$\boxed{\text{需要先把不同成分分离出来，并让它们分别经过不同的处理}}$$
$$\boxed{\mathbf w_1,\mathbf w_2,\ldots \text{需要分别设计}}$$

如果最后所有东西都只是同一个 desired symbol，而且可以直接设计一个任意的 full-digital $\mathbf w[k]$，那么很多所谓“多个 beam”最后都能压缩成一个等效：

$$\mathbf w_{\rm eq}[k].$$

# 目录 / 问题结构树

- **1. 什么时候一个 $w$ 就够了？**
    
      
    - 1.1 多径属于同一个 desired signal
        
          
        
    - 1.2 full-digital $w$ 可以自由设计
        
          
        
- **2. 什么时候真正需要多个 $w$？**
    
      
    - 2.1 不同 path 后面要做不同 delay 补偿
        
          
        
    - 2.2 不同 user / stream 必须分别检测
        
          
        
    - 2.3 desired / interference 要分别提取
        
          
        
    - 2.4 硬件限制导致一个 $w$ 无法完成所有事情
        
          
        
- **3. 对我们 MA + wideband 项目意味着什么？**
    
      
    - 3.1 不能把“有多个 path”直接等同于“需要 multibeam”
        
          
        
    - 3.2 真正有意义的是 path-resolved processing
        
          
        
    - 3.3 MA 的作用是提高这些 path 的可分离性
        
          
        

## 1. 什么情况下一个 $w$ 就够了？

### 1.1 如果所有 path 都是同一个信号

比如：

$$\mathbf y[k] = \left( \mathbf h_1[k] + \mathbf h_2[k] \right)x[k] + \mathbf n[k].$$

Path 1 和 Path 2 虽然：


$$\theta_1\neq\theta_2, \qquad \tau_1\neq\tau_2,$$

但它们携带的都是：

$$x[k].$$

于是整个 channel 本来就是：

$$\mathbf h[k] = \mathbf h_1[k]+\mathbf h_2[k].$$

那么直接：

$$z[k] = \mathbf w^H[k]\mathbf y[k]$$

就行。单用户白噪声情况下：

$$\boxed{\mathbf w[k]\propto\mathbf h[k]}$$

就是 MRC。这里根本不要求你显式地设计两个波束。

### 1.2 即使你人为设计两个波束，也可能是假 multibeam

假设你写：

$$z_1[k]=\mathbf w_1^H[k]\mathbf y[k],$$

$$z_2[k]=\mathbf w_2^H[k]\mathbf y[k].$$

最后：
$$z[k] = c_1[k]z_1[k]+c_2[k]z_2[k].$$
代进去：

$$z[k] = c_1[k]\mathbf w_1^H[k]\mathbf y[k] + c_2[k]\mathbf w_2^H[k]\mathbf y[k].$$

整理：
$$z[k] = \left( c_1[k]\mathbf w_1[k] + c_2[k]\mathbf w_2[k] \right)^H \mathbf y[k].$$

定义：
$$\boxed{\mathbf w_{\rm eq}[k] = c_1[k]\mathbf w_1[k] + c_2[k]\mathbf w_2[k]}$$

那么：
$$\boxed{z[k]=\mathbf w_{\rm eq}^H[k]\mathbf y[k]}$$

所以你看：

- **表面上**：两个 beam。
    
- **实际上**：
    
- **数学上**：还是一个 beamforming vector。
    
这就是我们刚才那个 MATLAB sanity experiment 想验证的事情。

## 2. 那什么时候才真正需要多个 $w$？

这里就是你现在最应该记住的判断标准：

$$\boxed{\text{多个输出分支在合并以前，必须分别做不同事情}}$$

这时候 $\mathbf w_1,\mathbf w_2$ 才有真正存在的意义。

### 2.1 不同 path 后面需要不同的 delay processing

例如：



$$\text{Path 1}:(\theta_1,\tau_1)$$

$$\text{Path 2}:(\theta_2,\tau_2).$$

先做：

  

$$z_1(t)=\mathbf w_1^H\mathbf y(t)$$

$$z_2(t)=\mathbf w_2^H\mathbf y(t).$$

然后：

  

- $z_1(t) \rightarrow$ delay compensation $\tau_1$
    
      
    
- $z_2(t) \rightarrow$ delay compensation $\tau_2$
    
      
    

再合：

  

$$z(t) = \tilde z_1(t)+\tilde z_2(t).$$

这个结构就是：

  

Plaintext

```
                y(t)
                 │
        ┌────────┴────────┐
        ↓                 ↓
       w1                w2
        ↓                 ↓
   path-1 branch     path-2 branch
        ↓                 ↓
     τ1补偿             τ2补偿
        │                 │
        └────────┬────────┘
                 ↓
              combine
```

这时候分支是有物理意义的。

但注意一个关键细节：如果你已经在 OFDM FFT 后、并且允许每个子载波独立设计 $\mathbf w[k]$，那么上述 delay compensation 很可能又可以吸收到 $\mathbf w[k]$ 里面。所以不能仅凭“不同 $\tau_l$”就说 multibeam 一定比单 $\mathbf w[k]$ 强。

  

## 3. 更典型的真正多个 $w$其实是多 stream

这个最好理解。比如两个用户：

$$\mathbf y = \mathbf h_1x_1+\mathbf h_2x_2+\mathbf n.$$

你希望同时恢复 $x_1$ 和 $x_2$。那么必须：

$$z_1=\mathbf w_1^H\mathbf y$$

$$z_2=\mathbf w_2^H\mathbf y.$$

因为：

  

$$z_1\rightarrow\hat x_1,$$

$$z_2\rightarrow\hat x_2.$$

这里你显然不能写成一个 $\mathbf w_{\rm eq}$ 然后说完事了，因为你要的是两个独立输出。所以：

$$\boxed{\text{多个独立数据流} \Rightarrow \text{多个真正的 receive beams}}$$

这才是最标准的 multibeam。


## 4. 回到我们这个项目，真正值得研究的是哪一种？

这里我觉得你现在可以把之前有点混在一起的几个概念彻底分开。


Plaintext

```
                multipath
                   │
          ┌────────┴────────┐
          ↓                 ↓
      “能不能合”          “要不要分”
          │                 │
          ↓                 ↓
 所有path同一个x       path需要独立处理
          │                 │
          ↓                 ↓
 一个 w[k]可能够       多个 w_l 有意义
```

我们的目标如果只是：


> “我有两条 multipath，所以给每条 path 一个 beam。”
> 

这个论据不够。应该是：


> 我要显式地获得不同的 path branch，因为之后我要对这些 branch 进行不同的 delay / weighting / interference / robustness processing。
> 

那么：

  

$$\boxed{\mathbf w_1,\ldots,\mathbf w_L}$$

才真正有研究意义。

  

## 5. MA 在这里的价值也一下清楚了

MA 不应该简单解释成：

> 移动天线，然后增大 channel gain。
>
对于我们的这个方向，更漂亮的解释其实是：

$$\boxed{\text{MA 改变不同 path 的空间 signature}}$$

即 $\mathbf a_{\mathbf P}(\theta_1), \mathbf a_{\mathbf P}(\theta_2)$。通过改变 $\mathbf P$，让 $\mathbf a_{\mathbf P}(\theta_1)$ 和 $\mathbf a_{\mathbf P}(\theta_2)$ 更加可分。

例如希望：

$$\boxed{\frac{\vert{}\mathbf a_1^H\mathbf a_2\vert{}}{\vert{}\mathbf a_1\vert{}\vert{}\mathbf a_2\vert{}} \downarrow}$$

于是：

- Path 1 ──→ $w_1$ 更容易提取
    
      

- Path 2 ──→ $w_2$ 更容易提取
    
      
    

这就形成了一个很漂亮的链条：

  

$$\boxed{\mathbf P \rightarrow \text{path separability} \rightarrow {\mathbf w_l} \rightarrow \text{delay alignment} \rightarrow \text{coherent combining}}$$

这比 $\mathbf P\rightarrow\text{channel gain}$ 要更贴合我们正在想的宽带 multipath 课题。

  

## 6. 但是还要加一个非常重要的条件

假如：

  

$$\theta_1=\theta_2$$

$$\tau_1\neq\tau_2.$$

那么：

  

$$\mathbf a(\theta_1) = \mathbf a(\theta_2).$$

你想设计 $\mathbf w_1$ 只提取 Path 1，再设计 $\mathbf w_2$ 只提取 Path 2，基本做不到。因为空间上它们就是同一个方向。

  

所以：

  

Plaintext

```
θ1 ≠ θ2
  ↓
空间可以分
  ↓
多个 spatial w_l 有意义
```

而：

  

Plaintext

```
θ1 ≈ θ2
τ1 ≠ τ2
  ↓
空间分不开
  ↓
靠 delay/tap/frequency-domain processing
```

这也是我们之前讨论“空域不能解决所有 multipath”的真正含义。

  

## 7. 所以你刚才那句话，我帮你改成一个更严格的版本

你现在说的是：

  

> 需要使用多个波束的情况，是那种需要分离设计 $w$ 的情况。
> 
>   

我建议你以后脑子里记成：

  

$$\boxed{\text{需要多个波束的关键，不是存在多个传播路径，而是接收机需要形成多个具有独立功能的空间输出分支。}}$$

所谓“独立功能”例如：

  

$$\begin{cases} \mathbf w_1:\text{提取 Path 1}\\ \mathbf w_2:\text{提取 Path 2} \end{cases}$$

然后：

  

$$\begin{cases} \text{Branch 1}\rightarrow\tau_1\text{ compensation}\\ \text{Branch 2}\rightarrow\tau_2\text{ compensation} \end{cases}$$

这才是真正意义上的：

  

$$\boxed{\text{path-wise multibeam receiver}}$$

## 8. 再给你一个以后特别好用的判断题

以后看到任何论文说：

  

> “We design multiple beams for different paths.”
> 
>   

你马上问它一句：

  

$$\boxed{\text{这些 beams 最后能不能等效成一个 }\mathbf w?}$$

**如果能：**

  

Plaintext

```
w1 ─┐
w2 ─┼→ linear combine
w3 ─┘
      ↓
   一个 w_eq
```

那 multibeam 很可能只是实现形式/参数化方式。

  

**如果不能简单压缩：**

  

Plaintext

```
w1 → delay τ1 → detector 1
w2 → delay τ2 → detector 2
w3 → interference cancellation
                    ↓
                 再组合
```

那这些 beam 才真正承担了：

  

$$\boxed{\text{不同功能}}$$

所以这个判断标准其实非常强：

  

$$\boxed{\textbf{“有没有必要保留多个独立输出？”}}$$

这比问“有没有多个 path”准确得多。

  

而对我们现在的 MA + 宽带项目来说，下一步真正值得研究的就变成了：**能不能利用 MA 主动把本来混在一起的 path 变得更可分，从而让多个独立的 $\mathbf w_l$ 分支真的有价值。** 这就比单纯说“MA + multibeam”清晰很多。