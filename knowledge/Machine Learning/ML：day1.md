## 1. python 的向量乘法和matlab对应关系

| 数学表达式     | Python语法             | Matlab 语法       |
| --------- | -------------------- | --------------- |
| 共轭内积      | np.vdot(w, x)        | w.' * x         |
| 不取共轭的内积   | np.dot(w, x)         | w .* x          |
| 逐元素乘法     | w * x                | w .* x          |
| 先逐元素乘，再求和 | np.sum(np.conj(w)*x) | sum(conj(w).*x) |
