---
codex_dataview: true
type: dashboard
project: Obsidian 知识库管理
area: Dataview Views
category: Dataview
parent: FADLS / TVT论文
status: active
priority: high
created: 2026-06-16
updated: 2026-06-16
tags:
  - Dataview
  - Obsidian
aliases: []
---
# FADLS / TVT论文视图

## 核心笔记

```dataview
TABLE type AS "类型", category AS "分类", parent AS "模块", status AS "状态", priority AS "优先级", created AS "创建", summary AS "主要内容"
FROM "obsidian note"
WHERE codex_dataview = true AND type != "dashboard" AND (project = "FADLS / TVT论文" OR contains(tags, "FADLS") OR contains(tags, "Movable-Antenna") OR contains(tags, "Anti-Jamming"))
SORT priority ASC, category ASC, updated DESC
```

## 研究想法与进行中

```dataview
TABLE category AS "分类", parent AS "模块", summary AS "主要内容", file.outlinks AS "关联", updated AS "更新"
FROM "obsidian note"
WHERE codex_dataview = true AND (project = "FADLS / TVT论文" OR contains(tags, "FADLS")) AND (type = "idea" OR type = "project" OR status = "active")
SORT updated DESC
```

## 算法 / SINR / MMSE

```dataview
TABLE type AS "类型", project AS "项目", category AS "分类", summary AS "主要内容", file.outlinks AS "关联"
FROM "obsidian note"
WHERE codex_dataview = true AND (contains(tags, "SINR") OR contains(tags, "MMSE") OR contains(tags, "FADLS") OR contains(tags, "Beamforming"))
SORT project ASC, updated DESC
```
