---
title: /
layout: default
permalink: /en/
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

welcome to my zone! here, i share some research and studies on cybersecurity. :)

{% assign en_posts = site.posts | where: "lang", "en" %}
{% assign latest_post = en_posts.first %}
{% if latest_post %}
**latest post: [{{ latest_post.title }}]({{ latest_post.url }})**

> {{ latest_post.excerpt | strip_html | strip_newlines}}

[> read more]({{ latest_post.url }})
{% endif %}

### **articles:**

<ul style="list-style: none; padding-left: 0; margin: 0;">
{% for post in site.posts %}
  {% if post.lang == "en" %}
  <li style="margin-bottom: 0.4rem;">
    [ {{ post.date | date: "%Y-%m-%d" }} ] <a href="{{ post.url }}">{{ post.title }}</a> {% if post.categories.size > 0 %}<span style="opacity: 0.5;">#{{ post.categories | join: " #" }}</span>{% endif %}
  </li>
  {% endif %}
{% endfor %}
</ul>
