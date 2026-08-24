---
layout: page
title: research
permalink: /en/research/
lang: en
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

<ul>
{% for post in site.categories.research %}
  <li>
    <span>[ {{ post.date | date: "%Y-%m-%d" }} ]</span>
    <a href="{{ post.url }}">{{ post.title }}</a>
  </li>
{% else %}
  <p>No research published yet.</p>
{% endfor %}
</ul>
