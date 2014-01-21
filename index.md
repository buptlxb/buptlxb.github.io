---
layout: page
title: HeLlo Xorlb!
tagline: LXB's hello world
---
{% include JB/setup %}

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

## To-Do

Blog my `LIFE`. Blog your `COMMING`. Blog our `KNOWLEDGE`.
