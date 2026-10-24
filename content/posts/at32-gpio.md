---
title: 'AT32 GPIO 相关笔记'
draft: true

categories: [Electronics]
tags: []
cover: "/images/posts/cover.jpg"
keywords: ['AT32', 'GPIO']
---

记录 AT32 的**GPIO**相关笔记，如：相关函数、常用的寄存器操作、IO 翻转状态等。

<!--more-->

## IO 翻转 Toggle

`odt` 寄存器:

输出模式：当引脚配置为通用输出时，写入 ODT 寄存器的值会直接输出到对应的 I/O 引脚上。0 代表低电平，1 代表高电平。

```c
#define STATE_LED_TOGGLE() StateLed_GPIO_PORT->odt ^= StateLed_PIN
```

## gpio 写函数
