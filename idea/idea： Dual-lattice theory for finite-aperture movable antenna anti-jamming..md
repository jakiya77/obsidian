期望信号和多径干扰在位置域分别形成可解析的相干/相消格点；MA 通过位置重构寻找 desired-constructive and jammer-destructive 的 dual-favorable positions；在有限孔径和最小间距约束下，这种位置是否存在、能放多少个、性能如何，是本文要解决的核心问题。

1. 推导 desired constructive lattice；
2. 推导 jammer destructive lattice；
3. 定义 exact dual-lattice intersection；
4. 说明 exact intersection 不一定存在；
5. 定义 approximate dual-favorable region；
6. 推导 finite-aperture dual-lattice capacity；
7. 再提出 dual-lattice score：

$$
η(p)=∣hs(p)∣2∣gj(p)∣2+ϵ.\eta(p)= \frac{|h_s(p)|^2} {|g_j(p)|^2+\epsilon}.η(p)=∣gj​(p)∣2+ϵ∣hs​(p)∣2​.
$$

多 jammer 版本：

$$
η(p)=∣hs(p)∣2∑qρq∣gq(p)∣2+ϵ.\eta(p)= \frac{|h_s(p)|^2} {\sum_q \rho_q |g_q(p)|^2+\epsilon}.η(p)=∑q​ρq​∣gq​(p)∣2+ϵ∣hs​(p)∣2​.
$$

最后才是：

> 选出满足间距约束的 NNN 个 MA 位置，再接 MRC/MMSE/MVDR 处理残余干扰。



## Fig 1 
