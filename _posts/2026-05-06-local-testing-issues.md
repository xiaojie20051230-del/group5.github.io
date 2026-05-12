---
layout: post
title: "本地测试中发现的问题与修复"
date: 2026-05-06 10:00:00 +0800
author: 张三
categories: [技术, 测试]
tags: [测试, 调试, 响应式]
pinned: true
---

## 测试策略

功能写完后不能马上交差，得测。我们的测试覆盖：

1. **页面可访问性** — 所有页面 200 OK
2. **链接有效性** — 上一篇/下一篇、分类锚点
3. **新功能验证** — TOC、代码复制、返回顶部
4. **响应式布局** — 1200px 断点、600px 移动端

## 问题一：目录跳转被导航栏遮挡

点击 TOC 目录项，页面平滑滚动到对应章节，但标题被 sticky 导航栏（`position: sticky; top: 0; height: ~64px`）盖住了。

**修复**：为标题添加 `scroll-margin-top`：

```scss
.post-content h2, .post-content h3 {
  scroll-margin-top: 80px;
}
```

浏览器在滚动到锚点时自动预留 80px 上边距，无需 JavaScript 计算偏移。

## 问题二：移动端无法复制代码

复制按钮默认 `opacity: 0`，只在 `:hover` 时显示。触屏设备没有 hover 事件，按钮永远不可见。

**修复**：移动端媒体查询强制显示：

```scss
@media (max-width: 600px) {
  .code-copy-btn { opacity: 1; }
}
```

## 问题三：Clipboard API 兼容性

`navigator.clipboard.writeText()` 在非 HTTPS 环境或旧浏览器中可能不存在。

**修复**：提供 `document.execCommand('copy')` 降级：

```js
if (navigator.clipboard) {
  navigator.clipboard.writeText(text).then(showCopied).catch(fallbackCopy);
} else {
  fallbackCopy(text);
}
```

## 问题四：Sass 编译生成小数 RGB

`darken($bg-color, 10%)` 编译后输出 `rgb(218.25, 223.5, 228.75)`。虽然浏览器能正确解析，但不够规范。

**评估**：测试确认所有目标浏览器渲染正常，暂不修复。如需可改为硬编码色值 `#d9e0e5`。

## 测试清单

| 测试项 | 结果 |
|--------|------|
| 全部 6 个页面正常访问 | 通过 |
| 上一篇/下一篇导航 | 通过 |
| 分类锚点跳转 | 通过 |
| 文章标题自动生成 id | 通过 |
| TOC 动态生成与滚动高亮 | 通过 |
| 返回顶部按钮显隐逻辑 | 通过 |
| 响应式布局断点 | 通过 |

## 收获

1. `scroll-margin-top` 是解决 sticky header 遮挡锚点的最优雅方案
2. 任何依赖 `:hover` 的功能都要考虑触摸设备
3. 现代 API 必须配降级方案
4. 自动化检查（链接有效性）比手动点更快更全
