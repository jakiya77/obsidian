### 哪些问题不是凸问题
- **书名：** _Convex Optimization_
    
- **章节：** 第 3 章 (Convex Functions)
    
- **内容：**
    
    - 凸函数的定义是 $f(\theta x + (1-\theta)y) \le \theta f(x) + (1-\theta)f(y)$。
        
    - 凸函数的非负加权和仍然是凸函数（$f + g$ 是凸的）。
        
    - 但是**凸函数减去凸函数**（$f - g$）破坏了不等式方向，因此结果是不确定的。
        

![Difference of Convex Functions graph的图片](https://encrypted-tbn3.gstatic.com/licensed-image?q=tbn:ANd9GcR3a7QIylb2WKaen9HjaBMRNzqWkP6yDDK7zI5Etv74X5O0rQdvZaZGFFkjdE8OIEBjTiFFCoyccnEphrd-wz9YiEbmQqv3JYigJvlkGPsJFdzPEGk)


直观理解：

想象有两个开口向上的抛物线（凸函数）。

- $f(x) = x^2$ (陡峭)
    
- $g(x) = 0.5x^2$ (平缓)
    
- $f - g = 0.5x^2$ (还是凸的)
    
- 但如果 $g(x) = 2x^2$ (更陡峭)，那么 $f - g = -x^2$ (变成开口向下的凹函数了)。
    
- 更复杂的情况：如果 $f$ 和 $g$ 中心不同，相减后可能变成一个**波浪形**（W shape），既有山峰又有山谷，这就是典型的非凸（Non-convex）。

### 如何处理非凸问题
####  1. 利用 Fenchel 共轭变换将非凸转化为凸问题
