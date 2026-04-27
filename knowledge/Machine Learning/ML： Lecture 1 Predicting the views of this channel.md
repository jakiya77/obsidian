>[!tldr]+ 
>1. 3 types of functions
>2. label ; model ; feature ;weight; bias
>3. Loss function definition and optimization process
>4. Linear Model would cause performance limitation 

 Different types of functions
 1. Regression : The function outputs a scalar
 2. Classification:Given options, the function output the correct one. \alpha go spam mail recognition
 3. structure learning: create something which structure(image and documents)

## how to find 

1. Function with unknown parameters ![[png： image 20260406151018.png]] 
2. Define Loss from Training Data![[png： image 20260406152137.png]]
    * The real value is" label "    
    ![[png： image 20260406152623.png]]
    ![[png： image 20260406152912.png]]
3. Optimization![[png： image 20260406153600.png]]
    * Update w iteratively
    * ![[png： image 20260406153850.png|335]]
    * ![[png： image 20260406154154.png]]
    * ![[png： image 20260406154313.png]]
![[png： image 20260406154639.png]]
![[png： image 20260406154939.png]]
> [!hint]- Lagging Effect
> 1. 蓝色的线比红色的延迟一天，这是由于模型在模仿过去的数据，进行拟合，而不是真正学习到东西。
> 2. 需要根据周期 比如七天来进行模型的调整，一个月诸如此类来调整模型

![[png： image 20260406160709.png]]
 
    当周期数增长之后，整体效果并没有得到什么改善
    之所以出现这种现象是由于模型是一些线性型，太简单了导致预测效果出现了平台
