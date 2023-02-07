---
layout: page
title: 归档
permalink: /archive/
---

<ul>
  {%- assign date_format = site.minima.date_format | default: "%Y-%m-%d" -%}
  {% for post in site.posts %}
    <li>
      <span class="post-meta">{{ post.date | date: date_format }}</span> <a href="{{ post.url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>