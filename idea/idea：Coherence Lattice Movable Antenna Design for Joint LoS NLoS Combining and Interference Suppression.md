Coherence-Lattice Movable Antenna Design for Joint LoS/NLoS Combining and Interference Suppression

先利用 LoS/NLoS 两条主导路径的相位结构，把多径从“衰落源”变成“相干增益”；再在有干扰时，把 coherence-lattice 设计扩展成 interference-aware wide-null MA，实现相干合并和干扰抑制的折中。

**主线故事**

这批图最适合讲一个两阶段故事：

> 先利用 LoS/NLoS 两条主导路径的相位结构，把多径从“衰落源”变成“相干增益”；再在有干扰时，把 coherence-lattice 设计扩展成 interference-aware wide-null MA，实现相干合并和干扰抑制的折中。

所以论文主线可以叫：

> Coherence-Lattice Movable Antenna Design for Joint LoS/NLoS Combining and Interference Suppression

**第一组：为什么 MA 能提升两径接收增益**

图：[exp1a_coherence_identity.png](/Users/jiaqix/Documents/Codex/2026-05-20/1-n-los-nlos-multipath-mathbf/ma_paper_extension_code/figures/exp1a_coherence_identity.png)

目的：说明 coherence lattice 的物理机制。

它说明了：

- 固定 ULA 的每个天线位置上，LoS/NLoS 相位有的相干、有的抵消。
- constructive lattice 能让每个阵元的相干因子接近 1。
- destructive lattice 是反例，SNR 会非常差。
- 接收功率可以被分解成 direct power 和 expanded expression，说明公式推导和数值一致。

论文里可以说：

> The MA gain originates from element-wise LoS/NLoS phase alignment, not from random aperture enlargement.
![[Pasted image 20260522154154.png]]

**第二组：什么时候能找到足够多的相干位置**

图：[exp1b_finite_aperture_feasibility.png](/Users/jiaqix/Documents/Codex/2026-05-20/1-n-los-nlos-multipath-mathbf/ma_paper_extension_code/figures/exp1b_finite_aperture_feasibility.png)

目的：说明 coherence lattice 不是随便都能用，它受孔径、最小间距和角度间隔限制。

它说明了：

- aperture 越大，可容纳的 fully coherent antennas 越多。
- AoA separation 越大，`T_coh` 越小，相干格点越密。
- `d_min` 和 aperture 共同决定 `N_coh^max`。

论文里可以说：

> The proposed design has an explicit feasibility condition determined by aperture, minimum antenna spacing, and LoS/NLoS angular separation.

**第三组：误差下是否还稳**

图：[exp2a_coherence_robustness.png](/Users/jiaqix/Documents/Codex/2026-05-20/1-n-los-nlos-multipath-mathbf/ma_paper_extension_code/figures/exp2a_coherence_robustness.png)

目的：验证 AoA 估计误差、相位误差、位置量化、互耦不会让方法立刻失效。

它说明了：

- AoA error 1 deg 时损失约 0.1 dB。
- phase error 20 deg 时损失约 0.25 dB。
- position quantization 到 0.1 lambda 时损失仍很小。
- 误差越大，mean coherence 下降，SNR loss 上升，趋势符合理论。

论文里可以说：

> The lattice design is robust to moderate CSI and position errors because approximate coherence still preserves most of the coherent gain.

**第四组：性能 scaling 和复杂度**

图：[exp3a_snr_vs_N.png](/Users/jiaqix/Documents/Codex/2026-05-20/1-n-los-nlos-multipath-mathbf/ma_paper_extension_code/figures/exp3a_snr_vs_N.png)  
图：[exp3b_snr_vs_aperture.png](/Users/jiaqix/Documents/Codex/2026-05-20/1-n-los-nlos-multipath-mathbf/ma_paper_extension_code/figures/exp3b_snr_vs_aperture.png)  
图：[exp3e_runtime.png](/Users/jiaqix/Documents/Codex/2026-05-20/1-n-los-nlos-multipath-mathbf/ma_paper_extension_code/figures/exp3e_runtime.png)

目的：证明 coherence lattice 不只是机制正确，而且性能明显优于 fixed/random MA，复杂度低。

它说明了：

- `exp3a` 中 coherence lattice 比 fixed/random 大约高 3 dB。
- coherence lattice 和 PGD 结果重合，但这里要谨慎写，因为 PGD 使用了 lattice 初始化。
- `exp3b` 说明 aperture 增大后，相干位置变多，SNR 提升。
- `exp3e` 说明 lattice 构造比 dual-beam local search 快很多。

论文里建议说：

> Under the dominant two-path MRC objective, the coherence-lattice construction attains the same objective value as the PGD validation solver, while avoiding iterative position search.

不要说“接近 PGD”，应该说“达到 PGD validation 的同样目标值”。

**第五组：车联网动态场景，作为应用验证**

图：[exp4a_vehicular_time_varying_snr.png](/Users/jiaqix/Documents/Codex/2026-05-20/1-n-los-nlos-multipath-mathbf/ma_paper_extension_code/figures/exp4a_vehicular_time_varying_snr.png)  
图：[exp4b_rician_multipath_extension.png](/Users/jiaqix/Documents/Codex/2026-05-20/1-n-los-nlos-multipath-mathbf/ma_paper_extension_code/figures/exp4b_rician_multipath_extension.png)

目的：说明这个方法可以放到 vehicular LoS/NLoS 场景里，但它不是核心理论。

它说明了：

- 车辆移动时 LoS/NLoS AoA 和 `T_coh` 慢变。
- tracking lattice 平均比 fixed/random 高约 1.7 到 1.8 dB。
- stale lattice 会因为位置更新不及时而掉性能。
- Rician K-factor 和弱多径扩展下，coherence lattice 仍保持优势。

论文里可以说：

> In slowly varying vehicular geometry, the proposed lattice can be updated according to dominant-path AoAs and maintains its gain under Rician two-path plus weak multipath channels.

这组图适合作为 application case，不建议作为主创新。

**第六组：加入干扰后，coherence-only 不够**

图：[exp5a_sinr_vs_inr.png](/Users/jiaqix/Documents/Codex/2026-05-20/1-n-los-nlos-multipath-mathbf/ma_paper_extension_code/figures/exp5a_sinr_vs_inr.png)  
图：[exp5e_metric_summary.png](/Users/jiaqix/Documents/Codex/2026-05-20/1-n-los-nlos-multipath-mathbf/ma_paper_extension_code/figures/exp5e_metric_summary.png)

目的：说明有强干扰时，单纯提升期望信号 SNR 不够，需要 wide-null 和 MA 位置联合设计。

它说明了：

- 低 INR 时，coherence lattice + MRC 很强，因为主要问题是期望信号增益。
- 高 INR 时，MRC 曲线迅速下降，因为没有抑制干扰。
- wide-null LCMV 曲线不随 INR 下降，说明干扰被压住了。
- interference-aware MA + wide-null LCMV 在 `INR = 30 dB` 时 SINR 最高，约 13.61 dB。
- 它比 fixed wide-null 高约 3.43 dB。

论文里可以说：

> Coherence-only reception is insufficient under strong directional interference. The interference-aware MA wide-null design achieves the best high-INR SINR by balancing desired signal gain, jammer leakage, and combiner noise gain.

**哪些图要谨慎使用**

[exp3d_dual_beam_vs_aperture.png](exp3d_dual_beam_vs_aperture.png.md)：可以说明 sidelobe 有改善，但 off-grid null 不总是 MA 更好，不建议做核心图。

[exp5c_beampatterns.png](/Users/jiaqix/Documents/Codex/2026-05-20/1-n-los-nlos-multipath-mathbf/ma_paper_extension_code/figures/exp5c_beampatterns.png)：只能辅助展示 null 在哪里，不适合判断谁最好。

[exp5d_gain_null_tradeoff.png](/Users/jiaqix/Documents/Codex/2026-05-20/1-n-los-nlos-multipath-mathbf/ma_paper_extension_code/figures/exp5d_gain_null_tradeoff.png)：现在不够直观，建议不要放主文。

**推荐主文图顺序**

1. `exp1a`：相干格点机制。
2. `exp1b`：可行性条件。
3. `exp3a`：SNR scaling。
4. `exp2a`：鲁棒性。
5. `exp5a`：强干扰下 SINR。
6. `exp5e`：综合指标解释为什么 aware MA wide-null 最好。

最终故事一句话：

> Coherence lattice provides a closed-form, geometry-driven way to exploit dominant LoS/NLoS multipath for SNR gain; when interference is present, the design can be extended with wide-null LCMV and MA position refinement to achieve the best SINR by trading off desired coherent gain and jammer suppression.