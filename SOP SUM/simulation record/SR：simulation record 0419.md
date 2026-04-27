1. 更改EF cos的幂 看一下效果的变化

| $cos^1(\theta)$                      | $cos^2(\theta)$                      | $cos^3(\theta)$                      | $cos^4(\theta)$                      |
| ------------------------------------ | ------------------------------------ | ------------------------------------ | ------------------------------------ |
| ![[Pasted image 20260419145903.png]] | ![[Pasted image 20260419145926.png]] | ![[Pasted image 20260419145945.png]] | ![[Pasted image 20260419150037.png]] |
| ![[png：最好情况.png]]                    | ![[png-rangular-cos2.png]]           | ![[png-rangular-cos3.png]]           | ![[png-regular-cos4.png]]            |
| ![[png：最坏情况cos1.png]]                | ![[png-worst-cos2.png]]              | ![[png-worst-cos3.png]]              | ![[png-worst-cos4.png]]              |
| ![[Pasted image 20260419150155.png]] | ![[Pasted image 20260419150224.png]] | ![[Pasted image 20260419150013.png]] | ![[Pasted image 20260419150119.png]] |
|                                      |                                      |                                      |                                      |
|                                      |                                      |                                      |                                      |

2. 更改EF 可以选择的角度 

| [-45, -20, 0, 20, 45] | <br>![[Pasted image 20260419141932.png\|390]] |
| --------------------- | --------------------------------------------- |
| [-45, 0, 20]          | ![[Pasted image 20260419142529.png\|387]]\|   |
| [-20, 0, 20 ]         | ![[Pasted image 20260419142425.png\|384]]     |
| [-45, 0, 45]          | ![[Pasted image 20260419142324.png\|390]]     |
|                       |                                               |
>[!error]+现在没有办法得到一个很稳定的结果 不同次的montecarlo1000次实验相差1-2个dB
>![[Pasted image 20260419145006.png|420]]
>是否可以使用CDF图来体现DMA的优越性
>

