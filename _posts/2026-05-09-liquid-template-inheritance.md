---
layout: post
title: "Liquid 模板继承机制与博客架构设计"
date: 2026-05-09 10:00:00 +0800
author: 张三
categories: [技术, Jekyll]
tags: [Liquid, 模板引擎, 布局, 前端架构]
pinned: true
---

## 什么是模板继承

在开发博客时，每个页面都有重复的结构：HTML 文档声明、`<head>` 中的 SEO 标签、顶部导航栏、底部页脚。如果每个页面都复制粘贴这些内容，维护将是一场噩梦。**模板继承**的作用就是：定义一个基础骨架，子页面只填充变化的部分。

## Jekyll 的继承链

我们的博客采用了三级继承结构：

```
default.html（基础骨架）
    │
    ├── post.html（文章布局）
    │       └── 被 23 篇文章使用
    │       └── 包含：标题、目录、元信息、正文、标签、上下篇导航
    │
    └── page.html（页面布局）
            └── 被 about / categories 使用
            └── 包含：简单标题 + 内容
```

## 父模板：default.html

`default.html` 负责定义所有页面的公共结构：

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <title>{% raw %}{% if page.title %}{{ page.title }} - {{ site.title }}{% else %}{{ site.title }}{% endif %}{% endraw %}</title>
  {% raw %}{% seo %}{% endraw %}
  <link rel="stylesheet" href="{% raw %}{{ '/assets/css/custom.css' | relative_url }}{% endraw %}">
</head>
<body>
  <header class="site-header">...</header>
  <main class="page-content">
    {% raw %}{{ content }}{% endraw %}
  </main>
  <footer class="site-footer">...</footer>
</body>
</html>
```

关键语法：`{% raw %}{{ content }}{% endraw %}` 是一个占位符，子模板的内容会被注入到这个位置。

## 子模板：post.html

文章页面需要特殊的布局：左侧目录、标题元信息、标签、上下篇导航。它通过 Front Matter 声明继承关系：

```yaml
---
layout: default
---
```

然后在 `{% raw %}{{ content }}{% endraw %}` 的位置填充自己的结构：

```html
<article class="post">
  <header class="post-header">
    <h1 class="post-title">{% raw %}{{ page.title }}{% endraw %}</h1>
    <div class="post-meta">
      <span>{% raw %}{{ page.date | date: "%Y年%m月%d日" }}{% endraw %}</span>
      <span>{% raw %}{{ page.author }}{% endraw %}</span>
    </div>
  </header>

  <div class="post-body">
    <aside class="toc-sidebar">...</aside>
    <div class="post-content">
      {% raw %}{{ content }}{% endraw %}
    </div>
  </div>
</article>
```

注意这里的双重 `content`：
- 第一个 `content` 是 `post.html` 注入到 `default.html` 的整体内容
- 第二个 `content` 是 Markdown 文章的正文注入到 `post.html` 的位置

## 渲染流程

当 Jekyll 处理一篇 Markdown 文章时，完整的渲染流程如下：

```
┌─────────────────┐
│  Markdown 源文件 │  ← 包含 Front Matter 和正文
└────────┬────────┘
         │ 1. 解析 Front Matter，确定使用 post.html
         ▼
┌─────────────────┐
│   post.html     │  ← 将 Markdown 正文插入 {{ content }}
└────────┬────────┘
         │ 2. post.html 继承 default.html
         ▼
┌─────────────────┐
│  default.html   │  ← 将 post.html 的全部内容插入 {{ content }}
└────────┬────────┘
         │ 3. 生成最终 HTML
         ▼
┌─────────────────┐
│   输出文件       │  ← _site/ 目录中的 .html 文件
└─────────────────┘
```

## 实用技巧

### 条件判断

在模板中经常需要根据页面类型显示不同内容：

```liquid
{% raw %}{% if page.layout == 'post' %}
  <div class="post-nav">...</div>
{% endif %}{% endraw %}
```

### 循环遍历

分类页面需要遍历所有文章并按分类分组：

```liquid
{% raw %}{% for category in site.categories %}
  <h2>{{ category[0] }}</h2>
  {% for post in category[1] %}
    <a href="{{ post.url }}">{{ post.title }}</a>
  {% endfor %}
{% endfor %}{% endraw %}
```

### 过滤器

Liquid 提供丰富的过滤器处理数据：

| 过滤器 | 作用 | 示例 |
|--------|------|------|
| `date` | 格式化日期 | `{% raw %}{{ "2026-05-01" | date: "%Y年%m月%d日" }}{% endraw %}` |
| `relative_url` | 生成相对路径 | `{% raw %}{{ "/about/" | relative_url }}{% endraw %}` |
| `strip_html` | 去除 HTML 标签 | `{% raw %}{{ "<p>text</p>" | strip_html }}{% endraw %}` |
| `truncate` | 截断字符串 | `{% raw %}{{ "long text" | truncate: 10 }}{% endraw %}` |

## 总结

模板继承让博客的代码高度 DRY（Don't Repeat Yourself）。我们的博客虽然只有 3 个布局文件，却支撑了 20+ 个页面的渲染。理解这个机制后，新增页面类型（比如标签页、归档页）也变得非常容易——只需新建一个继承 `default.html` 的布局即可。
