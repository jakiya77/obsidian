---
title: "Robust Predictive mmWave Beamforming for UAV Communications via Vision-Position Fusion"
authors: ["Nie Jiali", "Han Yu", "Cui Yuanhao", "Jin Shi", "Wen Chao-Kai"]
year: 2026
citation_key: nieRobustPredictiveMmWave2026
status: "reading"
collections: ["MA Multimodal"]
tags: [📝文献笔记, 🔖journalArticle]
---
[[dataset：Robust Predictive mmWave Beamforming for UAV Communications via Vision-Position Fusion]]

## 📚 元数据
- **作者**: Nie, Han, Cui, Jin, Wen
- **年份**: 2026
- **阅读状态**: reading
- **所属分类**: 📂MA Multimodal
- **期刊/出版社**: IEEE Transactions on Vehicular Technology
- **Zotero 链接**: [在 Zotero 中打开文献](zotero://select/items/bbt:nieRobustPredictiveMmWave2026)

## 📄 摘要
> The rapid growth of the low-altitude economy requires low-altitude wireless networks (LAWN) to provide reliable and high-throughput connectivity for uncrewed aerial vehicles (UAVs). However, the three-dimensional mobility of UAVs and the narrow beamwidth of millimeter-wave (mmWave) systems render conventional scan-based beam management inefficient, leading to severe beam aging and latency-induced misalignment. Existing methods primarily treat beam alignment as a reactive estimation problem, mapping instantaneous observations to the current optimal beam. In this work, we advocate a predictive paradigm shift from reactive beam selection to proactive beam forecasting. We first demonstrate that geometry-only beam selection based on global positioning system (GPS) and inertial measurement unit (IMU) data is restricted to reactive, current-slot alignment and degrades sharply under multi-step forecasting. Then we propose a Transformer-based multimodal predictive beamforming framework that fuses visual imagery at the ground base station (GBS) with UAV positional information. By exploiting self-attention with learnable positional embeddings, the model captures temporal dependencies and nonlinear motion dynamics to forecast future optimal beams. The proposed framework is validated using a real-world 60 GHz hardware measurement dataset. Experimental results demonstrate overall Top-1 and Top5 beam prediction accuracies of 81.9% and 99.3%, respectively. Moreover, under practical beam training overhead constraints, the proposed scheme achieves clear effective spectral efficiency gains over both conventional scan-based beam management and learning-based baselines, demonstrating the advantages of predictive multimodal beamforming for next-generation aerial networks.

---

## 📖 Zotero 独立笔记


---

## 📝 阅读笔记与高亮

### 🔴 核心论点 (结论/主要创新点)


> ground base station (p. 1)




![](<obsidian note/Paper Reading Assets/assets/nieRobustPredictiveMmWave2026/nieRobustPredictiveMmWave2026-3-x40-y541.png>)




![](<obsidian note/Paper Reading Assets/assets/nieRobustPredictiveMmWave2026/nieRobustPredictiveMmWave2026-3-x361-y69.png>)




![](<obsidian note/Paper Reading Assets/assets/nieRobustPredictiveMmWave2026/nieRobustPredictiveMmWave2026-4-x111-y423.png>)


- 💡 **批注**: 每个时刻都可以得到这个东西，包含了三个量：
C(t)：GBS 摄像头拍到的 RGB 图像；  G(t)：UAV 的 GPS 经纬度；  
Z(t)：高度、速度、姿态等 IMU 信息。但是实际用到的是RGB image + GPS + altitude认同感。



> where C (t) represents the RGB image captured by the static camera mounted at the GBS, G (t) denotes the horizontal GPS coordinates (latitude and longitude) of the UAV, and Z (t) contains the UAV’s onboard inertial measurements, including altitude, velocity, and attitude parameters (e.g., roll and pitch). The set D (t) therefore forms a multimodal subset of sensory data available to the GBS (p. 4)




### 🟡 重要细节 (研究方法/支撑数据)


> ground base station (p. 1)




![](<obsidian note/Paper Reading Assets/assets/nieRobustPredictiveMmWave2026/nieRobustPredictiveMmWave2026-3-x40-y541.png>)




![](<obsidian note/Paper Reading Assets/assets/nieRobustPredictiveMmWave2026/nieRobustPredictiveMmWave2026-3-x361-y69.png>)




![](<obsidian note/Paper Reading Assets/assets/nieRobustPredictiveMmWave2026/nieRobustPredictiveMmWave2026-4-x111-y423.png>)


- 💡 **批注**: 每个时刻都可以得到这个东西，包含了三个量：
C(t)：GBS 摄像头拍到的 RGB 图像；  G(t)：UAV 的 GPS 经纬度；  
Z(t)：高度、速度、姿态等 IMU 信息。但是实际用到的是RGB image + GPS + altitude认同感。



> where C (t) represents the RGB image captured by the static camera mounted at the GBS, G (t) denotes the horizontal GPS coordinates (latitude and longitude) of the UAV, and Z (t) contains the UAV’s onboard inertial measurements, including altitude, velocity, and attitude parameters (e.g., roll and pitch). The set D (t) therefore forms a multimodal subset of sensory data available to the GBS (p. 4)




### 🔵 疑问与扩展 (待查阅/难以理解的概念)


> ground base station (p. 1)




![](<obsidian note/Paper Reading Assets/assets/nieRobustPredictiveMmWave2026/nieRobustPredictiveMmWave2026-3-x40-y541.png>)




![](<obsidian note/Paper Reading Assets/assets/nieRobustPredictiveMmWave2026/nieRobustPredictiveMmWave2026-3-x361-y69.png>)




![](<obsidian note/Paper Reading Assets/assets/nieRobustPredictiveMmWave2026/nieRobustPredictiveMmWave2026-4-x111-y423.png>)


- 💡 **批注**: 每个时刻都可以得到这个东西，包含了三个量：
C(t)：GBS 摄像头拍到的 RGB 图像；  
G(t)：UAV 的 GPS 经纬度；  
Z(t)：高度、速度、姿态等 IMU 信息。
但是实际用到的是RGB image + GPS + altitude



> where C (t) represents the RGB image captured by the static camera mounted at the GBS, G (t) denotes the horizontal GPS coordinates (latitude and longitude) of the UAV, and Z (t) contains the UAV’s onboard inertial measurements, including altitude, velocity, and attitude parameters (e.g., roll and pitch). The set D (t) therefore forms a multimodal subset of sensory data available to the GBS (p. 4)




### 🟢 个人启发 (可迁移的方法/灵感)


> ground base station (p. 1)




![](<obsidian note/Paper Reading Assets/assets/nieRobustPredictiveMmWave2026/nieRobustPredictiveMmWave2026-3-x40-y541.png>)




![](<obsidian note/Paper Reading Assets/assets/nieRobustPredictiveMmWave2026/nieRobustPredictiveMmWave2026-3-x361-y69.png>)




![](<obsidian note/Paper Reading Assets/assets/nieRobustPredictiveMmWave2026/nieRobustPredictiveMmWave2026-4-x111-y423.png>)


- 💡 **批注**: 每个时刻都可以得到这个东西，包含了三个量：
C(t)：GBS 摄像头拍到的 RGB 图像；  G(t)：UAV 的 GPS 经纬度；  
Z(t)：高度、速度、姿态等 IMU 信息。但是实际用到的是RGB image + GPS + altitude认同感。



> where C (t) represents the RGB image captured by the static camera mounted at the GBS, G (t) denotes the horizontal GPS coordinates (latitude and longitude) of the UAV, and Z (t) contains the UAV’s onboard inertial measurements, including altitude, velocity, and attitude parameters (e.g., roll and pitch). The set D (t) therefore forms a multimodal subset of sensory data available to the GBS (p. 4)


