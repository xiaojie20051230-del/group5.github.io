---
layout: default
title: 首页
---

## 最新文章

{% assign pinned_posts = site.posts | where: 'pinned', true %}
{% assign normal_posts = site.posts | where_exp: 'post', 'post.pinned != true' %}

{% for post in pinned_posts %}
<div class="post" style="margin-bottom: 2rem;">
  <header class="post-header">
    <h2 class="post-title">
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
      <span style="display:inline-block;background:#e74c3c;color:#fff;font-size:0.75rem;padding:0.15rem 0.5rem;border-radius:4px;margin-left:0.5rem;vertical-align:middle;">置顶</span>
    </h2>
    <div class="post-meta">
      <span class="post-date">{{ post.date | date: "%Y年%m月%d日" }}</span>
      {% if post.categories %}
        <span class="post-categories">
          {% for cat in post.categories %}
            <a href="{{ '/categories/#' | append: cat | relative_url }}">{{ cat }}</a>
            {% unless forloop.last %}, {% endunless %}
          {% endfor %}
        </span>
      {% endif %}
    </div>
  </header>
  <div class="post-content">
    {{ post.excerpt }}
    <a href="{{ post.url | relative_url }}">阅读全文 &rarr;</a>
  </div>
</div>
{% endfor %}

{% for post in normal_posts %}
<div class="post" style="margin-bottom: 2rem;">
  <header class="post-header">
    <h2 class="post-title">
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    </h2>
    <div class="post-meta">
      <span class="post-date">{{ post.date | date: "%Y年%m月%d日" }}</span>
      {% if post.categories %}
        <span class="post-categories">
          {% for cat in post.categories %}
            <a href="{{ '/categories/#' | append: cat | relative_url }}">{{ cat }}</a>
            {% unless forloop.last %}, {% endunless %}
          {% endfor %}
        </span>
      {% endif %}
    </div>
  </header>
  <div class="post-content">
    {{ post.excerpt }}
    <a href="{{ post.url | relative_url }}">阅读全文 &rarr;</a>
  </div>
</div>
{% endfor %}
