---
title: "{{title}}"
authors: [{% for creator in creators %}"{{creator.lastName}} {{creator.firstName}}"{% if not loop.last %}, {% endif %}{% endfor %}]
year: {{date | format("YYYY")}}
citation_key: {{citationKey}}
tags: [📝文献笔记, 🔖{{itemType}}]
---

# PR：{{title}}

**📚 元数据**
- **作者**: {% for creator in creators %}[[{{creator.lastName}}]]{% if not loop.last %}, {% endif %}{% endfor %}
- **年份**: [[{{date | format("YYYY")}}]]
- **期刊/出版社**: {{publicationTitle}}
- **Zotero 链接**: [在 Zotero 中打开文献](zotero://select/library/items/{{itemKey}})

**📄 摘要**
> {{abstractNote}}

---

**📝 阅读笔记与高亮**

**🔴 核心论点 (结论/主要创新点)**
{% for annotation in annotations | filterby("color", "#ff6666") %}
> {{annotation.annotatedText}} [(p. {{annotation.pageLabel}})]({{annotation.backlink}})
{% if annotation.comment %}
- 💡 **批注**: {{annotation.comment}}
{% endif %}
{% endfor %}

**🟡 重要细节 (研究方法/支撑数据)**
{% for annotation in annotations | filterby("color", "#ffd400") %}
> {{annotation.annotatedText}} [(p. {{annotation.pageLabel}})]({{annotation.backlink}})
{% if annotation.comment %}
- 💡 **批注**: {{annotation.comment}}
{% endif %}
{% endfor %}

**🔵 疑问与扩展 (待查阅/难以理解的概念)**
{% for annotation in annotations | filterby("color", "#2ea8e5") %}
> {{annotation.annotatedText}} [(p. {{annotation.pageLabel}})]({{annotation.backlink}})
{% if annotation.comment %}
- 💡 **批注**: {{annotation.comment}}
{% endif %}
{% endfor %}

**🟢 个人启发 (可迁移的方法/灵感)**
{% for annotation in annotations | filterby("color", "#5fb236") %}
> {{annotation.annotatedText}} [(p. {{annotation.pageLabel}})]({{annotation.backlink}})
{% if annotation.comment %}
- 💡 **批注**: {{annotation.comment}}
{% endif %}
{% endfor %}