---
layout: page
title: HeLlo Xorlb!
tagline: "  Welcome to LXB's world"
---
{% include JB/setup %}

## Posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

## To-Do

Blog my `LIFE`. Blog your `COMMING`. Blog our `KNOWLEDGE`.
