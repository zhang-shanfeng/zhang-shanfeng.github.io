---
title: 'My Hugo Getting Started'
date: '2026-08-11T15:50:23+08:00'
draft: false
lastmod: '2026-08-11T15:50:23+08:00'

categories: [notes]
tags: [Hugo]

cover:
  image: "/posts/cover.jpg"
  alt: "cover"
  caption: "cover"

# 作者
author: "zhang-shanfeng"

showtoc: true
comments: true

keywords: ['Hugo']
---

`Hugo` 博客搭建完成了，特此写一篇文章记录下搭建过程和相关配置等。

- `Hugo` 官网：[https://gohugo.io/](https://gohugo.io/ "Hugo 官网")
- 官网快速入门教程：[https://gohugo.io/getting-started/quick-start/](https://gohugo.io/getting-started/quick-start/ "官网快速入门教程")

## 下载安装

我的电脑是 Windows 系统，选择了直接下载 Hugo 二进制文件的方式，下载后把二进制文件的目录添加到系统环境变量即可。下载地址：[https://github.com/gohugoio/hugo/releases](https://github.com/gohugoio/hugo/releases "Hugo releases 页面") 选择最新的版本下载即可。

## 安装主题

我选择了 `PaperMod` 主题，安装方法主题官网也有，我把官网的粘贴到下面：

```bash
hugo new site MyFreshWebsite --format yaml # 首先创建一个站点，并且指定配置文件的格式为 yaml
```

官方提供了好几种安装方法，这里我只粘贴了 Git Clone 的方法：

```bash
git clone https://github.com/adityatelange/hugo-PaperMod themes/PaperMod --depth=1
cd themes/PaperMod
git pull # 升级主题
```

In `hugo.yaml` add:

```yaml
theme: ["PaperMod"]
```

## 我的主题配置

```yaml
baseURL: https://zhang-shanfeng.github.io
paginate: 10
title: zhang-shanfeng Blog
theme: PaperMod

enableRobotsTXT: true
buildDrafts: false
buildFuture: false
buildExpired: false

minify:
  disableXML: true
  minifyOutput: true

# 必须配置，否则不会生成搜索索引
outputs:
  home:
    - HTML
    - RSS
    - JSON

params:
  title: zhang-shanfeng Blog
  description: "这是一个个人博客网站，核心关注领域：电子技术与编程开发，另外该网站也是我学习工作中用作记录笔记的地方，欢迎大家访问我的网站。"
  keywords: [Blog, ESP32, PCB, Tauri, Vue, FreeRTOS, Linux, 电子仪器, 数据采集电路, 数据生成, 数据分析, 工具软件, sfeekit]
  author: zhang-shanfeng
  images: ./assets/icons/128x128.png
  DateFormat: "January 2, 2006"
  defaultTheme: auto # dark, light
  disableThemeToggle: false

  ShowReadingTime: true
  ShowShareButtons: true
  ShowPostNavLinks: true
  ShowBreadCrumbs: true
  ShowCodeCopyButtons: true
  ShowWordCount: true
  ShowRssButtonInSectionTermList: true
  UseHugoToc: true
  disableSpecial1stPost: false
  disableScrollToTop: false
  comments: false
  hidemeta: false
  hideSummary: false
  showtoc: true
  tocopen: false


  assets:
    favicon: "/icons/icon.ico"
    favicon32x32: "/icons/32x32.png"

  label:
    text: "zhang-shanfeng Blog"
    icon: /icons/64x64.png
    iconHeight: 35

  # profile-mode
  profileMode:
    enabled: false # needs to be explicitly set
    title: ExampleSite
    subtitle: "This is subtitle"
    imageUrl: "/icons/128x128.png"
    imageWidth: 120
    imageHeight: 120
    imageTitle: my image
    buttons:
      - name: Posts
        url: posts
      - name: Tags
        url: tags

  # home-info mode
  homeInfoParams:
    Title: "Hi there \U0001F44B"
    Content: Welcome to my blog

  socialIcons:
    - name: "github"
      url: "https://github.com/zhang-shanfeng"
    - name: "bilibili"
      url: "https://space.bilibili.com/334715750"

  # for search
  # https://fusejs.io/api/options.html
  fuseOpts:
    isCaseSensitive: false
    shouldSort: true
    location: 0
    distance: 1000
    threshold: 0.4
    minMatchCharLength: 0
    limit: 10 # refer: https://www.fusejs.io/api/methods.html#search
    keys: ["title", "permalink", "summary", "content"]

  cover:
      hidden: false # hide everywhere but not in structured data
      hiddenInList: true # hide on list pages and home
      hiddenInSingle: false # hide on single page

menu:
  main:
    - identifier: about
      name: About
      pageRef: /about
      weight: 30
    - identifier: tags
      name: Tags
      url: "/tags/"
      weight: 20
    - identifier: categories
      name: Categories
      url: "/categories/"
      weight: 10
    - identifier: archives
      name: Archive
      url: "/archives/"
      weight: 10
    - identifier: search
      name: Search
      url: "/search/"
      weight: 10

```

## 目录介绍

- archetypes: 原型模板目录，里面可以创建多个模板文件，用 hugo new 创建文章等使用的模板，默认该目录下有一个 default.md 文件
- assets: 会被 Hugo 构建管道处理，用途：自定义 CSS、JavaScript、需要优化的图片
- content 放文章、分类、页面的位置
- data
- i18n
- layouts
- public
- static: 静态目录
- themes: 主题目录

## 自定义 CSS

在 assets/css/extended/ 目录下创建 CSS 文件，Hugo 构建时会自动将其与主题样式合并、压缩成一个文件：

```bash
assets/
└── css/
    └── extended/
        └── custom.css    ← 你写的自定义样式
```

```css
/* assets/css/extended/custom.css */
.post-content {
    font-size: 17px;
}
```

## 文章图片

`PaperMod` 支持将图片放在 `assets` 目录下：

```bash
assets/
└── images/
    └── posts/
        └── cover.jpg
```

在文章中使用：

```md
---
title: 'My Hugo Getting Started'
date: '2026-08-11T15:50:23+08:00'

cover:
  image: "/posts/cover.jpg"
  alt: "cover"
  caption: "cover"
---
```

## 创建分类页

在 `content` 下创建 `categories` 目录，并在该目录下创建 `_index.md` 文件：

```md
---
title: Categories
layout: categories
---

```

添加菜单链接：

```yaml
menu:
  main:
    - identifier: categories
      name: Categories
      url: "/categories/"
      weight: 10
```

## 创建标签页

在 `content` 下创建 `tags` 目录，并在该目录下创建 `_index.md` 文件：

```md
---
title: Tags
layout: tags
---

```

添加菜单链接：

```yaml
menu:
  main:
    - identifier: tags
      name: Tags
      url: "/tags/"
      weight: 20
```

## 创建关于页

在 `content` 下创建 `about` 目录，并在该目录下创建 `index.md` 文件：

```md
---
date: '2026-08-11T13:13:11+08:00'
draft: true
title: 'About'
url: /about/
noMeta: true
description: "核心关注领域：电子技术与编程开发。"
---

### 🛠 技术栈

- Rust | C | TypeScript | Python
- Tauri | Vue | Docker
- vsCode | Zed
- 模电电路 | 嵌入式
- 千问

### 📬 联系方式

- GitHub: [zhang-shanfeng](https://github.com/zhang-shanfeng)
- Email: shanfeng_zhang[@]icloud.com

```

添加菜单链接：

```yaml
menu:
  main:
    - identifier: about
      name: About
      pageRef: /about
      weight: 30
```

## 创建搜索页

在 `content` 下创建 `search.md` 文件：

```md
---
title: "Search"
layout: "search"
summary: "search"
---

```

添加菜单链接：

```yaml
menu:
  main:
    - identifier: search
      name: Search
      url: "/search/"
      weight: 10
```

## 创建归档页

在 `content` 下创建 `archives.md` 文件：

```md
---
title: "Archive"
layout: "archives"
url: "/archives/"
summary: archives
---

```

添加菜单链接：

```yaml
menu:
  main:
    - identifier: archives
      name: Archive
      url: "/archives/"
      weight: 10

```

## 配置数学公式

### hugo.yaml

```yaml
markup:
  goldmark:
    extensions:
      passthrough:
        enable: true
        delimiters:
          block:
            - - '\['
              - '\]'
            - - '$$'
              - '$$'
          inline:
            - - '\('
              - '\)'
            - - '$'
              - '$'

```

在 `layouts` 下创建 `partials` 目录，在 `partials` 目录下创建 `extend_head.html` 文件：

```html
<link
    rel="stylesheet"
    href="https://cdn.jsdelivr.net/npm/katex@0.16.22/dist/katex.min.css" />

<script
    defer
    src="https://cdn.jsdelivr.net/npm/katex@0.16.22/dist/katex.min.js"></script>

<script
    defer
    src="https://cdn.jsdelivr.net/npm/katex@0.16.22/dist/contrib/auto-render.min.js"></script>

<script>
    document.addEventListener("DOMContentLoaded", function () {
        renderMathInElement(document.body, {
            delimiters: [
                { left: "\\[", right: "\\]", display: true },
                { left: "$$", right: "$$", display: true },
                { left: "\\(", right: "\\)", display: false },
                { left: "$", right: "$", display: false },
            ],

            throwOnError: false,
        });
    });
</script>

```

这样就明确告诉 KaTeX:

```
\[ ... \]    → 块公式
$$ ... $$    → 块公式

\( ... \)    → 行内公式
$ ... $      → 行内公式
```

完整配置成：

```
Hugo Goldmark Passthrough
        ↓
保留 LaTeX 数学公式
        ↓
KaTeX
        ↓
浏览器渲染
        ↓
PaperMod
```

支持的 4 中数学公式语法：

行内公式：

```md
这是爱因斯坦公式：\(E = mc^2\)。
```

或

```md
这是爱因斯坦公式：$E = mc^2$。
```

块级公式：

```md
\[
E = mc^2
\]
```

或

```md
$$
E = mc^2
$$
```

例子：

### 行内公式

爱因斯坦质能方程为 \(E = mc^2\)。

牛顿第二定律：

\(F = ma\)。

### 块级公式

动能公式：

\[
E_k = \frac{1}{2}mv^2
\]

### 积分

\[
\int_0^1 x^2\,dx = \frac{1}{3}
\]

### 极限

\[
\lim_{x \to 0} \frac{\sin x}{x} = 1
\]

### 矩阵

\[
A =
\begin{bmatrix}
1 & 2 \\
3 & 4
\end{bmatrix}
\]

### 求和

\[
\sum_{i=1}^{n} i = \frac{n(n+1)}{2}
\]

### 微分方程

\[
\frac{dy}{dx} + P(x)y = Q(x)
\]

## 彩色引用

> 这是一个普通的引用
> 普通引用不带色彩，是 `PaperMod` 主题自带的样式。

> [!Note]
> 这是一个普通提示。

> [!Tip]
> 这是一个实用技巧。

> [!Warning]
> 这是一个**警告**。

> [!important]
> 这是一个 **重要的**。

> [!caution]
> 这是一个**危险**。


## 使用 GitHub Actions 自动化部署

在项目根目录创建：`.github\workflows` 目录，在workflows下创建  `gh-pages.yml` 文件，内容如下：

```yml
name: Deploy Hugo site to Pages

on:
  push:
    branches: ["main"] # 监听 main 分支推送
  workflow_dispatch:   # 允许手动触发

# 设置权限
permissions:
  contents: read
  pages: write
  id-token: write

# 避免并发冲突
concurrency:
  group: "pages"
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v5
        with:
          submodules: recursive
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: '0.164.0'
          extended: true

      - name: Build with Hugo
        run: hugo --minify

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v4
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

设置 Github 仓库 Pages 页面，Build and deployment 下的 Source 选择 `Github Actions`，然后就可以用 `git push` 推送仓库了，之后 `Github Actions` 就会自动化完成部署。
