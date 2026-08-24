---
layout: page
title: writeups
permalink: /en/writeups/
lang: en
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
{% assign en_writeups = site.categories.writeup | where: "lang", "en" %}
{% for post in en_writeups %}
  <li style="margin-bottom: 0.4rem;">
    <span>[ {{ post.date | date: "%Y-%m-%d" }} ]</span>
    <a href="{{ post.url }}">{{ post.title }}</a>
  </li>
{% else %}
  <li><p>No writeups published yet.</p></li>
{% endfor %}
</ul>
