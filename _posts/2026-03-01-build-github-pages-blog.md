---
title: "从零搭建 GitHub Pages 博客：Minimal Mistakes 主题实战"
date: 2026-03-01
categories:
  - 技术
  - 教程
tags:
  - GitHub Pages
  - Jekyll
  - 博客
---

本文记录了使用 GitHub Pages + Jekyll 搭建个人博客的完整过程，包括主题选择、配置步骤、遇到的问题及解决方案。

## 什么是 GitHub Pages？

GitHub 官方提供的免费静态网站托管服务，具有以下优势：

- **完全免费** - 托管、域名、HTTPS 都免费
- **无需服务器** - 代码推送到 GitHub 自动部署
- **版本控制** - 所有内容都在 Git 里，随时回滚
- **专注写作** - 只需写 Markdown，不用管前端代码

## 为什么选择 Minimal Mistakes 主题？

Minimal Mistakes 是 GitHub 上最受欢迎的 Jekyll 主题之一，特点包括：

- 响应式设计，适配各种设备
- 多种颜色皮肤可选（支持黑色主题）
- 丰富的配置选项
- 完善的文档支持

## 搭建步骤

### 1. 创建仓库

登录 GitHub，创建一个名为 `用户名.github.io` 的仓库（用户名替换为你的 GitHub 用户名）。

### 2. 配置文件

创建 `_config.yml` 配置文件：

```yaml
title: "你的博客名称"
author: "你的用户名"
description: "博客描述"

remote_theme: "mmistakes/minimal-mistakes@4.27.0"
minimal_mistakes_skin: "dark"

markdown: kramdown
paginate: 5

plugins:
  - jekyll-paginate
  - jekyll-sitemap
  - jekyll-include-cache

defaults:
  - scope:
      path: "_posts"
    values:
      layout: single
      author_profile: true
```

### 3. 创建首页

创建 `index.html`：

```html
---
layout: home
title: "首页"
---
```

### 4. 创建关于页面

在 `_pages` 目录下创建 `about.md`：

```yaml
---
layout: single
title: "关于"
permalink: /about/
---

你好！欢迎来到我的博客。
```

### 5. 写文章

在 `_posts` 目录下创建 Markdown 文件，文件名格式：`年-月-日-标题.md`

```yaml
---
title: "文章标题"
date: 2026-03-01
categories:
  - 分类
---

正文内容...
```

### 6. 推送到 GitHub

```bash
git add .
git commit -m "Initial blog"
git push -u origin main
```

## 遇到的问题及解决方案

### 问题 1：首次部署显示 404

**原因**：GitHub Pages 首次构建需要较长时间（1-10分钟）

**解决**：等待几分钟后刷新页面

### 问题 2：Jekyll 构建失败

**原因**：早期配置使用了不兼容的插件组合

**解决**：使用 Minimal Mistakes 官方推荐的配置，添加 `jekyll-include-cache` 插件

### 问题 3：主题未生效

**原因**：`remote_theme` 版本配置不完整

**解决**：使用完整版本号，如 `mmistakes/minimal-mistakes@4.27.0`

## 如何切换颜色主题

Minimal Mistakes 提供多种皮肤，在 `_config.yml` 中添加：

```yaml
minimal_mistakes_skin: "dark"  # 可选：default, air, aqua, contrast, dark, dirt, neon, mint, plum
```

## 博客地址

搭建完成后，访问 `https://你的用户名.github.io` 即可查看博客。

## 总结

使用 GitHub Pages + Jekyll 搭建博客是一个性价比极高的方案：

- 零成本托管
- 零维护负担
- 专注于内容创作
- 享受 Git 版本控制的便利

希望本文对你有所帮助，祝你博客之旅愉快！
