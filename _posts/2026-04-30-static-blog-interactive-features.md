---
layout: post
title: "为静态博客添加交互功能"
date: 2026-04-30 10:00:00 +0800
author: 张三
categories: [技术, 前端]
tags: [JavaScript, TOC, 用户体验]
pinned: true
---

## 为什么静态博客需要交互？

Jekyll 生成的是纯静态 HTML，没有后端服务器。但这不代表博客只能是"死"的页面。通过客户端 JavaScript，我们完全可以添加实用的交互功能，提升阅读体验。

这次我们为博客加了三个功能：文章目录（TOC）、代码复制按钮、返回顶部按钮。

## 文章目录：自动扫描标题生成

TOC 不需要手动维护。页面加载时，JavaScript 自动扫描文章中的 `h2` 和 `h3` 标题，动态生成目录：

```js
const headings = postContent.querySelectorAll('h2, h3');
headings.forEach(function (heading, index) {
  if (!heading.id) {
    heading.id = 'heading-' + index;
  }
  // 生成目录链接...
});
```

用 `IntersectionObserver` 实现滚动高亮当前章节，比监听 `scroll` 事件性能更好：

```js
const observer = new IntersectionObserver(function (entries) {
  entries.forEach(function (entry) {
    if (entry.isIntersecting) {
      // 高亮对应目录项
    }
  });
}, { rootMargin: '-10% 0px -80% 0px' });
```

响应式布局上，桌面端（>1200px）TOC 以 `position: sticky` 固定在左侧，小屏幕隐藏，不干扰阅读。

## 代码复制：减少用户操作步骤

技术博客免不了贴代码。让用户选中→复制，不如直接给按钮：

- 鼠标悬停代码块时显示复制按钮
- 点击后调用 `navigator.clipboard.writeText()`
- 提供 `document.execCommand('copy')` 降级方案，兼容旧浏览器
- 复制成功后按钮变绿，2 秒后恢复

## 返回顶部：看似 trivial，实则必要

长文阅读时，返回顶部是高频操作。实现很简单：

```js
window.addEventListener('scroll', function () {
  if (window.scrollY > 300) {
    backToTopBtn.classList.add('show');
  }
});
```

但细节决定体验：
- 滚动超过 300px 才显示，避免页面顶部时多余
- `transform` 动画比直接改 `display` 更流畅
- 点击后 `scrollTo({ behavior: 'smooth' })` 平滑回顶

## 总结

静态博客的交互不靠后端，全靠前端三板斧：

1. **DOM 操作** — 动态生成内容
2. **事件监听** — 响应用户操作
3. **CSS 动画** — 提供视觉反馈

这些功能零依赖、零配置，部署到 GitHub Pages 直接可用。
