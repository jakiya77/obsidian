

## TL;DR

先记住最核心的关系：

$$  
\boxed{  
\text{SINR} \rightarrow \text{描述接收质量}  
}  
$$
$$  
\boxed{  
\text{SE} \rightarrow \text{描述单位带宽能可靠传多少信息}  
}  
$$

二者在理想高斯信道模型下通过 Shannon 公式联系：
# $$  
\boxed{  
\mathrm{SE} = 

\log_2(1+\mathrm{SINR})  
\quad  
[\mathrm{bit/s/Hz}]  
}  
$$

对于我们现在的研究：

```text
研究 MA / 多波束 / taps 为什么有效
        ↓
优先看 output SNR / SINR

想说明最终通信性能提高多少
        ↓
再看 Spectral Efficiency
```



---

## 目录 / 问题结构树

```text
1. SINR 与 SE 分别是什么？
   ├─ 1.1 SNR / SINR 的物理意义
   └─ 1.2 SE 的物理意义

2. SINR 是怎么推导出来的？
   ├─ 2.1 接收信号模型
   ├─ 2.2 波束形成后的信号
   └─ 2.3 输出 SINR

3. SE 是怎么从 SINR 得到的？
   ├─ 3.1 Shannon 容量
   ├─ 3.2 除以带宽
   └─ 3.3 得到 bit/s/Hz

4. OFDM 宽带系统怎么计算 SE？
   ├─ 4.1 每个子载波一个 SINR
   ├─ 4.2 每个子载波一个 SE
   └─ 4.3 对所有子载波求平均

5. SINR 和 SE 到底什么时候用？
   ├─ 5.1 机制研究 → SINR
   ├─ 5.2 系统通信性能 → SE
   └─ 5.3 论文实验 → 两者配合

6. 常见误区
   ├─ 6.1 平均 SINR ≠ 平均 SE
   ├─ 6.2 SE 不是“接收端速率”
   └─ 6.3 log2(1+SINR) 不是所有实际系统的真实吞吐量
```

---

# 1. SINR 与 SE 分别是什么？

## 1.1 SNR / SINR：接收质量

先区分两个概念。

### 1.1.1 只有噪声

如果系统中：

$$  
\text{desired signal}+\text{noise}  
$$

那么用：

# $$  
\boxed{  
\mathrm{SNR} = 

\frac{\text{Signal Power}}  
{\text{Noise Power}}  
}  
$$

---

### 1.1.2 同时存在干扰

如果系统中：

$$  
\text{desired signal}  
+  
\text{interference}  
+  
\text{noise},  
$$

那么：

# $$  
\boxed{  
\mathrm{SINR}=

\frac{\text{Signal Power}}  
{\text{Interference Power}+\text{Noise Power}}  
}  
$$

所以它本质上回答：

> **我最终接收到的期望信号，相对于那些“不想要的东西”，到底有多强？**

---

## 1.2 SE：单位频谱资源能够承载多少信息

Spectral Efficiency：

$$  
\boxed{\mathrm{SE}=\text{Spectral Efficiency}}  
$$

单位：

$$  
\boxed{\mathrm{bit/s/Hz}}  
$$

物理意义是：

> **每占用 1 Hz 带宽，每秒理论上可以可靠传输多少 bit。**

例如：

$$  
\mathrm{SE}=4\ \mathrm{bit/s/Hz}.  
$$

表示理想情况下：

$$  
1,\mathrm{Hz}  
$$

频谱资源每秒能传：

$$  
4,\mathrm{bit}.  
$$

如果带宽：

$$  
B=100,\mathrm{MHz},  
$$

粗略理想容量：

$$  
C=B\times \mathrm{SE}  
$$

所以：

# $$  
C

# 100\times10^6\times4

400\ \mathrm{Mbit/s}.  
$$

---

# 2. SINR 是怎么推导出来的？

## 2.1 从最基本的接收模型开始

假设接收阵列收到：

# $$  
\mathbf y

\mathbf h x  
+  
\mathbf g s  
+  
\mathbf n.  
$$

其中：

$$  
\mathbf h  
$$

是 desired channel，

$$  
x  
$$

是期望信号，

$$  
\mathbf g  
$$

是干扰信道，

$$  
s  
$$

是干扰信号，

$$  
\mathbf n  
$$

是噪声。

---

## 2.2 接收端用波束 $\mathbf w$

接收输出：

# $$  
z

\mathbf w^H\mathbf y.  
$$

代进去：

# $$  
z

\mathbf w^H\mathbf h x  
+  
\mathbf w^H\mathbf g s  
+  
\mathbf w^H\mathbf n.  
$$

于是：

```text
输出 z
│
├─ desired
│    wᴴ h x
│
├─ interference
│    wᴴ g s
│
└─ noise
     wᴴ n
```

---

## 2.3 分别计算功率

如果：

$$  
\mathbb E|x|^2=P_s,  
$$

那么 desired power：

# $$  
P_{\rm desired}

P_s|\mathbf w^H\mathbf h|^2.  
$$

如果干扰：

$$  
\mathbb E|s|^2=P_j,  
$$

那么：

# $$  
P_{\rm interference}

P_j|\mathbf w^H\mathbf g|^2.  
$$

白噪声：

$$  
\mathbf n  
\sim  
\mathcal{CN}  
(\mathbf 0,\sigma^2\mathbf I),  
$$

经过波束形成以后：

# $$  
\mathbb E |\mathbf w^H\mathbf n|^2

\sigma^2|\mathbf w|^2.  
$$

于是最终：

# $$  
\boxed{  
\mathrm{SINR}

\frac{  
P_s|\mathbf w^H\mathbf h|^2  
}{  
P_j|\mathbf w^H\mathbf g|^2  
+  
\sigma^2|\mathbf w|^2  
}  
}  
$$

这就是我们以前 TVT 那种 output SINR 的来源。

---

# 3. 如果没有 interference 呢？

那就是：

$$  
\mathbf y=\mathbf h x+\mathbf n.  
$$

于是：

# $$  
\boxed{  
\mathrm{SNR}

\frac{  
P_s|\mathbf w^H\mathbf h|^2  
}{  
\sigma^2|\mathbf w|^2  
}  
}  
$$

所以我们当前的：

```matlab
snr_single
snr_noTap
snr_tap
```

严格来说都是：

$$  
\boxed{\text{post-combining SNR}}  
$$

因为现在没有 jammer / interference。

---

# 4. SE 是怎么从 SINR 推导出来的？

## 4.1 从 Shannon capacity 开始

经典 AWGN 信道容量：

# $$  
\boxed{  
C

B\log_2  
\left(  
1+\mathrm{SNR}  
\right)  
}  
$$

其中：

$$  
C  
$$

单位是：

$$  
\mathrm{bit/s}.  
$$

---

## 4.2 为什么除以带宽？

我们希望衡量：

> 我每用 1 Hz，到底能传多少 bit/s？

于是：

# $$  
\frac{C}{B}

\log_2  
(1+\mathrm{SNR}).  
$$

因此：

# $$  
\boxed{  
\mathrm{SE}

\log_2  
(1+\mathrm{SNR})  
}  
$$

单位：

$$  
\boxed{\mathrm{bit/s/Hz}}  
$$

---

## 4.3 有 interference 时

把 SNR 换成有效 SINR：

# $$  
\boxed{  
\mathrm{SE}

\log_2  
(1+\mathrm{SINR})  
}  
$$

本质上是：

> 把 interference 当作额外噪声处理。

---

# 5. 一个具体数字例子

假设：

$$  
\mathrm{SINR}=10\ \mathrm{dB}.  
$$

先转成线性值：

# $$  
\gamma

# 10^{10/10}

$$

于是：

# $$  
\mathrm{SE}

# \log_2(1+10)

\log_2(11)  
\approx3.46.  
$$

因此：

$$  
\boxed{  
10\ \mathrm{dB}  
\Longrightarrow  
3.46\ \mathrm{bit/s/Hz}  
}  
$$

注意计算时必须用：

$$  
\boxed{\text{线性 SINR}}  
$$

不能直接：

$$  
\log_2(1+10\mathrm{dB}).  
$$

---

# 6. 为什么 SE 与 SINR 是对数关系？

这是很重要的直觉。

假设：

$$  
\mathrm{SINR}=0\ \mathrm{dB}  
$$

即线性：

$$  
1.  
$$

那么：

$$  
R=\log_2(2)=1.  
$$

---

如果：

$$  
\mathrm{SINR}=10\ \mathrm{dB}  
$$

即：

$$  
10,  
$$

那么：

$$  
R\approx3.46.  
$$

---

如果：

$$  
\mathrm{SINR}=20\ \mathrm{dB}  
$$

即：

$$  
100,  
$$

那么：

# $$  
R

\log_2(101)  
\approx6.66.  
$$

所以：

|SINR|SE|
|---|---|
|0 dB|1 bit/s/Hz|
|10 dB|3.46 bit/s/Hz|
|20 dB|6.66 bit/s/Hz|
|30 dB|9.97 bit/s/Hz|

你会发现：

$$  
\boxed{  
\text{SINR 是指数尺度变化，SE 是对数增长}  
}  
$$

所以不断堆 SINR，会出现：

$$  
\boxed{\text{边际通信收益递减}}  
$$

---

# 7. 为什么研究机制时 SINR 更合适？

这是跟你现在项目最相关的。

假设 MA 使：

$$  
5\ \mathrm{dB}  
\rightarrow  
10\ \mathrm{dB}.  
$$

我们可以直接说：

$$  
\boxed{  
\text{MA 提升了约 }5\text{ dB output SINR}  
}  
$$

这非常适合解释：

```text
MA位置改变
    ↓
desired gain ↑
或者 interference leakage ↓
或者 noise enhancement ↓
    ↓
output SINR ↑
```

所以 SINR 跟物理机制之间的距离很短。

---

# 8. SE 为什么更适合最终通信性能？

因为通信最终关心的是：

$$  
\boxed{\text{传多少信息}}  
$$

而不是：

$$  
\boxed{\text{SINR 数字本身有多大}}  
$$

例如方案 A：

$$  
\mathrm{SINR}=10,\mathrm{dB}  
$$

方案 B：

$$  
\mathrm{SINR}=15,\mathrm{dB}.  
$$

看 SINR：

$$  
+5,\mathrm{dB}.  
$$

但最终更有通信意义的是比较：

$$  
\log_2(1+10)  
$$

和：

$$  
\log_2(1+31.62).  
$$

也就是：

$$  
3.46  
$$

vs.

$$  
5.03\ \mathrm{bit/s/Hz}.  
$$

于是可以说：

$$  
\boxed{  
\text{通信效率增加约 }  
1.57\ \mathrm{bit/s/Hz}  
}  
$$

---

# 9. OFDM 情况为什么 SE 尤其重要？

这是你现在宽带代码的核心。

## 9.1 每个子载波信道不同

OFDM 中：

$$  
\mathbf h[1],  
\mathbf h[2],  
\dots,  
\mathbf h[K]  
$$

通常不同。

因此每个子载波都有：

# $$  
\gamma_k

\mathrm{SINR}[k].  
$$

---

## 9.2 每个子载波对应自己的 SE

第 $k$ 个子载波：

# $$  
\boxed{  
R_k

\log_2(1+\gamma_k)  
}  
$$

因此：

```text
subcarrier 1 → SINR1 → R1
subcarrier 2 → SINR2 → R2
subcarrier 3 → SINR3 → R3
...
subcarrier K → SINRK → RK
```

---

## 9.3 整个 OFDM 的平均 spectral efficiency

如果每个子载波等带宽、等时间资源，那么：

# $$  
\boxed{  
R_{\rm avg}

\frac1K  
\sum_{k=1}^{K}  
\log_2(1+\gamma_k)  
}  
$$

这就是我们 MATLAB：

```matlab
R_MB_tap = mean(log2(1 + snr_tap));
```

的来源。

---

# 10. 为什么不能先平均 SINR 再算 SE？

这是一个非常常见的错误。

错误写法：

$$  
\log_2  
\left(  
1+  
\frac1K  
\sum_k\gamma_k  
\right).  
$$

正确写法：

$$  
\boxed{  
\frac1K  
\sum_k  
\log_2(1+\gamma_k)  
}  
$$

一般：

$$  
\boxed{  
\frac1K  
\sum_k\log_2(1+\gamma_k)  
\neq  
\log_2  
\left(  
1+  
\frac1K\sum_k\gamma_k  
\right)  
}  
$$

因为：

$$  
\log_2(1+x)  
$$

是非线性的。

---

# 11. 一个具体例子：为什么平均 SINR 会骗人？

假设两个子载波。

方案 A：

$$  
\gamma_1=20,  
\qquad  
\gamma_2=0.  
$$

平均：

$$  
\bar\gamma_A=10.  
$$

方案 B：

$$  
\gamma_1=10,  
\qquad  
\gamma_2=10.  
$$

同样：

$$  
\bar\gamma_B=10.  
$$

所以看平均 SINR：

$$  
\boxed{\text{A 和 B 一样}}  
$$

但 SE：

方案 A：

# $$  
R_A

\frac12  
\left[  
\log_2(21)+\log_2(1)  
\right]  
\approx2.20.  
$$

方案 B：

# $$  
R_B

\log_2(11)  
\approx3.46.  
$$

所以：

$$  
\boxed{  
R_B>R_A  
}  
$$

这说明：

> **宽带系统中，仅看平均 SINR 可能掩盖频率选择性。**

---

# 12. 所以什么时候用 SINR，什么时候用 SE？

## 12.1 研究物理机制：优先 SINR

如果你问：

- MA 为什么有增益？
    
- 多波束为什么有用？
    
- ZF 为什么 noise enhancement？
    
- interference suppression 到底提高多少？
    
- taps 是否改善输出质量？
    

优先：

$$  
\boxed{\text{Output SINR/SNR}}  
$$

因为它可以直接拆成：

$$  
\frac{\text{Signal}}  
{\text{Interference}+\text{Noise}}.  
$$

这特别适合分析“增益来源”。

---

## 12.2 研究最终系统性能：优先 SE

如果你问：

- 系统能传多少信息？
    
- 两种 receiver 谁的通信性能更高？
    
- 宽带每个子载波差异最终造成什么影响？
    
- 总体通信收益是多少？
    

使用：

$$  
\boxed{\text{Spectral Efficiency}}  
$$

---

# 13. 对我们当前项目怎么选？

我建议以后采用两层指标。

```text
MA / multi-beam / taps
        ↓
改变信号、干扰、噪声
        ↓
post-combining SNR/SINR
        ↓
log2(1+SINR)
        ↓
Spectral Efficiency
```

所以：

## 13.1 第一层：机制指标

$$  
\boxed{\mathrm{SINR}}  
$$

用来解释：

$$  
\Delta\theta  
$$

怎么影响 path separation，

$$  
B\Delta\tau  
$$

怎么影响 delay alignment，

MA 怎么影响：

$$  
\rho,\quad  
\operatorname{cond}(\mathbf A^H\mathbf A),  
$$

最后怎么影响 output SINR。

---

## 13.2 第二层：通信指标

# $$  
\boxed{  
R_{\rm avg}

\frac1K  
\sum_k  
\log_2(1+\gamma_k)  
}  
$$

用来说明：

> 这些物理层增益最终转换成了多少 bit/s/Hz。

---

# 14. 我们当前代码里的关系

你现在代码其实正好就是：

```text
Receiver architecture
│
├─ Single beam
├─ Multi-beam no taps
├─ Multi-beam + taps
└─ Full digital
        ↓
每个子载波得到 snr[k]
        ↓
        ├───────────────┐
        ↓               ↓
机制分析             通信性能
SNR[k]              log2(1+SNR[k])
                        ↓
                    average over k
                        ↓
                       SE
```

所以当前代码中：

```matlab
snr_tap(k)
```

是：

$$  
\boxed{\text{第 }k\text{ 个子载波的输出接收质量}}  
$$

而：

```matlab
R_MB_tap = mean(log2(1 + snr_tap));
```

是：

$$  
\boxed{  
\text{该接收结构在整个 OFDM 带宽上的平均 achievable SE}  
}  
$$

---

# 15. 一个需要特别注意的地方：SE 不等于实际吞吐量

我们现在：

$$  
R=\log_2(1+\mathrm{SINR})  
$$

本质上是：

$$  
\boxed{\text{ideal achievable spectral efficiency}}  
$$

它假设：

- 高斯码本；
    
- 足够长编码；
    
- 理想接收；
    
- interference 可视作 Gaussian noise；
    
- 没有有限调制阶数限制；
    
- 没算 CP overhead；
    
- 没算 pilot overhead；
    
- 没算 coding gap；
    
- 没算硬件损伤。
    

实际系统可能使用：

$$  
64\text{-QAM},  
$$

那么最多：

$$  
\log_2 64=6  
$$

bits/symbol。

即使：

$$  
\log_2(1+\mathrm{SINR})=10,  
$$

真实调制也不一定能达到：

$$  
10,\mathrm{bit/s/Hz}.  
$$

所以论文中最好叫：

$$  
\boxed{\text{achievable spectral efficiency}}  
$$

而不是直接写：

$$  
\boxed{\text{actual throughput}}  
$$

---

# 16. 最终速记版

以后你看到两个指标，可以这样判断：

```text
我现在想知道：
│
├─ “接收效果到底好不好？”
│       ↓
│     SNR / SINR
│
├─ “为什么好？”
│       ↓
│     S / I / N 分解
│     SINR最合适
│
├─ “最终能多传多少信息？”
│       ↓
│     Spectral Efficiency
│
└─ “宽带多个子载波总体怎么样？”
        ↓
      Σ log2(1+SINR[k])
```

最终关系可以压缩成：

$$  
\boxed{  
\underbrace{  
\text{MA / Beam / Tap}  
}_{\text{设计变量}}  
\rightarrow  
\underbrace{  
\mathrm{SINR}  
}_{\text{物理层接收质量}}  
\rightarrow  
\underbrace{  
\log_2(1+\mathrm{SINR})  
}_{\text{信息论映射}}  
\rightarrow  
\underbrace{  
\mathrm{SE}  
}_{\text{频谱利用效率}}  
}  
$$

对我们当前这条研究线，我会把 **Output SINR/SNR 当作“解释机制的主指标”**，把 **Achievable SE 当作“说明最终通信收益的系统指标”**。两者最好同时保留，而不是二选一。