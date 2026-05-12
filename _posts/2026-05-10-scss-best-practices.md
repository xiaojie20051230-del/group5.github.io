---
layout: post
title: "SCSS 实践：变量、嵌套与代码复用"
date: 2026-05-10 10:00:00 +0800
author: 张三
categories: [技术, 前端]
tags: [SCSS, CSS, 预处理器, 样式架构]
pinned: true
---

## 为什么用 SCSS

原生 CSS 没有变量、没有嵌套、没有复用机制。当项目规模扩大时，维护成百上千行的 CSS 文件会变得非常困难。SCSS（Sassy CSS）是 CSS 的超集，它在编译后生成纯 CSS，但开发体验提升了一个数量级。

## 变量系统

### 定义颜色主题

我们的博客采用了一套统一的颜色变量：

```scss
$primary-color: #2c3e50;   // 深蓝灰：导航栏背景
$accent-color:  #3498db;   // 亮蓝色：链接、按钮、高亮
$text-color:    #333;      // 正文文字
$bg-color:      #f8f9fa;   // 页面背景
$max-width:     800px;     // 内容最大宽度
```

这些变量的好处是：**改一处，全局生效**。如果老师要求调整配色，只需修改几个变量值，不用在几十处颜色声明中逐一替换。

### 语义化命名

避免使用 `$blue`、`$red` 这种描述外观的命名，应该描述**用途**：

```scss
// 不推荐
$blue: #3498db;
background: $blue;

// 推荐
$accent-color: #3498db;
background: $accent-color;
```

这样当品牌色从蓝色改为绿色时，变量名依然有意义。

## 嵌套语法

### 选择器嵌套

原生 CSS 中， descendant 选择器需要重复写父级类名：

```css
.site-header { background: #2c3e50; }
.site-header .main-nav { max-width: 800px; }
.site-header .main-nav .logo { color: white; }
```

SCSS 允许按 HTML 结构嵌套：

```scss
.site-header {
  background: $primary-color;

  .main-nav {
    max-width: $max-width;
    display: flex;
    justify-content: space-between;

    .logo {
      color: white;
      font-size: 1.5rem;
    }
  }
}
```

编译后生成的 CSS 与上面完全一致，但源代码的结构清晰得多。

### 父选择器引用

`&` 符号代表当前选择器的父级，常用于伪类和状态修饰：

```scss
.nav-links a {
  color: rgba(255, 255, 255, 0.9);
  transition: color 0.3s;

  &:hover {
    color: $accent-color;
  }

  &.active {
    font-weight: bold;
    border-bottom: 2px solid $accent-color;
  }
}
```

编译结果：

```css
.nav-links a { color: rgba(255, 255, 255, 0.9); }
.nav-links a:hover { color: #3498db; }
.nav-links a.active { font-weight: bold; border-bottom: 2px solid #3498db; }
```

## 代码复用

### Mixins

当一段样式需要在多个地方使用时，可以定义成 Mixin：

```scss
@mixin flex-center {
  display: flex;
  justify-content: center;
  align-items: center;
}

.back-to-top {
  @include flex-center;
  width: 44px;
  height: 44px;
  border-radius: 50%;
}
```

### 运算

SCSS 支持数学运算，常用于尺寸计算：

```scss
$base-spacing: 1rem;

.post {
  padding: $base-spacing;
  margin-bottom: $base-spacing * 2;  // 2rem
}
```

## 模块化组织

我们的 `custom.scss` 虽然把所有样式写在了一个文件里，但逻辑上可以分为几个模块：

```scss
// 1. 变量定义
$primary-color: #2c3e50;
// ...

// 2. 基础重置
* { margin: 0; padding: 0; }

// 3. 布局组件
.site-header { ... }
.post { ... }

// 4. 功能组件
.code-copy-btn { ... }
.search-box { ... }

// 5. 响应式断点
@media (max-width: 600px) { ... }
```

对于更大的项目，应该把模块拆分成独立的文件，用 `@import` 组合：

```scss
@import 'variables';
@import 'base';
@import 'components/header';
@import 'components/post';
@import 'components/search';
```

## 总结

SCSS 不是魔法，它的核心价值在于：

1. **变量**：统一管理设计 token（颜色、间距、字体）
2. **嵌套**：按 DOM 结构组织选择器，提高可读性
3. **复用**：Mixins 和函数减少重复代码

本博客的全部样式只用一个文件、约 500 行 SCSS 就实现了完整的自定义主题，这得益于 SCSS 提供的高效抽象能力。
