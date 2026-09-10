---
layout: page
title: research
permalink: /research/
---

<div class="ascii-wrapper">
<pre class="ascii-art">
██▀▀▀▀▀█ ██▀▀▀▀▀█ ██▀▀▀▀▀▀ ██▀▀▀▀▀█ ██▀▀▀▀▀█ ██▀▀▀▀▀█ ██▀▀▀▀▀█ ██    ██
▒█▀▀▀▀█▀ ▒█▀▀▀    ▀▀▀▀▀▀▀█ ▒█▀▀▀    ▒█▀▀▀ ▒█ ▒█▀▀▀▀█▀ ▒█       ▒█▀▀▀▀▀█
▓▓   ▀▀▓ ▓▓             ▓▓ ▓▓       ▓▓    ▓▓ ▓▓   ▀▀▓ ▓▓       ▓▓    ▓▓
█▒    █▒ █▒       █▒    █▒ █▒       █▒    █▒ █▒    █▒ █▒       █▒    █▒
▓░    ▓░ ▓░       ▓░    ▓░ ▓░       ▓░    ▓░ ▓░    ▓░ ▓░       ▓░    ▓░
▒█    ▒█ ▒█       ▒█    ▒█ ▒█       ▒█    ▒█ ▒█    ▒█ ▒█       ▒█    ▒█
░▓    ░▓ ░▓       ░▓    ░▓ ░▓       ░▓    ░▓ ░▓    ░▓ ░▓       ░▓    ░▓
 ▒     ▒  ▒    ▄▄  ▒     ▒  ▒    ▄▄  ▒     ▒  ▒     ▒  ▒    ▄▄  ▒     ▒
▀▀    ▀▀ ▀▀▀▀▀▀▀▀ ▀▀▀▀▀▀▀▀ ▀▀▀▀▀▀▀▀ ▀▀    ▀▀ ▀▀    ▀▀ ▀▀▀▀▀▀▀▀ ▀▀    ▀▀
</pre>
</div>

---

### **CVEs & Vulnerability Research:**

<ul style="list-style: none; padding-left: 0; margin: 0;">
{% assign pt_research = site.categories.research | where_exp: "item", "item.lang != 'en'" %}
{% for post in pt_research %}
  <li style="margin-bottom: 0.4rem;">
    <span>[ {{ post.date | date: "%Y-%m-%d" }} ]</span>
    <a href="{{ post.url }}">{{ post.title }}</a>
  </li>
{% else %}
  <li><p>Nenhuma pesquisa publicada ainda.</p></li>
{% endfor %}
</ul>
