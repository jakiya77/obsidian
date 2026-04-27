### II. SYSTEM MODEL

Consider an adversary suppressive jamming scenario where a single-antenna transmitter communicates with a legitimate receiver in the presence of a single-antenna jammer. To achieve high-accuracy channel estimation and robust jamming mitigation with low hardware overhead, the receiver is equipped with a single-RF dynamic metasurface antenna (DMA) consisting of $N_e$ metamaterial elements.

#### A. Signal Reception Model

We assume a flat fading multipath channel model for both the legitimate communication link and the jamming link. Let $\mathbf{h} \in \mathbb{C}^{N_e \times 1}$ and $\mathbf{g} \in \mathbb{C}^{N_e \times 1}$ denote the channel vectors from the transmitter and the jammer to the receiver's DMA, respectively. The received radio frequency (RF) signal vector $\mathbf{y}(t) \in \mathbb{C}^{N_e \times 1}$ at the DMA array can be expressed as:

$$\mathbf{y}(t) = \mathbf{h} x_T(t) + \mathbf{g} x_J(t) + \mathbf{z}(t)$$

where $x_T(t)$ and $x_J(t)$ represent the transmitted legitimate signal and the suppressive jamming signal, respectively. $\mathbf{z}(t) \sim \mathcal{CN}(\mathbf{0}, \sigma_z^2 \mathbf{I}_{N_e})$ is the additive white Gaussian noise (AWGN) vector.

During the pilot transmission phase, the transmitter and the jammer send $K$ symbols. By collecting the observations over $K$ time slots, the received RF signal matrix $\mathbf{Y} \in \mathbb{C}^{N_e \times K}$ is given by:

$$\mathbf{Y} = \mathbf{h} \mathbf{x}_T^T + \mathbf{g} \mathbf{x}_J^T + \mathbf{Z}$$

where $\mathbf{x}_T \in \mathbb{C}^{K \times 1}$ and $\mathbf{x}_J \in \mathbb{C}^{K \times 1}$ are the pilot sequences, and $\mathbf{Z} \in \mathbb{C}^{N_e \times K}$ is the noise matrix.

#### B. Amplitude-and-Phase Tunable DMA Architecture

Unlike conventional conventional phase-only antenna arrays or fully-digital massive MIMO systems, the proposed receiver utilizes a single-RF DMA architecture to dramatically reduce the power consumption and the required number of high-resolution analog-to-digital converters (ADCs). Specifically, the radiation pattern of the DMA can be rapidly reconfigured within a single symbol duration.

Let $M$ denote the number of reconfigurable sub-time slots within one symbol duration. The DMA configuration matrix over $M$ states is denoted as $\mathbf{\Phi} = [\boldsymbol{\phi}_1, \dots, \boldsymbol{\phi}_M]^T \in \mathbb{C}^{M \times N_e}$, where $\boldsymbol{\phi}_m = [\phi_{m,1}, \dots, \phi_{m,N_e}]^T \in \mathbb{C}^{N_e \times 1}$ represents the radiation response vector of the DMA at the $m$-th state.

A key distinction of our proposed architecture from existing pure-phase DMA models is the adoption of an advanced **amplitude-and-phase tunable** design. For the $n$-th metamaterial element at the $m$-th state, its complex response is constrained by:

$$|\phi_{m,n}| \le 1, \quad \forall m \in \{1,\dots,M\}, \ \forall n \in \{1,\dots,N_e\}$$

This convex unit-disc constraint mathematically encapsulates the passive and lossy nature of the metamaterial elements. It implies that the DMA can independently adjust the phase and attenuate the amplitude of the incident electromagnetic waves, without active power amplification. This extended degree of freedom is crucial for synthesizing deep spatial nulls at the RF front-end to prevent ADC saturation under suppressive jamming.

The equivalent baseband signal matrix $\mathbf{Q} \in \mathbb{C}^{M \times K}$ sampled by the single RF chain after $M$ rapid pattern switchings is given by:

$$\mathbf{Q} = \mathbf{\Phi} \mathbf{Y} = \mathbf{\Phi} \mathbf{h} \mathbf{x}_T^T + \mathbf{\Phi} \mathbf{g} \mathbf{x}_J^T + \mathbf{\Phi} \mathbf{Z}$$

#### C. Dual-Layer Analog-Digital Processing Framework

To overcome the inherent magnitude mismatch caused by the purely analog correlation and to achieve true minimum mean square error (MMSE) reception, we propose a joint analog-digital dual-layer processing framework.

Upon acquiring the low-dimensional observation matrix $\mathbf{Q}$, a digital combiner $\mathbf{w} \in \mathbb{C}^{M \times 1}$ is applied in the baseband. To account for potential digital hardware constraints, the weight vector is bounded by $|w_m| \le 1, \forall m$. Furthermore, to compensate for the significant array gain accumulation ($O(N_e)$) introduced by the analog summation and to align the signal energy with the reference pilot scale, an unconstrained digital automatic gain control (AGC) scaling factor $\beta > 0$ is explicitly introduced.

The final estimated symbol sequence $\hat{\mathbf{x}}_T^T \in \mathbb{C}^{1 \times K}$ through the proposed dual-layer architecture is formulated as:

$$\hat{\mathbf{x}}_T^T = \beta \mathbf{w}^H \mathbf{Q} = \beta \mathbf{w}^H \mathbf{\Phi} \mathbf{Y}$$

where the amplitude-and-phase tunable matrix $\mathbf{\Phi}$ acts as a coarse spatial analog filter to mitigate strong jamming prior to the ADC, while the digital combination variables $\{\mathbf{w}, \beta\}$ serve as an exact fine-tuning equalizer to guarantee the matching performance.

---

### III. PROBLEM FORMULATION AND MOTIVATION

#### A. Bottleneck of Conventional Phase-Only Matching

In conventional single-RF DMA or RIS-aided receiver architectures, the interference mitigation and signal matching are typically formulated as a direct minimum mean square error (MMSE) problem executed solely by the analog metamaterial elements. Let $\boldsymbol{\phi}_{opt}$ denote the ideal unconstrained combining vector. The conventional formulation attempts to map $\boldsymbol{\phi}_{opt}$ onto a discrete phase-only codebook, essentially solving:

$$\min_{\boldsymbol{\phi}} \| \mathbf{x}_T^T - \boldsymbol{\phi}^T \mathbf{Y} \|_2^2 \quad \text{s.t.} \ |\phi_n| = 1, \forall n.$$

However, this conventional paradigm suffers from a fundamental magnitude mismatch. The received RF signal $\mathbf{Y}$ inherently contains a large array gain scaled by $O(N_e)$ due to the spatial coherent summation of $N_e$ elements. Conversely, the reference pilot sequence $\mathbf{x}_T^T$ is typically of the $O(1)$ magnitude scale. Imposing a strict constant-modulus constraint ($|\phi_n| = 1$) deprives the analog network of any amplitude scaling capability. Consequently, forcibly matching the $O(N_e)$ signal to an $O(1)$ target using purely passive phase shifters leads to severe projection errors, significantly degrading the depth of spatial nulls and rendering the interference suppression ineffective unless an excessively long pilot sequence is utilized to average out the residual errors.

#### B. Proposed Joint Dual-Layer MMSE Formulation

To fundamentally resolve the magnitude mismatch and unleash the full potential of DMA for suppressive jamming mitigation, we formulate a joint analog-digital dual-layer MMSE optimization problem. Instead of relying solely on the rigid analog front-end, we distribute the spatial filtering and matching tasks across the amplitude-and-phase tunable DMA and the digital baseband.

The objective is to minimize the error between the transmitted pilot sequence and the reconstructed signal after the dual-layer processing. The proposed optimization problem is mathematically formulated as:

$$\min_{\mathbf{\Phi}, \mathbf{w}, \beta} \mathcal{J}(\mathbf{\Phi}, \mathbf{w}, \beta) = \left\| \mathbf{x}_T^T - \beta \mathbf{w}^H \mathbf{\Phi} \mathbf{Y} \right\|_2^2$$

$$\text{s.t.} \quad |\Phi_{m,n}| \le 1, \quad \forall m \in \{1,\dots,M\}, \ n \in \{1,\dots,N_e\} \quad (16\text{a})$$

$$|w_m| \le 1, \quad \forall m \in \{1,\dots,M\} \quad (16\text{b})$$

$$\beta > 0 \quad (16\text{c})$$

In this joint formulation, each variable plays a distinct and physically meaningful role:

1. **Analog Coarse Filtering ($\mathbf{\Phi}$):** Constrained by the passive unit-disc condition in (16a), the DMA leverages its amplitude attenuation and phase shifting capabilities to synthesize deep physical nulls toward the jammer. This crucial step reduces the high-power jamming signal to fall within the linear dynamic range of the ADCs.
    
2. **Digital Fine-Tuning ($\mathbf{w}$):** The digital weight vector in (16b) coherently combines the $M$ sub-time slot observations to eliminate residual jamming and compensate for analog hardware imperfections.
    
3. **Gain Alignment ($\beta$):** The unconstrained scalar $\beta$ in (16c) acts as an automatic gain control (AGC) factor, elegantly absorbing the $O(N_e)$ array gain and perfectly aligning the signal magnitudes without corrupting the spatial subspace optimization.
    

Remarkably, from an optimization perspective, relaxing the non-convex constant-modulus constraint ($|\phi_{m,n}| = 1$) to the continuous passive constraint ($|\Phi_{m,n}| \le 1$) transforms the feasible set from a highly non-convex ring to a mathematically tractable convex solid disc. As will be detailed in the following section, this transformation enables the derivation of highly efficient alternating optimization algorithms involving complex-variable derivatives (e.g., Wirtinger calculus), preventing the solution from being trapped in severe local optima.


---

## IV. PROPOSED JOINT DUAL-LAYER OPTIMIZATION ALGORITHM

The joint MMSE optimization problem formulated in (16) is highly non-convex due to the strong coupling between the analog DMA matrix $\mathbf{\Phi}$ and the digital variables $\{\mathbf{w}, \beta\}$. Finding a global optimal solution directly is mathematically intractable. To tackle this challenge, we propose an efficient Alternating Optimization (AO) framework that decouples the original problem into two tractable subproblems, solving them iteratively until convergence.

### A. Optimal Digital Baseband Design

In the first subproblem, we optimize the digital fine-tuning weight $\mathbf{w}$ and the AGC factor $\beta$ while keeping the analog DMA matrix $\mathbf{\Phi}$ fixed. Given a fixed $\mathbf{\Phi}$, we define the equivalent baseband signal matrix as $\mathbf{Y}_{eq} = \mathbf{\Phi} \mathbf{Y}$. The subproblem is reduced to:

$$\min_{\mathbf{w}, \beta} \quad \left\| \mathbf{x}_T^T - \beta \mathbf{w}^H \mathbf{Y}_{eq} \right\|_2^2$$

$$\text{s.t.} \quad |w_m| \le 1, \quad \forall m$$

$$\beta > 0$$

To efficiently solve this, we introduce an unconstrained auxiliary vector $\tilde{\mathbf{w}} = \beta \mathbf{w}$. The problem is then transformed into a standard unconstrained Least Squares (LS) problem with respect to $\tilde{\mathbf{w}}$:

$$\min_{\tilde{\mathbf{w}}} \left\| \mathbf{x}_T^T - \tilde{\mathbf{w}}^H \mathbf{Y}_{eq} \right\|_2^2$$

Setting the complex derivative of the objective function with respect to $\tilde{\mathbf{w}}^*$ to zero, we obtain the optimal closed-form solution:

$$\tilde{\mathbf{w}}^{\star} = (\mathbf{Y}_{eq}\mathbf{Y}_{eq}^H)^{-1} \mathbf{Y}_{eq} (\mathbf{x}_T^T)^H$$

After obtaining the unconstrained global optimum $\tilde{\mathbf{w}}^{\star}$, we must map it back to satisfy the digital hardware constraints. The optimal AGC factor $\beta^{\star}$ and the bounded digital weight $\mathbf{w}^{\star}$ are perfectly recovered by extracting the maximum amplitude from $\tilde{\mathbf{w}}^{\star}$:

$$\beta^{\star} = \max_m |\tilde{w}^{\star}_m|$$

$$\mathbf{w}^{\star} = \frac{\tilde{\mathbf{w}}^{\star}}{\beta^{\star}}$$

This substitution guarantees that the element with the maximum magnitude in $\mathbf{w}^{\star}$ reaches exactly 1, ensuring full utilization of the digital dynamic range while strictly satisfying the $|w_m| \le 1$ constraint.

### B. Optimal Analog DMA Design via PGD

In the second subproblem, we optimize the amplitude-and-phase tunable DMA matrix $\mathbf{\Phi}$ while keeping $\mathbf{w}$ and $\beta$ fixed. The optimization problem is cast as:

$$\min_{\mathbf{\Phi}} \ f(\mathbf{\Phi}) = \left\| \mathbf{x}_T^T - \beta \mathbf{w}^H \mathbf{\Phi} \mathbf{Y} \right\|_F^2 \quad \text{s.t.} \ |\Phi_{m,n}| \le 1$$

Unlike traditional constant-modulus constraints that result in NP-hard non-convex spaces, our passive unit-disc constraint $|\Phi_{m,n}| \le 1$ forms a convex set. Since the objective function $f(\mathbf{\Phi})$ is a convex quadratic function with respect to $\mathbf{\Phi}$, this subproblem is a standard Convex Quadratic Program (CQP), guaranteeing a global optimum.

To solve it with low computational complexity, we adopt the Projected Gradient Descent (PGD) method. **A critical mathematical detail arises here regarding the complex gradient calculation.** Because our objective function $f(\mathbf{\Phi})$ is a real-valued scalar (a squared norm) defined over complex variables, it does not satisfy the Cauchy-Riemann equations, making it non-analytic (non-holomorphic). In the standard complex plane, its derivative simply does not exist.

To resolve this, we utilize **Wirtinger calculus**, which treats the complex variable $\mathbf{\Phi}$ and its complex conjugate $\mathbf{\Phi}^*$ as mathematically independent variables. In optimization theory, the direction of the steepest ascent for a real-valued function is strictly defined by the derivative with respect to its **conjugate complex number ($\mathbf{\Phi}^*$)**, rather than the variable itself ($\mathbf{\Phi}$). Therefore, computing the gradient relative to the conjugate matrix is not an approximation, but the mathematically rigorous way to find the optimal descent direction. The correct gradient is derived as:

$$\nabla_{\mathbf{\Phi}^*} f(\mathbf{\Phi}) = \beta \mathbf{w} \left( \beta \mathbf{w}^H \mathbf{\Phi} \mathbf{Y} - \mathbf{x}_T^T \right) \mathbf{Y}^H$$

In each iteration $i$, the DMA matrix is first updated along the negative gradient direction with a step size $\mu$:

$$\hat{\mathbf{\Phi}}^{(i+1)} = \mathbf{\Phi}^{(i)} - \mu \nabla_{\mathbf{\Phi}^*} f(\mathbf{\Phi}^{(i)})$$

Subsequently, an element-wise projection operator $\mathcal{P}_{\mathcal{C}}(\cdot)$ is applied to map the updated matrix back into the feasible convex disc space:

$$\Phi_{m,n}^{(i+1)} = \mathcal{P}_{\mathcal{C}} \left( \hat{\Phi}_{m,n}^{(i+1)} \right) = \frac{\hat{\Phi}_{m,n}^{(i+1)}}{\max \left( 1, |\hat{\Phi}_{m,n}^{(i+1)}| \right)}$$

This elegant projection mathematically ensures that elements exceeding the passive bound are scaled down to the boundary of the unit circle, while strictly preserving their phase information.

### C. Overall Algorithm

The overall proposed joint dual-layer optimization procedure is summarized in **Algorithm 1**. The algorithm iteratively updates the digital parameters and the analog DMA matrix until the fractional decrease of the objective value falls below a predefined threshold $\epsilon$. Due to the convex nature of the subproblems and the exact closed-form solution in the digital domain, the sequence of the objective values is monotonically non-increasing and bounded from below (by zero), thereby guaranteeing convergence.

> ## **Algorithm 1: Proposed Joint Dual-Layer AO Algorithm**
> 
> **Input:** Received RF matrix $\mathbf{Y}$, pilot sequence $\mathbf{x}_T^T$, max iterations $I_{max}$, tolerance $\epsilon$, PGD step size $\mu$.
> 
> **Initialize:** Set iteration index $t = 0$. Initialize $\mathbf{\Phi}^{(0)}$ with random phases and unit amplitudes.
> 
> **Repeat**
> 
> - **Step 1: Digital Baseband Optimization**
>     
>     1. Compute equivalent baseband matrix: $\mathbf{Y}_{eq} = \mathbf{\Phi}^{(t)} \mathbf{Y}$.
>         
>     2. Calculate unconstrained optimal weight $\tilde{\mathbf{w}}^{\star}$ via LS closed-form expression.
>         
>     3. Extract $\beta^{(t+1)} = \max_m |\tilde{w}^{\star}_m|$.
>         
>     4. Update digital combiner $\mathbf{w}^{(t+1)} = \tilde{\mathbf{w}}^{\star} / \beta^{(t+1)}$.
>         
> - **Step 2: Analog DMA Optimization via PGD**
>     
>     5. Set internal PGD index $i = 0$ and initial state $\mathbf{\Phi}_{PGD}^{(0)} = \mathbf{\Phi}^{(t)}$.
>     
>     6. **While** PGD not converged **do**
>     
>     * Compute complex gradient $\nabla_{\mathbf{\Phi}^*} f(\mathbf{\Phi}_{PGD}^{(i)})$.
>     
>     * Gradient descent step: $\hat{\mathbf{\Phi}}^{(i+1)} = \mathbf{\Phi}_{PGD}^{(i)} - \mu \nabla_{\mathbf{\Phi}^*} f(\mathbf{\Phi}_{PGD}^{(i)})$.
>     
>     * Apply convex projection: $\mathbf{\Phi}_{PGD}^{(i+1)} = \mathcal{P}_{\mathcal{C}} \left( \hat{\mathbf{\Phi}}^{(i+1)} \right)$.
>     
>     * $i \leftarrow i + 1$.
>     
>     7. **End While**
>     
>     8. Update DMA matrix $\mathbf{\Phi}^{(t+1)} = \mathbf{\Phi}_{PGD}^{(i)}$.
>     
> - $t \leftarrow t + 1$.
>     
> 
> **Until** Fractional decrease of objective function is less than $\epsilon$ or $t = I_{max}$.
> 
> **Output:** Optimized variables $\mathbf{\Phi}^{\star}$, $\mathbf{w}^{\star}$, and $\beta^{\star}$.


