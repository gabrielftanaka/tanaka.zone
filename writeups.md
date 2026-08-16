---
layout: page
title: writeups
permalink: /writeups/
---

Writeups de máquinas (HTB/THM), desafios de CTF, relatórios de bug bounty, dicas práticas e anotações de certificações.

---

### **Labs & Writeups:**

<ul style="list-style: none; padding-left: 0; margin: 0;">
{% assign pt_writeups = site.categories.writeup | where_exp: "item", "item.lang != 'en'" %}
{% for post in pt_writeups %}
  <li style="margin-bottom: 0.4rem;">
    <span>[ {{ post.date | date: "%Y-%m-%d" }} ]</span>
    <a href="{{ post.url }}">{{ post.title }}</a>
  </li>
{% else %}
  <li><p>Nenhum writeup publicado ainda.</p></li>
{% endfor %}
</ul>
