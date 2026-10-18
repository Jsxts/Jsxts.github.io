# Jsxts 的博客

个人博客，基于 **Hugo + PaperMod + GitHub Pages** 搭建。

- 线上地址：https://jsxts.github.io/
- 仓库：https://github.com/Jsxts/Jsxts.github.io
- 部署：推送到 `main` 后由 GitHub Actions 自动构建并发布

## 写一篇新文章

```bash
# 1. 新建文章（会自动生成带 front matter 的 md 文件）
hugo new content posts/文章名.md

# 2. 编辑 content/posts/文章名.md

# 3. 本地预览（-D 表示连草稿一起显示）
hugo server -D

# 4. 确认无误后，把 draft 改成 false，然后发布
git add -A
git commit -m "新增文章：xxx"
git push
```

推送后约 1~2 分钟，线上就会更新。

### 文章开头的参数

```yaml
---
title: "文章标题"
date: 2026-10-18T12:00:00+08:00   # 注意别写成未来时间，否则不会发布
draft: true                       # true = 草稿，不会被发布
author: "Jsxts"
description: "给搜索引擎和分享卡片看的描述"
summary: "列表页显示的摘要"
tags: ["标签1", "标签2"]
categories: ["分类"]
showToc: true                     # 是否显示右侧目录
---
```

> ⚠️ 两个最容易踩的坑：
> 1. `draft: true` 的文章不会发布，记得改成 `false`。
> 2. `date` 如果是**未来时间**，Hugo 默认也不发布。

## 常用命令

| 命令 | 作用 |
| --- | --- |
| `hugo server -D` | 本地预览，改文件自动刷新（http://localhost:1313） |
| `hugo new content posts/xxx.md` | 新建文章 |
| `hugo` | 本地构建，输出到 `public/` |
| `hugo --gc --minify` | 清理缓存并压缩，和线上构建一致 |

## 目录结构

```
.
├── archetypes/          # 新建文章的模板
├── content/
│   ├── posts/           # 文章都放这里
│   ├── about.md         # 关于页
│   ├── archives.md      # 归档页（layout: archives）
│   └── search.md        # 搜索页（layout: search）
├── layouts/             # 想改主题样式时在这里覆盖主题模板
├── static/              # 图标等静态文件，原样复制到网站根目录
├── themes/PaperMod/     # 主题（git submodule）
├── hugo.toml            # 站点主配置
└── .github/workflows/   # 自动部署配置
```

## 配置

几乎所有可调项都在 `hugo.toml` 里，注释已写明用途。常改的几处：

- `title` / `description` / `author` — 站点信息
- `[params.homeInfoParams]` — 首页欢迎语
- `[menu]` — 顶部导航栏
- `[params.socialIcons]` — 社交图标
- `[params] DateFormat` — 日期显示格式

## 背景图

全站背景图是 `assets/images/bg.jpg`，由 Hugo 自动转成 WebP 并加内容指纹。

**换图：直接替换 `assets/images/bg.jpg` 就行，不用改任何代码。**

相关文件只有两个，都可以删掉来还原成纯色背景：

| 文件 | 作用 |
| --- | --- |
| `assets/images/bg.jpg` | 背景图片本身 |
| `assets/css/extended/background.css` | 背景图 + 毛玻璃面板的全部样式 |
| `layouts/_partials/extend_head.html` | 把处理后的图片地址注入 `--bg-image` 变量 |

调参提示（都在 `background.css` 顶部）：

- `--panel`：内容面板的透明度，调大 = 内容更清晰、背景更淡
- `--entry`：文章列表卡片的透明度
- `.main` 的 `blur(20px)`：毛玻璃模糊强度
- `background-position`：改 `center center` 可调整图片裁剪位置

## 部署

`.github/workflows/hugo.yaml` 已配好：推送 `main` 分支 → 自动装 Hugo → 构建 → 发布到 GitHub Pages。

注意工作流里的 `submodules: recursive`，主题是以 git submodule 方式引入的，
**如果换电脑重新克隆仓库，必须带上子模块**：

```bash
git clone --recurse-submodules https://github.com/Jsxts/Jsxts.github.io.git
# 已经克隆过但 themes/PaperMod 是空的：
git submodule update --init --recursive
```

## 更新主题

```bash
git submodule update --remote --merge themes/PaperMod
```

## 其他

- 本仓库的 git 提交身份配置为 `jsxts <jsxts@users.noreply.github.com>`，
  如需修改：`git config user.email "你的邮箱"`
- 构建时可能出现 `.Language.LanguageDirection was deprecated` 之类的警告，
  这是主题模板尚未适配新版 Hugo 导致的，不影响使用。
