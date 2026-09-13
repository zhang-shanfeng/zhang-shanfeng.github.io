---
title: 'AT32 单片机 DWT us 延时代码分享'
date: '2026-09-12T10:00:51+08:00'
draft: true
lastmod: '2026-09-12T10:00:51+08:00'

# 摘要和描述
summary: ""
description: ""

categories: ['electronics']
tags: [AT32, DWT, us]

cover:
  image: "/posts/cover.jpg"
  alt: "cover"
  caption: "AT32 DWT us 延时"

# 作者
author: "zhang-shanfeng"

showtoc: true
comments: true

keywords: []
---

在这里分享自己的一份使用 DWT (Data Watchpoint and Trace) 计数器实现的 us 延时，该函数使用 DWT 的 CYCCNT 计数器来实现微秒级延时。

## 头文件

```c
/**************************************************************************************************
 * @file xtp_delay.h
 * @author zhang-shanfeng (xiaotupo@163.com)
 * @brief 
 * @version V0.1.0
 * @date 2025-08-31
 * 
 * @copyright Copyright (c) 2025 by zhang-shanfeng, All Rights Reserved.
 * @website https://zhang-shanfeng.github.io/
 * 
 *************************************************************************************************/
#ifndef __XTP_DELAY_H
#define __XTP_DELAY_H

#ifdef __cplusplus
extern "C" {
#endif

#include "at32f421.h"

void delay_us(uint32_t us);
void delay_ms(uint32_t ms);
	
#ifdef __cplusplus
}
#endif

#endif

```

## 源文件

```c
/**************************************************************************************************
 * @file xtp_delay.c
 * @author zhang-shanfeng (xiaotupo@163.com)
 * @brief 
 * @version V0.1.0
 * @date 2025-08-31
 * 
 * @copyright Copyright (c) 2025 by zhang-shanfeng, All Rights Reserved.
 * @website https://zhang-shanfeng.github.io/
 * 
 *************************************************************************************************/
#include "xtp_delay.h"

/**
 * @brief us 延时函数,使用 DWT (Data Watchpoint and Trace) 计数器
 * * 该函数使用 DWT 的 CYCCNT 计数器来实现微秒级延时。
 * * 不要用该函数实现 ms 延时, 因为会降低系统性能。
 * * @note 该函数需要在系统启动时使能 DWT 计数器。
 * 
 * @param us 
 */
void delay_us(uint32_t us)
{
	/* 仅在首次调用时初始化DWT（判断计数器是否未使能）*/
    if (!(DWT->CTRL & DWT_CTRL_CYCCNTENA_Msk)) {
        CoreDebug->DEMCR |= CoreDebug_DEMCR_TRCENA_Msk;  // 使能调试跟踪
        DWT->CYCCNT = 0;                                 // 清零计数器
        DWT->CTRL |= DWT_CTRL_CYCCNTENA_Msk;             // 使能CYCCNT
    }
	
	uint32_t start = DWT->CYCCNT;
	uint32_t ticks = (SystemCoreClock / 1000000) * us;
	while((DWT->CYCCNT - start) < ticks);
}

void delay_ms(uint32_t ms)
{
    if (!(DWT->CTRL & DWT_CTRL_CYCCNTENA_Msk)) {
        CoreDebug->DEMCR |= CoreDebug_DEMCR_TRCENA_Msk;
        DWT->CYCCNT = 0;
        DWT->CTRL |= DWT_CTRL_CYCCNTENA_Msk;
    }

    uint32_t start = DWT->CYCCNT;
    uint32_t ticks = (SystemCoreClock / 1000) * ms; // 一毫秒对应的时钟周期数
    while ((DWT->CYCCNT - start) < ticks);
}
```

> [!tip]
> 要注意的是使用这个函数需要在 MDK5 中开启 `C99` 标准，否则默认的为 C89 标准，会报如下错误：
> ```bash
> ..\..\lcd\xtp_delay.c(31): error:  #268: declaration may not appear after executable > statement in block
>   	uint32_t start = DWT->CYCCNT;
> ..\..\lcd\xtp_delay.c(32): error:  #268: declaration may not appear after executable statement in block
>   	uint32_t ticks = (SystemCoreClock / 1000000) * us;
> ..\..\lcd\xtp_delay.c(44): error:  #268: declaration may not appear after executable  statement in block
> ```

❌ `C89` 非法：声明出现在可执行语句之后错误的例子：

```c
void f(void)
{
    int a = 10;
    a = a + 1;       // ← 可执行语句
    int b = a + 10;  // ← 声明出现在可执行语句之后，报错 #268
}
```
