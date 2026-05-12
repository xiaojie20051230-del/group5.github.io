---
layout: post
title: "Jekyll 环境搭建踩坑记录"
date: 2026-04-28 10:00:00 +0800
author: 张三
categories: [技术, Jekyll]
tags: [Jekyll, Ruby, 环境配置]
pinned: true
---

## 引言

搭建 Jekyll 博客看起来很简单，跟着官方文档走就行。但在 Windows 环境下，特别是中文用户，坑还是不少的。这篇记录一下我们小组在环境搭建阶段踩过的坑，给后来的同学做个参考。

## Ruby 安装：路径里的中文是万恶之源

我们一开始把 Ruby 装在了 `E:\临时\创新实验\jekyll\Ruby31-x64` 路径下，结果 `gem install jekyll bundler` 时直接报错：

```
linking shared-object json/ext/parser.so
C:/.../msys2/ld.exe: cannot find ���/...: No such file or directory
collect2.exe: error: ld returned 1 exit status
```

注意看，报错信息里全是乱码（`���`）。这是因为 MSYS2 的链接器不支持中文路径，遇到"临时"、"创新实验"这些汉字就懵掉了。

**解决方法**：把 Ruby 移到纯英文路径，比如 `D:\dev\Ruby31-x64`，问题立刻解决。

## Gem 安装慢？换源！

默认的 RubyGems 源在国内访问很慢，经常超时。换成清华或 Ruby China 的镜像源：

```bash
gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/
bundle config mirror.https://rubygems.org https://gems.ruby-china.com
```

## Jekyll  serve 启动失败

安装完后运行 `bundle exec jekyll serve`，可能会遇到依赖缺失的错误。确保在博客目录下执行：

```bash
cd my-blog
bundle install
bundle exec jekyll serve
```

`bundle install` 会根据 Gemfile 安装项目所需的精确版本依赖，避免版本冲突。

## 总结

Windows 上搭 Jekyll 环境，记住三点：

1. **路径用英文**，别让中文坑了你
2. **gem 换国内源**，速度提升十倍
3. **用 bundle 管理依赖**，别全局乱装 gem

环境搭好后，后面的静态页面生成、本地预览都是顺滑的。下一篇会记录主题定制阶段遇到的 CSS 问题。
