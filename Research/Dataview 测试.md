---
codex_dataview: true
type: dashboard
project: Obsidian 知识库管理
area: Research
category: Research
parent: Dataview 测试
status: active
priority: low
created: '2026-06-16'
updated: '2026-06-16'
summary: 1. 当前文件测试
tags:
- Research
- dashboard
- FADLS
aliases:
- Dataview 测试
previous_type: test
---
# Dataview 测试

## 1. 当前文件测试
如果 Dataview 正常启用，下面应该显示本文件。

```dataview
LIST
WHERE file.name = this.file.name
```
**
## 2. 问题笔记测试
如果查询路径正常，下面应该显示 `FADLS-Q001`。

```dataview
TABLE WITHOUT ID file.link AS "问题", id AS "ID", status AS "状态", priority AS "优先级"
WHERE type = "question" AND contains(file.folder, "Research/Questions")
SORT priority_rank DESC
```

## 3. 待办进度测试
如果任务查询正常，下面应该显示问题笔记里的未完成 checkbox。

```dataview
TASK
WHERE contains(file.folder, "Research/Questions") AND !completed
GROUP BY file.link
```
