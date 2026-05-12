---
layout: page
title: 文章分类
permalink: /categories/
---

{% for category in site.categories %}
<section class="category-section">
  <h2 id="{{ category[0] }}">
    {{ category[0] }}
    <span class="count">({{ category[1].size }} 篇)</span>
  </h2>
  <ul class="post-list">
    {% for post in category[1] %}
      <li>
        <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
        <time>{{ post.date | date: "%Y-%m-%d" }}</time>
      </li>
    {% endfor %}
  </ul>
</section>
{% endfor %}
