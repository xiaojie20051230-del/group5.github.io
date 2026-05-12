---
layout: post
title: "响应式布局实践：从固定宽度到流式适配"
date: 2026-05-08 10:00:00 +0800
author: 张三
categories: [技术, 前端]
tags: [响应式, CSS, 媒体查询, 移动端]
pinned: true
---

## 引言

在搭建这个博客的过程中，我深刻体会到一个道理：**你的网站必须在任何设备上都能正常阅读**。桌面端用户可能用 27 寸显示器，而移动端用户可能只在 375px 宽的屏幕上浏览。响应式设计（Responsive Design）就是解决这个问题的核心方案。

## 核心概念

### 流式布局 vs 固定布局

固定布局使用像素（px）定义宽度，而流式布局使用百分比或弹性单位。在我们的博客中，`max-width` 配合 `margin: 0 auto` 是一种常见的居中策略：

```scss
$post-max-width: 800px;

.post {
  max-width: $post-max-width;
  margin: 0 auto;
  padding: 0 1rem;
}
```

当屏幕小于 800px 时，内容会自动收缩；大于 800px 时，内容居中显示，不会无限拉长导致阅读困难。

### 视口设置

移动设备默认会模拟桌面宽度渲染网页，导致文字过小。必须在 HTML 的 `<head>` 中加入：

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

这告诉浏览器：**按设备实际宽度渲染，初始缩放为 1 倍**。

## 媒体查询实战

### 导航栏的响应式改造

桌面端导航栏是水平排列的：

```scss
.main-nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
```

但在小屏幕上，水平空间不足，需要改为垂直堆叠：

```scss
@media (max-width: 600px) {
  .main-nav {
    flex-direction: column;
    gap: 1rem;
  }
}
```

### TOC 侧边栏的显隐控制

文章目录（Table of Contents）在桌面端作为侧边栏固定在右侧，但在移动端会挤占正文空间。我们的解决方案是：**小屏幕直接隐藏，大屏幕才显示**。

```scss
.toc-sidebar {
  display: none;
}

@media (min-width: 1200px) {
  .toc-sidebar {
    display: block;
    width: 220px;
    position: sticky;
    top: 80px;
  }
}
```

## 常见陷阱

### 图片溢出

未设置最大宽度的图片会在小屏幕上撑破容器：

```scss
.post-content img {
  max-width: 100%;
  height: auto;
}
```

### 触摸目标过小

按钮和链接在移动端的最小点击区域应为 44×44 像素。我们的返回顶部按钮就遵循了这个原则：

```scss
.back-to-top {
  width: 44px;
  height: 44px;
}
```

## 测试方法

1. **Chrome DevTools**：按 F12 → 切换设备工具栏 → 选择 iPhone SE / iPad / Desktop
2. **真实设备**：用手机访问局域网地址（Jekyll 启动时加上 `--host 0.0.0.0`）
3. **逐步缩放**：直接缩小浏览器窗口，观察断点（breakpoint）处的变化

## 总结

响应式设计不是"为手机单独做一个网站"，而是**让同一套代码自适应所有设备**。关键记住三点：

1. 设置正确的 viewport
2. 使用流式布局和弹性盒子
3. 用媒体查询在关键断点调整布局

本博客的响应式改造共涉及 3 个媒体查询断点（1200px、768px、600px），覆盖了从手机到桌面的全场景。
