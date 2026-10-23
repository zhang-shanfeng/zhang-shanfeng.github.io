---
title: 'Ne555 试验集合'
date: '2026-09-30T11:24:56+08:00'
draft: true
lastmod: '2026-09-30T11:24:56+08:00'

# 摘要和描述
summary: ""
description: ""

categories: ['electronics']
tags: ['NE555']

cover:
  image: "/posts/cover.jpg"
  alt: "cover"
  caption: "NE555"

# 作者
author: "zhang-shanfeng"

showtoc: true
comments: true

keywords: ['NE555']
---

**NE555** 是一款集成精密计时器芯片，能够实现延时、震荡等多种功能。通过外部简单的电阻、电容就能实现各种实用的功能。下面我们来看看这个芯片怎么实用。

下图是 **NE555** 内部的简化原理图：

![ne555](/images/posts/ne555.png)

## 引脚

|引脚|类型|描述|
|---|:---:|---|
|1（**GND**）|-| 接地|
|2（**TRIG**）|输入|计时输入开始。TRIG < ½ CONT 时将输出端置于高电平，放电端为开路状态|
|3（**OUT**）|输出|高电流计时器输出信号|
|4（**RESET**）|输入|低电平有效复位输入可强制将输出端和放电端置于低电平。|
|5（**CONT**）|输入/输出|控制比较器阈值，输出 2/3 × VCC，允许旁路电容器连接|
|6（**THRES**）|输入|计时输入结束。THRES > CONT 时将输出端和放电端置于低电平|
|7（**DISCH**）|输出|集电极开路输出，可对计时电容器进行放电|
|8（**VCC**）|-|4.5V 至 16V 输入电源电压。18V 时为 SE555（最大值）。|

## 真值表
