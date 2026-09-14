---
title: "{{title}}"
authors: [{% for creator in creators %}"{{creator.lastName}} {{creator.firstName}}"{% if not loop.last %}, {% endif %}{% endfor %}]
year: {{date | format("YYYY")}}
citation_key: {{citationKey}}
status: "{% for t in tags %}{% if 'status/' in t.tag %}{{t.tag | replace('status/', '')}}{% endif %}{% endfor %}"
collections: [{% for c in collections %}"{{c.name}}"{% if not loop.last %}, {% endif %}{% endfor %}]
tags: [📝文献笔记, 🔖{{itemType}}{% for t in tags %}{% if not 'status/' in t.tag %}, "{{t.tag | replace(' ', '-')}}"{% endif %}{% endfor %}]
---

# PR：{{title}}

## 📚 元数据
- **作者**: {% for creator in creators %}{{creator.lastName}}{% if not loop.last %}, {% endif %}{% endfor %}
- **年份**: {{date | format("YYYY")}}
- **阅读状态**: {% for t in tags %}{% if 'status/' in t.tag %}{{t.tag | replace('status/', '')}}{% endif %}{% endfor %}
- **所属分类**: {% for c in collections %}📂{{c.name}}{% if not loop.last %}, {% endif %}{% endfor %}
- **期刊/出版社**: {{publicationTitle}}
- **Zotero 链接**: [在 Zotero 中打开文献](zotero://select/items/bbt:{{citationKey}})

## 📄 摘要
> {{abstractNote}}

---

## 📖 Zotero 独立笔记
{% for note in itemNotes %}
{% if note.noteTitle %}
**📌 {{ note.noteTitle }}**
{% endif %}
{{ note.note | safe }}
{% endfor %}

---

## 📝 阅读笔记与高亮

### 🔴 核心论点 (结论/主要创新点)
{% for annotation in annotations | filterby("color", "#ff6666") %}
{% if annotation.type == "image" %}
![](<{{ annotation.imagePath | replace('/Users/jiaqix/Documents/Obsidian Vault/', '') }}>)
{% else %}
> {{annotation.annotatedText}} (p. {{annotation.pageLabel}})
{% endif %}
{% if annotation.comment %}
- 💡 **批注**: {{annotation.comment}}
{% endif %}
{% endfor %}

### 🟡 重要细节 (研究方法/支撑数据)
{% for annotation in annotations | filterby("color", "#ffd400") %}
{% if annotation.type == "image" %}
![](<{{ annotation.imagePath | replace('/Users/jiaqix/Documents/Obsidian Vault/', '') }}>)
{% else %}
> {{annotation.annotatedText}} (p. {{annotation.pageLabel}})
{% endif %}
{% if annotation.comment %}
- 💡 **批注**: {{annotation.comment}}
{% endif %}
{% endfor %}

### 🔵 疑问与扩展 (待查阅/难以理解的概念)
{% for annotation in annotations | filterby("color", "#2ea8e5") %}
{% if annotation.type == "image" %}
![](<{{ annotation.imagePath | replace('/Users/jiaqix/Documents/Obsidian Vault/', '') }}>)
{% else %}
> {{annotation.annotatedText}} (p. {{annotation.pageLabel}})
{% endif %}
{% if annotation.comment %}
- 💡 **批注**: {{annotation.comment}}
{% endif %}
{% endfor %}

### 🟢 个人启发 (可迁移的方法/灵感)
{% for annotation in annotations | filterby("color", "#5fb236") %}
{% if annotation.type == "image" %}
![](<{{ annotation.imagePath | replace('/Users/jiaqix/Documents/Obsidian Vault/', '') }}>)
{% else %}
> {{annotation.annotatedText}} (p. {{annotation.pageLabel}})
{% endif %}
{% if annotation.comment %}
- 💡 **批注**: {{annotation.comment}}
{% endif %}
{% endfor %}