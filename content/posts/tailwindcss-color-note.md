---
title: 'Tailwindcss 中的颜色笔记'
date: '2026-08-20T11:00:28+08:00'
draft: true
lastmod: '2026-08-20T11:00:28+08:00'

# 摘要和描述
summary: ""
description: "记录 TailwindCSS V4 中预定义的颜色的使用方法、不透明度的用法、怎么写、注意事项、哪些类可用使用 /num 添加不透明度等。"

categories: ['frontend']
tags: ["TailwindCSS"]

cover:
  image: "/posts/cover.jpg"
  alt: "cover"
  caption: "cover"

# 作者
author: "zhang-shanfeng"

showtoc: true
comments: true

keywords: []
---

今天记录一下 TailwindCSS V4 中颜色的使用和不透明度用法等，没有深入学习的时候会发现代码中有这样的写法：`class="bg-black/50"` 就是把 `bg-black`颜色的不透明度转换为 `50%`，不经过专门查询还真不知道有这样的语法，特此记录一下。


## 内置的调色板

1. **Red** 红色
2. **Orange** 橙色
3. **Amber** 琥珀色
4. **Yellow** 黄色
5. **Lime** “酸橙绿”或“青柠色”
6. **Green** 绿色
7. **Emerald** “祖母绿”或“翡翠绿”
8. **Teal** 蓝绿色
9. **Cyan** 青色
10. **Sky** 天蓝色
11. **Blue** 蓝色
12. **Indigo** “Indigo”代表靛蓝色，是一种介于深蓝色和紫色之间的颜色，可以理解为蓝中带紫的深邃色调。
13. **Violet** “Violet”代表紫罗兰色，是一种偏冷调、介于红色和蓝色之间但更接近蓝色的淡紫色。
14. **Purple** 紫色
15. **Fuchsia** 紫红色
16. **Pink** 粉色
17. **Rose** 玫瑰色
18. **Slate** “Slate”代表石板灰，是一种带有蓝色或紫色调的深灰色，灵感来源于天然石板岩的颜色。
19. **Gray** 灰色
20. **Zinc** “Zinc”代表锌银色，是一种带有轻微蓝调的浅金属灰色，灵感来源于金属锌（Zinc）本身的颜色。
21. **Neutral** “中性色”、“中间色”
22. **Stone** “石灰色”、“石色”
23. **Taupe** 灰褐色
24. **Mauve** “锦葵紫”、“淡紫灰色”
25. **Mist** “雾色”、“薄雾灰”
26. **Olive** “橄榄绿”或“橄榄色”


## 颜色使用方法

默认调色板中的每种颜色都包含 11 个色阶，50 为最浅色，950 为最深色

例子：

```html
<div>
  <div class="bg-sky-50"></div>
  <div class="bg-sky-100"></div>
  <div class="bg-sky-200"></div>
  <div class="bg-sky-300"></div>
  <div class="bg-sky-400"></div>
  <div class="bg-sky-500"></div>
  <div class="bg-sky-600"></div>
  <div class="bg-sky-700"></div>
  <div class="bg-sky-800"></div>
  <div class="bg-sky-900"></div>
  <div class="bg-sky-950"></div>
</div>
```

以下是使用您调色板的所有实用程序的完整列表：

|Utility|Description|
|---|---|
|`bg-*`|背景颜色|
|`text-*`|文本颜色|
|`decoration-*`|文本装饰颜色|
|`border-*`|元素边框颜色|
|`outline-*`|设置元素的轮廓颜色|
|`shadow-*`|设置盒子阴影的颜色|
|`inset-shadow-*`|设置内嵌框阴影的颜色|
|`inset-ring-*`|设置戒指阴影的颜色|
|`accent-*`|设置表单控件的强调色|
|`caret-*`|设置表单控件中的光标颜色|
|`scrollbar-thumb-*`|设置元素滚动条的滑块颜色|
|`scrollbar-track-*`|设置元素滚动条的轨道颜色|
|`fill-*`|设置 SVG 元素的填充颜色|
|`caret-*`|设置表单控件中的光标颜色|
|`stroke-*`|设置 SVG 元素的描边颜色|

## 不透明度

您可以使用类似 bg-black/75 的语法调整颜色的不透明度，其中 75 将颜色的 alpha 通道设置为 75%：

```html
<div>
  <div class="bg-sky-500/10"></div>
  <div class="bg-sky-500/20"></div>
  <div class="bg-sky-500/30"></div>
  <div class="bg-sky-500/40"></div>
  <div class="bg-sky-500/50"></div>
  <div class="bg-sky-500/60"></div>
  <div class="bg-sky-500/70"></div>
  <div class="bg-sky-500/80"></div>
  <div class="bg-sky-500/90"></div>
  <div class="bg-sky-500/100"></div>
</div>
```

此语法还支持任意值和 CSS 变量简写：

```html
<div class="bg-pink-500/[71.37%]"><!-- ... --></div>
<div class="bg-cyan-400/(--my-alpha-value)"><!-- ... --></div>
```

## mode 深色选择器

使用 `dark` 变体编写类似 `dark:bg-gray-800` 的类，该类仅在深色模式激活时应用颜色：

```html
<div class="bg-white dark:bg-gray-800 rounded-lg px-6 py-8 ring shadow-xl ring-gray-900/5">
  <div>
    <span class="inline-flex items-center justify-center rounded-md bg-indigo-500 p-2 shadow-lg">
      <svg class="h-6 w-6 stroke-white" ...>
        <!-- ... -->
      </svg>
    </span>
  </div>
  <h3 class="text-gray-900 dark:text-white mt-5 text-base font-medium tracking-tight ">Writes upside-down</h3>
  <p class="text-gray-500 dark:text-gray-400 mt-2 text-sm ">
    The Zero Gravity Pen can be used to write in any orientation, including upside-down. It even works in outer space.
  </p>
</div>
```

## CSS 中的引用

颜色以 CSS 变量的形式暴露在 `--color-*` 命名空间中，因此您可以使用类似 `--color-blue-500` 和 `--color-pink-700` 的变量在 CSS 中引用它们：

```css
@import "tailwindcss";
@layer components {
  .typography {
    color: var(--color-gray-950);
    a {
      color: var(--color-blue-500);
      &:hover {
        color: var(--color-blue-800);
      }
    }
  }
}
```

您也可以在实用程序类中将这些值用作任意值：

```html
<div class="bg-[light-dark(var(--color-white),var(--color-gray-950))]">
  <!-- ... -->
</div>
```

为了在 CSS 中以变量形式引用颜色时快速调整其不透明度，Tailwind 包含一个特殊的 `--alpha()` 函数：

```css
@import "tailwindcss";
@layer components {
  .DocSearch-Hit--Result {
    background-color: --alpha(var(--color-gray-950) / 10%);
  }
}
```

## 自定义颜色

使用 @theme 可以在 --color-* 主题命名空间下为项目添加自定义颜色：

```css
@import "tailwindcss";
@theme {
  --color-midnight: #121063;
  --color-tahiti: #3ab7bf;
  --color-bermuda: #78dcca;
}
```

现在，除了默认颜色之外，您的项目中还将提供 `bg-midnight`、`text-tahiti` 和 `fill-bermuda` 等实用工具。

## 覆盖默认颜色

通过定义同名的新主题变量，可以覆盖任何默认颜色：

```css
@import "tailwindcss";
@theme {
  --color-gray-50: oklch(0.984 0.003 247.858);
  --color-gray-100: oklch(0.968 0.007 247.896);
  --color-gray-200: oklch(0.929 0.013 255.508);
  --color-gray-300: oklch(0.869 0.022 252.894);
  --color-gray-400: oklch(0.704 0.04 256.788);
  --color-gray-500: oklch(0.554 0.046 257.417);
  --color-gray-600: oklch(0.446 0.043 257.281);
  --color-gray-700: oklch(0.372 0.044 257.287);
  --color-gray-800: oklch(0.279 0.041 260.031);
  --color-gray-900: oklch(0.208 0.042 265.755);
  --color-gray-950: oklch(0.129 0.042 264.695);
}
```

## 禁用默认颜色

通过将主题命名空间设置为 `initial` 来禁用任何默认颜色：

```css
@import "tailwindcss";
@theme {
  --color-lime-*: initial;
  --color-fuchsia-*: initial;
}
```

对于您不打算使用的颜色，此功能尤其有用，可以从输出中删除相应的 CSS 变量。

## 使用自定义调色板

使用 `--color-*: initial` 参数可以完全禁用所有默认颜色，并定义您自己的自定义调色板：

```css
@import "tailwindcss";
@theme {
  --color-*: initial;
  --color-white: #fff;
  --color-purple: #3f3cbb;
  --color-midnight: #121063;
  --color-tahiti: #3ab7bf;
  --color-bermuda: #78dcca;
}
```

## 参考其他变量

在定义引用其他颜色的颜色时，请使用 @theme 内联命令：

```css
@import "tailwindcss";
:root {
  --acme-canvas-color: oklch(0.967 0.003 264.542);
}
[data-theme="dark"] {
  --acme-canvas-color: oklch(0.21 0.034 264.665);
}
@theme inline {
  --color-canvas: var(--acme-canvas-color);
}
```

参考：

- [https://tailwindcss.com/docs/colors](https://tailwindcss.com/docs/colors)
