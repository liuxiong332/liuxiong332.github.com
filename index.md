---
layout: page
title: 欢迎!
tagline: 雄的blog
---
{% include JB/setup %}


近期更新...

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
