---
layout: post
title: "CSS Reset 与列表排版陷阱"
date: 2026-04-29 10:00:00 +0800
author: 张三
categories: [技术, 前端]
tags: [CSS, SCSS, 排版]
pinned: true
---

## 问题现象

文章详情页中，有序列表的序号（1., 2., 3., 4.）左侧边缘比上方正文段落的左边缘更靠左，视觉上序号"凸出"一截，排版参差不齐。

## 原因分析

我们的 CSS 中使用了全局重置：

```scss
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
```

这清除了浏览器默认的列表内边距。而 `<ol>` 默认的 `list-style-position: outside` 会将序号放在内容框外部，导致序号紧贴容器左边缘，与有内边距的正文段落不对齐。

用 Chrome DevTools 查看元素：

| 元素 | 默认 padding-left |
|------|-------------------|
| `<ol>` (浏览器默认) | 40px |
| `<ol>` (CSS Reset 后) | 0 |

## 解决方法

在 `assets/css/custom.scss` 的 `.post-content` 下添加：

```scss
ol, ul {
  padding-left: 2em;
  margin: 1rem 0;
}
```

给有序列表和无序列表统一增加左内边距，使序号整体右移，与正文段落左边缘对齐。

## 延伸思考

CSS Reset 是把双刃剑。它抹平了浏览器差异，但也需要你有意识地恢复必要的默认样式。常见的需要显式重置的元素包括：

- `ol, ul` — 列表缩进
- `blockquote` — 引用块边距和边框
- `figure` — 图片标题间距
- `hr` — 分隔线样式

理解每个浏览器默认样式存在的理由，才能用好 Reset。
