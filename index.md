---
layout: page
title: CJLD军火库
tagline: 小小技术BLOG
---
{% include JB/setup %}
<div class="posts">
<h2>Summary</h2>
<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a><br>
    <div class="breadcrumb">{{ post.excerpt }}<a class="readMore" href="{{ BASE_PATH }}{{ post.url }}">read more &rArr;</a></div></li>
  {% endfor %}
</ul>
</div>
<div class="ToDoDiv">
<h2>To-Do List</h2>
  <div class="breadcrumb">
    {% for page in site.pages %}
      {% if page.group == "todo" %}
        {{ page.content | markdownify }}
      {% endif %}
    {% endfor %}
  </div>
</div>
