---
layout: page
title: CJLD军火库
tagline: 小小技术BLOG
---
{% include JB/setup %}

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a><br>
    <div class="breadcrumb">{{ post.excerpt }}<a class="readMore" href="{{ BASE_PATH }}{{ post.url }}">read more &rArr;</a></div></li>
  {% endfor %}
</ul>
