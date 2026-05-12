---
layout: post
title: "GitHub Actions 自动化部署实战笔记"
date: 2026-05-11 10:00:00 +0800
author: 张三
categories: [技术, DevOps]
tags: [GitHub Actions, CI/CD, 自动化, 部署]
pinned: true
---

## 为什么需要 CI/CD

手动部署的痛点显而易见：每次修改代码后，都要在本地执行 `jekyll build`，然后把生成的 `_site` 目录上传到服务器。这个过程**重复、容易出错、无法协作**。CI/CD（持续集成/持续部署）的理念是：**代码提交后，一切自动化**。

## GitHub Actions 简介

GitHub Actions 是 GitHub 提供的自动化工作流平台。它的核心概念是：

- **Workflow（工作流）**：定义在 `.github/workflows/*.yml` 中
- **Event（触发事件）**：push、pull_request、定时触发等
- **Job（任务）**：一组步骤的集合
- **Step（步骤）**：具体的命令或 action

## 我们的工作流配置

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
        if: github.ref == 'refs/heads/main'
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ '{{ secrets.GITHUB_TOKEN }}' }}
          publish_dir: ./_site
```

## 配置逐行解析

### 触发条件

```yaml
on:
  push:
    branches: [main]
```

只有当代码被推送到 `main` 分支时才触发构建。这样可以避免 feature 分支的半成品代码污染线上环境。

### 环境准备

```yaml
- uses: actions/checkout@v4
```

检出仓库代码到工作目录。`@v4` 表示使用第 4 版 action。

```yaml
- name: Setup Ruby
  uses: ruby/setup-ruby@v1
  with:
    ruby-version: '3.1'
    bundler-cache: true
```

安装指定版本的 Ruby，并自动缓存 gem 依赖。`bundler-cache: true` 是性能关键——第一次构建会下载所有依赖，后续构建直接从缓存读取，速度提升 5~10 倍。

### 构建阶段

```yaml
- name: Build site
  run: bundle exec jekyll build
  env:
    JEKYLL_ENV: production
```

`JEKYLL_ENV: production` 会启用生产环境优化。例如，`jekyll-seo-tag` 插件只有在 production 环境下才会生成搜索引擎需要的元数据。

### 部署阶段

```yaml
- name: Deploy to GitHub Pages
  if: github.ref == 'refs/heads/main'
  uses: peaceiris/actions-gh-pages@v3
  with:
    github_token: ${{ '{{ secrets.GITHUB_TOKEN }}' }}
    publish_dir: ./_site
```

这里有几个关键点：

1. `if: github.ref == 'refs/heads/main'`：双重保险，确保只在 main 分支部署
2. `secrets.GITHUB_TOKEN`：GitHub 自动提供的临时凭证，无需手动配置
3. `publish_dir: ./_site`：Jekyll 的默认输出目录

## 部署效果

配置完成后，每次 push 代码到 main 分支，Actions 会自动执行：

```
Setup Ruby → Install Dependencies → Build Site → Deploy to gh-pages
   (3s)         (5s/缓存后0s)         (8s)          (5s)
```

整个流程约 20 秒，比手动部署快得多。

## 常见问题

### 构建成功但页面 404

检查仓库 Settings → Pages 的 Source 是否设置为 **GitHub Actions**，而不是 Deploy from a branch。

### 时区问题导致文章不显示

Jekyll 默认使用 UTC 时间。如果文章设置 `date: 2026-05-11 10:00:00 +0800`，在 UTC 时间凌晨 2 点之前会被视为"未来文章"而跳过。解决方案：

1. 把日期设成过去时间
2. 或在 `_config.yml` 中添加 `future: true`

### 缓存失效

当修改 `Gemfile` 时，bundler 缓存可能过时。可以在 Actions 界面手动点击 "Re-run jobs" 强制刷新。

## 扩展思考

这套工作流是 CI/CD 的入门形态。更完善的方案可以加入：

- **PR 预览环境**：每个 Pull Request 自动生成预览链接
- **构建检查**：在合并前验证 Jekyll 能否成功构建
- **链接检查**：用 `htmlproofer` 检查死链
- **Lighthouse CI**：自动化性能评分

## 总结

GitHub Actions 让静态博客的部署变得**零成本、零维护**。它的本质是把"本地执行命令"搬到"云端执行命令"，但配合 Git 的版本控制和协作能力，整个开发体验提升了一个档次。对于我们这个课程项目来说，掌握这套流程的意义已经超出了作业本身——它是现代前端工程化的基本功。
