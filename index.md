---
layout: page
tagline: 
---
{% include JB/setup %}
    
## BLOG 

记录每一篇愿意记忆的片段

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>


