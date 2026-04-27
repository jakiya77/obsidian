# 基于 HPBW 的自适应蛇形扫描（Obsidian 笔记稿）

## 📌 概要

- 在角域以**近似等分辨率**方式采样 (θ,ϕ) 点，兼顾精度与扫描效率。
    
- **口径决定分辨率，分辨率决定步距**。以 HPBW 作为角域步距的量级基准；方位向考虑 cosθ 的投影效应。
    

---

## 角度定义与阵面口径

- 约定：θ 为仰角（0度指向z轴正方向，90度落到 xy 平面），ϕ 为方位角（x轴为 0度，逆时针为正）。
    
- 阵面等效口径：  
    $$  
    D_x=(N_x-1)d,\quad D_y=(N_y-1)d.  
    $$
    

---

## 核心原理

- HPBW 经验式（小角度、线性相位孔径近似）：  
    $$  
    \mathrm{HPBW}(D)\approx\frac{0.886\lambda}{D}.  
    $$
    
- 方位向的cosθ  因子来自单位球面的第一基本形式（在本角度定义下）：  
    $$  
    \mathrm{d}s^2=\mathrm{d}\theta^2+\cos^2\theta\mathrm{d}\phi^2,  
    $$  
    意味着同样的dφ在大的 θ 处对应更短的弧长（等效口径投影Dx​cosθ 更小），**方位分辨率变差**、HPBW 变宽。
    

---

## 核心公式（= 步距量级）

- 仰角方向：  
    $$  
    \Delta\theta\ \sim\ \mathrm{HPBW}_\theta\ \approx\ \frac{0.886\lambda}{D_y}.  
    $$
    
- 方位方向（随 θ变化）：  
    $$  
    \Delta\phi(\theta)\ \sim\ \mathrm{HPBW}_\phi(\theta)\ \approx\ \frac{0.886\lambda}{D_x\cos\theta}.  
    $$
    

$$
\Delta\theta \sim \mathrm{HPBW}_{\theta} \approx \frac{0.886\,\lambda}{D_y}~~~~~~~~~\Delta\phi(\theta) \sim \mathrm{HPBW}_{\phi}(\theta) \approx \frac{0.886\,\lambda}{D_x\cos\theta}.
$$

---

## 扫描流程（高层）

1. **设步距**：按上式得到 (\Delta\theta)、(\Delta\phi(\theta))。
    
2. **逐圈取样**：以 (\Delta\theta) 取“仰角圈”；每圈上 (\phi\in[0^\circ,360^\circ)) 以 (\Delta\phi(\theta)) 均匀采样。[[]]
    
3. **蛇形遍历**：相邻仰角圈交替反转 (\phi) 的遍历顺序（偶圈正序，奇圈逆序），降低路径跳变。
    
4. **分辨率标注**：为每个 ((\theta,\phi)) 记录 (\mathrm{HPBW}_\theta,\ \mathrm{HPBW}_\phi(\theta)) 作为后续自适应加密/插值依据。
    

--