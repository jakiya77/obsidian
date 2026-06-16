---
codex_dataview: true
type: question
project: FADLS / TVT论文
area: Research
category: Research
parent: 问题索引
status: active
priority: high
created: '2026-06-16'
updated: '2026-06-16'
summary: 1. 原始主问题
tags:
- FADLS
- SINR
- greedy
- MMSE
- candidate-pool
- Research
- question
aliases:
- FADLS-Q001：SINR-greedy 是否应该考虑候选池？
previous_project: FADLS
previous_parent: 算法设计
id: FADLS-Q001
priority_rank: 3
source: ChatGPT
---
# FADLS-Q001：SINR-greedy 是否应该考虑候选池？

## 1. 原始主问题
SINR-based greedy 算法是否应该基于全体 feasible 坐标逐步搜索，而不是沿用 FADLS 的 dual-lattice 候选池？

## 2. 派生子问题
- Q1：`Γ_MMSE(S)` 中的 `S` 到底表示天线/位置选择集合，还是候选池？
- Q2：SINR-greedy 的每一步评分是否应该遍历全体可选坐标？
- Q3：如果 SINR-greedy 使用 FADLS 候选池，会不会削弱高复杂度参考算法的公平性？
- Q4：论文里如何区分 FADLS 的低复杂度 candidate pool 和 SINR-greedy 的 full-grid search？

## 3. 已解决结论
- SINR-greedy 作为高复杂度参考，更自然的定义是在全体 feasible 坐标中逐步加入使 MMSE 输出 SINR 最大的坐标。
- FADLS 可以先构造 dual-favorable candidate pool，再在候选池中做低复杂度筛选。
- 两者的搜索空间和复杂度不同，需要在算法描述、复杂度分析和实验公平性说明中写清楚。

## 4. 尚未解决的问题
- 代码里是否已经确保 `sinr_greedy_select_fullgrid.m` 没有误用 FADLS candidate pool。
- 复杂度分析中 full-grid SINR-greedy 的代价表达式是否需要重新写。
- 实验图注和正文是否明确说明 SINR-greedy 是 full-grid baseline。

## 5. 下一步动作
- [x] 检查 MATLAB 代码中 SINR-greedy 的输入和搜索范围。
- [ ] 修改论文算法描述。
- [ ] 修改复杂度分析。
- [ ] 检查仿真实验说明和 figure caption。

## 6. 可作为新对话标题的问题
- SINR-greedy 算法复杂度推导
- FADLS 与 SINR-greedy 公平性比较
- MMSE 输出 SINR 评分函数推导

## 7. 检索关键词
`FADLS-SINR-greedy-候选池` `MMSE-SINR-推导` `SINR-greedy-fullgrid`

## 8. 相关链接
- [[obsidian note/Research/FADLS/FADLS 问题索引|FADLS 问题索引]]
