---
layout: page
title: Life of The Wanderer
tagline: Supporting tagline
---
{% include JB/setup %}
    
## Posts History

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

## I am passionate about

  <ul>
    <li>Free Software and Open Source</li>
    <li>programming</li>
    <li>technology</li>
    <li>travelling</li>
    <li>cycling</li>
    <li>photography</li>
    <li>farming</li>
    <li>investments</li>
  </ul>