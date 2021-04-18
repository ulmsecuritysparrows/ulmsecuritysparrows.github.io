---
layout: page
title: USS
tagline: Ulm Security Sparrows
---

<div align="center">
<img src='assets/logo/header-uss.png' style="margin-right:10px" width='200'
height='198' alt='USS'>
<img src='assets/logo/header-uss_script.png' style="margin-top:144px;" width='490'
height='54' alt='Ulm Security Sparrows'>
</div>

## Posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
