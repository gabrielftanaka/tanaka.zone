---
layout: page
title: writeups
permalink: /en/writeups/
lang: en
---

Machine writeups (HTB/THM), CTF challenges, bug bounty reports, practical tips, and certification notes.

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
