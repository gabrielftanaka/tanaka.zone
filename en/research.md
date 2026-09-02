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

<ul style="list-style: none; padding-left: 0; margin: 0;">
{% assign en_research = site.categories.research | where: "lang", "en" %}
{% for post in en_research %}
  <li style="margin-bottom: 0.4rem;">
    <span>[ {{ post.date | date: "%Y-%m-%d" }} ]</span>
    <a href="{{ post.url }}">{{ post.title }}</a>
  </li>
{% else %}
  <li><p>No writeups published yet.</p></li>
{% endfor %}
</ul>
