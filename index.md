---
title: tanaka
layout: default
permalink: /
---
<div class="ascii-wrapper">
<pre class="ascii-art">
██▀▀▀▀▀█ ██▀▀▀▀▀█ ██▀▀▀▀▀█ ██▀▀▀▀▀█ ██   ▄▄▀ ██▀▀▀▀▀█
      ▒█ ▒█▀▀▀ ▒█ ▒█    ▒█ ▒█▀▀▀ ▒█ ▒█▀▀▀▀▄  ▒█▀▀▀ ▒█
      ▓▓ ▓▓    ▓▓ ▓▓    ▓▓ ▓▓    ▓▓ ▓▓    ▓▓ ▓▓    ▓▓
      █▒ █▒    █▒ █▒    █▒ █▒    █▒ █▒    █▒ █▒    █▒
      ▓░ ▓░    ▓░ ▓░    ▓░ ▓░    ▓░ ▓░    ▓░ ▓░    ▓░
      ▒█ ▒█    ▒█ ▒█    ▒█ ▒█    ▒█ ▒█    ▒█ ▒█    ▒█
      ░▓ ░▓    ░▓ ░▓    ░▓ ░▓    ░▓ ░▓    ░▓ ░▓    ░▓
       ▒  ▒     ▒  ▒     ▒  ▒     ▒  ▒     ▒  ▒     ▒
      ▀▀ ▀▀    ▀▀ ▀▀    ▀▀ ▀▀    ▀▀ ▀▀    ▀▀ ▀▀    ▀▀
</pre>
</div>

# tanaka | @tanakz
offensive security researcher.

bem-vindo à minha zona! aqui compartilho algumas pesquisas e estudos em cybersecurity. :)

{% assign latest_post = site.posts.first %}
{% if latest_post %}
**último post: [{{ latest_post.title }}]({{ latest_post.url }})**

> {{ latest_post.excerpt | strip_html | strip_newlines}}

[> read more]({{ latest_post.url }})
{% endif %}

### **artigos:**

<ul style="list-style: none; padding-left: 0; margin: 0;">
{% for post in site.posts %}
  {% unless post.lang == "en" %}
  <li style="margin-bottom: 0.4rem;">
    [ {{ post.date | date: "%Y-%m-%d" }} ] <a href="{{ post.url }}">{{ post.title }}</a> {% if post.categories.size > 0 %}<span style="opacity: 0.5;">#{{ post.categories | join: " #" }}</span>{% endif %}
  </li>
  {% endunless %}
{% endfor %}
</ul>
