---
layout: page
title: research
permalink: /research/
---

Pesquisas de vulnerabilidades, análises e exploração de CVEs e artigos aprofundados de segurança.

---

### **CVEs & Vulnerability Research:**

<ul>
{% for post in site.categories.research %}
  <li>
    <span>[ {{ post.date | date: "%Y-%m-%d" }} ]</span>
    <a href="{{ post.url }}">{{ post.title }}</a>
  </li>
{% else %}
  <p>Nenhuma pesquisa publicada ainda.</p>
{% endfor %}
</ul>
