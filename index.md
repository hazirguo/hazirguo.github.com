---
layout: page
title: Hello World!
tagline: Weclome to you~
---
{% include JB/setup %}

This is my all blog posts list:

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>


