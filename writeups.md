---
layout: page
title: writeups
permalink: /writeups/
---

<div class="ascii-wrapper">
<pre class="ascii-art">
██ ██ ██ ██▀▀▀▀▀█ ▀▀▀▀█▀▀▀ ██▀▀▀▀▀█ ██▀▀▀▀▀█ ██    ██ ██▀▀▀▀▀█ ██▀▀▀▀▀▀
▒█ ▒█ ▒█ ▒█▀▀▀▀█▀    ▒█          ▒█ ▒█▀▀▀    ▒█    ▒█ ▒█▀▀▀▀▀▀ ▀▀▀▀▀▀▀█
▓▓ ▓▓ ▓▓ ▓▓   ▀▀▓    ▓▓          ▓▓ ▓▓       ▓▓    ▓▓ ▓▓             ▓▓
█▒ █▒ █▒ █▒    █▒    █▒          █▒ █▒       █▒    █▒ █▒       █▒    █▒
▓░ ▓░ ▓░ ▓░    ▓░    ▓░          ▓░ ▓░       ▓░    ▓░ ▓░       ▓░    ▓░
▒█ ▒█ ▒█ ▒█    ▒█    ▒█          ▒█ ▒█       ▒█    ▒█ ▒█       ▒█    ▒█
░▓ ░▓ ░▓ ░▓    ░▓    ░▓          ░▓ ░▓       ░▓    ░▓ ░▓       ░▓    ░▓
 ▒  ▒  ▒  ▒     ▒     ▒           ▒  ▒    ▄▄  ▒     ▒  ▒        ▒     ▒
▀▀▀▀▀▀▀▀ ▀▀    ▀▀ ▀▀▀▀▀▀▀▀       ▀▀ ▀▀▀▀▀▀▀▀ ▀▀▀▀▀▀▀▀ ▀▀       ▀▀▀▀▀▀▀▀
</pre>
</div>

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
