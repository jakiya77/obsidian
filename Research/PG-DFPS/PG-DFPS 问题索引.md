---
codex_dataview: true
type: project-index
project: FADLS / TVT论文
area: Research
category: Research
parent: 问题索引
status: active
priority: high
created: '2026-06-16'
updated: '2026-06-16'
summary: Canvas：obsidian note/Research/FADLS/FADLS 问题地图.canvas
tags:
- FADLS
- question-index
- Research
- project-index
- SINR
- MMSE
aliases:
- FADLS 问题索引
previous_project: FADLS
---
# FADLS 问题索引

## 入口
- Canvas：[[FADLS 问题地图.canvas|FADLS 问题地图]]
- 总览：[[obsidian note/Research/问题总览|问题总览]]
- 问题目录：`obsidian note/Research/Questions/FADLS/`

## 命名规则
- `FADLS-Q001：SINR-greedy 是否应该考虑候选池？`
- `FADLS-Q002：MMSE 输出 SINR 评分函数推导`
- `FADLS-Q003：Dual-lattice exact intersection 为什么稀疏`

## 当前未解决问题
```dataview
TABLE WITHOUT ID file.link AS "问题", id AS "ID", parent AS "父主题", status AS "状态", priority AS "优先级", created AS "创建时间"
WHERE type = "question" AND contains(file.folder, "Research/Questions/FADLS") AND status != "done"
SORT priority_rank DESC, created DESC
```

## 待办进度
```dataview
TASK
WHERE contains(file.folder, "Research/Questions/FADLS") AND !completed
GROUP BY file.link
```

## 已完成动作
```dataview
TASK
WHERE contains(file.folder, "Research/Questions/FADLS") AND completed
GROUP BY file.link
```

## 按父主题聚合
```dataview
TABLE rows.file.link AS "问题", rows.status AS "状态", rows.priority AS "优先级"
WHERE type = "question" AND contains(file.folder, "Research/Questions/FADLS")
GROUP BY parent
SORT parent ASC
```

## 已完成问题
```dataview
TABLE WITHOUT ID file.link AS "问题", id AS "ID", parent AS "父主题", created AS "创建时间"
WHERE type = "question" AND contains(file.folder, "Research/Questions/FADLS") AND status = "done"
SORT created DESC
```

## 固定主线
- 系统模型
- dual-lattice 物理动机
- 算法设计
- 仿真实验
- 论文写作
