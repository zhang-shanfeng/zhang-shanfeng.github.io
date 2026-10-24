---
title: 'C 语言宏笔记'
date: '2026-10-02T20:57:36+08:00'
draft: false
lastmod: '2026-10-02T20:57:36+08:00'

# 摘要和描述
summary: ""
description: ""

categories: ['frontend', 'electronics', 'notes', 'sharing']
tags: []

cover: "/images/posts/cover.jpg"

# 作者
author: "zhang-shanfeng"

showtoc: true
comments: true

keywords: ['C语言','宏定义','条件宏']
---

记录以下 C 语言的宏定义、条件判断、推荐写法等。

<!--more-->

## 用 1/0 值宏作为判断条件（编译开关）

```c
#define USE_DMA 1

#if (USE_DMA == 1)
...
#endif
```

## 多条分支的 #if

每个 #if 都配 #endif 注释

```c
#if defined(AT32F435)     /* AT32F435 */
    ...
#elif defined(AT32F437)   /* AT32F437 */
    ...
#endif /* AT32F435 / AT32F437 */
```

## defined() 是否定义宏

`defined()` 不要求宏有值，只要定义就为真。

```c
#ifndef AT32_CONFIG_H
#define AT32_CONFIG_H

/* ------- 芯片选择 ------- */
#define AT32F435
/* #define AT32F437 */

/* ------- 功能开关 ------- */
#define USE_USART1      1
#define USE_SPI1        1
#define USE_DEBUG_LOG   1

/* ------- 派生宏 ------- */
#if defined(AT32F435) || defined(AT32F437)
    #define AT32F4XX
#endif

#endif /* AT32_CONFIG_H */
```

条件组合：

```c
/* 与 */
#if defined(FEATURE_A) && defined(FEATURE_B)
    ...
#endif

/* 或 */
#if defined(FEATURE_A) || defined(FEATURE_B)
    ...
#endif

/* 非 */
#if !defined(FEATURE_A)
    ...
#endif

/* 复杂组合（推荐加括号） */
#if (defined(AT32F435) || defined(AT32F437)) && (USE_USART1 == 1)
    ...
#endif
```
