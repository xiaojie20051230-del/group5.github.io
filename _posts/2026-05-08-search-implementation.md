---
layout: post
title: "静态博客的搜索功能实现原理"
date: 2026-05-08 14:00:00 +0800
author: 张三
categories: [技术, Jekyll]
tags: [搜索, Fuse.js, JavaScript, JSON]
pinned: true
---

## 问题背景

静态博客没有数据库，也没有后端服务器。当文章数量增加到 20+ 篇时，读者如何快速找到想要的内容？传统的方案是依赖分类和标签，但这需要读者对内容结构有一定了解。**搜索功能**成为提升用户体验的关键。

## 核心思路：预生成索引 + 客户端查询

静态博客实现搜索的核心公式可以概括为：

$$\text{搜索} = \text{预生成索引} + \text{前端加载} + \text{客户端模糊匹配}$$

这与传统动态网站的搜索架构形成鲜明对比：

| 维度 | 动态网站搜索 | 静态博客搜索 |
|------|------------|------------|
| 后端服务 | 需要 | 不需要 |
| 数据库 | MySQL/Elasticsearch | 无 |
| 实时性 | 实时查询 | 构建时生成索引 |
| 维护成本 | 高 | 极低 |
| 适用规模 | 大型站点 | 中小型博客 |

## 第一步：生成搜索索引

Jekyll 构建时，我们用 Liquid 模板遍历所有文章，生成一个 JSON 文件：

```json
[
  {
    "title": "Jekyll 环境搭建踩坑记录",
    "content": "Ruby 安装过程中遇到中文路径问题...",
    "url": "/jekyll-setup-notes/",
    "categories": ["技术", "Jekyll"],
    "tags": ["Jekyll", "Ruby"],
    "date": "2026-04-28"
  }
]
```

对应的 Liquid 代码逻辑：

```liquid
{% raw %}{% for post in site.posts %}
  {
    "title": {{ post.title | jsonify }},
    "content": {{ post.content | strip_html | truncate: 300 | jsonify }},
    "url": {{ post.url | relative_url | jsonify }},
    "categories": {{ post.categories | jsonify }},
    "tags": {{ post.tags | jsonify }}
  }
{% endfor %}{% endraw %}
```

关键点：
- `strip_html` 去除 Markdown 中的 HTML 标签，只保留纯文本
- `truncate: 300` 限制摘要长度，控制索引文件体积
- 索引文件大小直接影响首次加载速度

## 第二步：前端加载索引

页面加载后，通过 `fetch()` 异步获取索引：

```javascript
let fuse;
let searchIndex = [];

fetch('{{ "/search.json" | relative_url }}')
  .then(res => res.json())
  .then(data => {
    searchIndex = data;
    fuse = new Fuse(data, {
      keys: ['title', 'content', 'categories', 'tags'],
      threshold: 0.3,
      ignoreLocation: true
    });
  });
```

## 第三步：Fuse.js 模糊匹配

[Fuse.js](https://fusejs.io/) 是一个轻量级的模糊搜索库，支持以下特性：

- **容错搜索**：输入 "jekll" 也能匹配 "jekyll"
- **多字段权重**：标题匹配的优先级高于正文内容
- `threshold: 0.3`：值越小，匹配越严格；值越大，结果越宽松

搜索执行代码：

```javascript
function performSearch(query) {
  if (!query || query.length < 2) return [];
  return fuse.search(query).map(r => r.item);
}
```

## 交互设计细节

### 防抖处理

用户每输入一个字符就触发搜索会造成性能浪费。我们加入 200ms 防抖：

```javascript
let debounceTimer;
searchInput.addEventListener('input', (e) => {
  clearTimeout(debounceTimer);
  debounceTimer = setTimeout(() => {
    const results = performSearch(e.target.value);
    renderResults(results);
  }, 200);
});
```

### 键盘导航

支持上下箭头选择结果，Enter 键打开文章：

```javascript
document.addEventListener('keydown', (e) => {
  if (e.key === 'Escape') closeResults();
  if (e.key === 'ArrowDown') selectNext();
  if (e.key === 'ArrowUp') selectPrev();
});
```

### 移动端适配

搜索结果面板在小屏幕上容易溢出。我们的方案是：

```scss
.search-results {
  width: 320px;
  max-width: calc(100vw - 2rem);
}
```

## 性能数据

以本博客 23 篇文章为例：

| 指标 | 数值 |
|------|------|
| 索引文件大小 | ~12 KB（gzip 后 ~4 KB）|
| 首次加载时间 | ~50ms（本地网络）|
| 单次搜索耗时 | < 5ms |
| 内存占用 | 可忽略不计 |

## 总结

静态博客的搜索方案证明了：**没有后端，也能实现不错的搜索体验**。这种架构特别适合个人博客、文档站点等中小型内容场景。如果未来文章数量增长到数百篇，可以考虑：

1. 分页加载索引（按分类拆分多个 JSON 文件）
2. 接入 Algolia DocSearch 等专业搜索服务
3. 使用 Web Worker 在后台线程执行搜索，避免阻塞 UI
