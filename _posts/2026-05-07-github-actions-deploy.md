---
layout: post
title: "GitHub Actions 自动部署实战"
date: 2026-05-07 10:00:00 +0800
author: 张三
categories: [技术, DevOps]
tags: [GitHub Actions, CI/CD, GitHub Pages]
pinned: true
---

## 为什么用 GitHub Actions？

手动部署 Jekyll 站点的流程是：本地 `jekyll build` → 把 `_site` 文件夹上传到服务器。每次更新都要重复一遍，容易遗漏，也无法追溯。

GitHub Actions 帮我们把这个流程自动化：push 代码到 main 分支 → 自动构建 → 自动部署到 GitHub Pages。

## 工作流配置

在 `.github/workflows/pages.yml` 中定义：

```yaml
name: Build and Deploy Jekyll Site

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: '3.1'
          bundler-cache: true

      - name: Build site
        run: bundle exec jekyll build
        env:
          JEKYLL_ENV: production

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./_site
```

## 关键点解析

### 1. 触发条件

```yaml
on:
  push:
    branches: [main]
```

只在 `main` 分支有 push 时触发，避免 feature 分支的半成品被部署。

### 2. Ruby 环境

```yaml
- uses: ruby/setup-ruby@v1
  with:
    ruby-version: '3.1'
    bundler-cache: true
```

`bundler-cache: true` 会自动缓存 gem 依赖，下次构建时复用，速度提升明显。

### 3. 构建环境变量

```yaml
env:
  JEKYLL_ENV: production
```

设置 `JEKYLL_ENV=production` 后，Jekyll 会启用生产环境优化，比如压缩 CSS、禁用草稿等。

### 4. 部署权限

```yaml
with:
  github_token: ${{ secrets.GITHUB_TOKEN }}
```

`GITHUB_TOKEN` 是 Actions 自动提供的，无需手动创建 Personal Access Token，安全性更好。

## 常见问题

### _site 被 gitignore 了怎么办？

Jekyll 的 `_site` 目录通常被 `.gitignore` 排除，但 GitHub Actions 是在 runner 上全新构建的，不受本地 gitignore 影响。构建完成后直接推送到 `gh-pages` 分支即可。

### 构建失败怎么排查？

在 Actions 页面查看日志：

1. **依赖错误** → 检查 `Gemfile` 和 `Gemfile.lock` 是否提交到仓库
2. **Sass 警告** → 通常是主题使用了废弃的语法，不影响构建
3. **路径错误** → 检查 `baseurl` 和 `url` 配置

## 总结

CI/CD 的核心价值不是"自动化"本身，而是**标准化**和**可追溯**。每次部署都有日志记录，每次回滚都有 commit 对应，团队协作时不会因为"我本地能跑"而扯皮。
