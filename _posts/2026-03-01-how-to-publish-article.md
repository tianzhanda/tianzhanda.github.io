---
title: "GitHub Pages + Jekyll 发布文章指南"
date: 2026-03-01
categories:
  - 技术
  - 教程
tags:
  - GitHub Pages
  - Jekyll
  - Git
---

本文介绍 GitHub Pages + Jekyll 博客如何发布文章，包括完整流程和常用命令。

## 发布流程

### 1. 创建文章文件

在 `_posts` 目录下创建 Markdown 文件，文件名格式：

```
年-月-日-标题.md
```

例如：

```bash
touch _posts/2026-03-01-my-first-post.md
```

### 2. 编写 Front Matter

文件开头添加 YAML 头信息：

```yaml
---
title: "文章标题"
date: 2026-03-01
categories:
  - 分类
tags:
  - 标签
---
```

### 3. 编写正文

使用 Markdown 语法编写内容。

### 4. 本地预览

```bash
bundle exec jekyll serve
```

访问 http://localhost:4000 查看效果。

### 5. 推送到 GitHub

```bash
git add .
git commit -m "Add: 文章标题"
git push origin main
```

## 常用命令

### Git

```bash
git status          # 查看状态
git add .           # 添加所有更改
git commit -m "信息" # 提交
git push origin main # 推送
git pull origin main # 拉取
```

### Jekyll

```bash
bundle exec jekyll serve    # 启动本地服务
bundle exec jekyll serve --drafts  # 预览草稿
bundle exec jekyll build    # 构建静态网站
```

## 常见问题

- **文章不显示**：检查文件名格式和 Front Matter 是否正确
- **构建失败**：查看 GitHub Actions 错误日志
- **404 错误**：首次部署需等待几分钟
