**期刊信息：IEEE COMMUNICATIONS SURVEYS & TUTORIALS, VOL. 25, NO. 2, SECOND QUARTER 2023

information security protection ：过去对于信息安全保护一般分为PLS（physical layer security）、steganography（速写加密）这两类；但是两种手段存在问题，他们只能以一种混乱的顺序保护信息，但是如果窃听者的解码技术进步很快，则可以做到解码。故而我们引入了隐蔽通信（covert communciations）一种让willie发现不了有信息在传输的技术。covert communications主要采取的是**不确定性手段**。
![[截屏2026-01-16 11.32.39.png]]
不确定性手段：是一类**利用随机性来混淆敌方检测器**的核心理论与方法。简单来说，它的核心逻辑是：**如果要让敌人无法判断“这里有没有信号”，最好的办法是让环境本身就充满了“捉摸不透的波动”。**
三种技术的直观区别：
![[Pasted image 20260116155002.png]]

### covert communication的历史：
1. 二十世纪初期，二战期间扩频一直被使用，但是fundamental performance limitation 从来都未被分析过
2. 1996年提出了s low probability of detection (LPD)--使用的是spread spectrum（扩频），隐蔽通信 (Covert Communication/LPD) 作为一个正式的学术概念或研究领域正式emerged
3. 2013年B. A. Bash 、D. Goeckel, and D. Towsley, 在论文“Limits of reliable communication with low probability of detection on AWGN channels,” IEEE J. Sel. Areas Commun., vol. 31,no. 9, pp. 1921–1930, Sep. 2013.中提出了揭示并量化了扩频在隐蔽通信中的性能限制->square root law(SRL)


### covert communication的模型
![[Pasted image 20260118144622.png]]

Alice 按照概率1 给发送信息，概率2保持沉默；在接收到的功率信息通过一个threshold判断Ailce是否发送消息了：![[截屏2026-01-18 14.48.00.png]]
![[截屏2026-01-18 14.48.41.png]]根据判断的结果和实际情况，可以分成以下FA(false alarm) 和MD（miss detection）
* FA  α = Pr{D1|H0} 没发以为发了
* MD β = Pr{D0|H1} 发了以为没发
*  Willie端的目的是通过设置不同的Γ 来 min α+β 
* ![[截屏2026-01-18 14.54.37.png]]
* 这里的全变分距离 (VT​)： **Total Variation Distance**：衡量 P0​（只有噪声的分布）和 P1​（信号+噪声的分布）这就两个分布长得**有多不一样**。

- 如果两个分布完全重合，VT​=0，那么错误率 =1−0=1（Willie 瞎猜）。
    
- 如果两个分布分得很开，VT​ 很大，错误率就变小了（Willie 能看出来了）[[Telecom：全分变距离Total Variation Distance;散度；相对熵]]
- ![[Pasted image 20260118165048.png]]
- 处理具体问题时候的最情况假设：假设willie拥有最优门限Γ∗，在此基础上联合优化发射功率、擦欢呼速率