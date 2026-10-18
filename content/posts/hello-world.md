---
title: "第一篇文章：博客搭建完成"
date: 2026-10-18T10:00:00+08:00
draft: false
author: "Jsxts"
description: "用 Hugo + PaperMod + GitHub Pages 搭建个人博客的记录。"
summary: "记录这次博客搭建的技术选型与部署方式。"
tags: ["Hugo", "GitHub Pages", "PaperMod"]
categories: ["博客搭建"]
showToc: true
TocOpen: false
hidemeta: false
comments: false
---

## 技术选型

这个博客由三部分组成：

| 部分 | 作用 |
| --- | --- |
| **Hugo** | 静态站点生成器，把 Markdown 编译成 HTML |
| **PaperMod** | 主题，负责页面外观 |
| **GitHub Pages** | 免费托管，通过 GitHub Actions 自动部署 |

## 关于写作

新增文章只需要在 `content/posts/` 下新建一个 `.md` 文件，然后 `git push`，
GitHub Actions 会自动构建并发布。

一个行内代码示例：`hugo new content posts/新文章.md`。

```bash
# 本地预览
hugo server -D

# 本地构建
hugo --gc --minify
```

> 提示：草稿文章设置 `draft: true` 后，本地用 `hugo server -D` 可以预览，
> 但不会发布到线上。

祝写作愉快 🎉
