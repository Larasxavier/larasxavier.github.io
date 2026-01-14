---
layout: home
---

Bem-vinda(o) ao meu repositório de artigos técnicos.

{% for post in site.posts %}
- [{{ post.title }}]({{ post.url }}) — {{ post.date | date: "%d/%m/%Y" }}
{% endfor %}

