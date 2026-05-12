---
layout: post
title: "GitHub Actions 自动化部署实践"
date: 2026-05-09 14:00:00 +0800
categories: [技术, DevOps]
tags: [CI/CD, GitHub Actions, 自动化]
author: 王五
---

## CI/CD 简介

**CI（Continuous Integration，持续集成）** 是指开发过程中不断将代码集成到主干，每次集成都会通过自动化测试验证。

**CD（Continuous Delivery，持续交付）** 是指在持续集成的基础上，将代码自动部署到生产环境。

## GitHub Actions 工作流

我们博客使用的 GitHub Actions 工作流包含以下步骤：

1. **检出代码** — 从 GitHub 仓库拉取最新代码
2. **设置 Ruby 环境** — 安装 Ruby 和 Bundler
3. **构建站点** — 运行 `jekyll build` 生成静态文件
4. **部署到 Pages** — 将生成的 `_site/` 目录部署到 GitHub Pages

## 工作流配置文件

工作流定义在 `.github/workflows/pages.yml` 文件中：

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

## 自动化部署的优势

- **一键发布** — 只需 `git push`，无需手动上传文件
- **版本可追溯** — 每次部署对应一次 Git 提交
- **协作友好** — 多人协作时，合并代码即自动部署
- **免费** — GitHub Actions 对公开仓库免费
