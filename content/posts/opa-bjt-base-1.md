---
title: '用运算放大器驱动三极管推挽电路基础分析'
date: '2026-08-24T23:45:32+08:00'
draft: true
lastmod: '2026-08-24T23:45:32+08:00'

# 摘要和描述
summary: ""
description: ""

categories: ['electronics']
tags: ["运放", "BJT"]

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

在这里分享一个用运放驱动三极管推挽电路的最简方法，电路中用一个 TL072 运放和两个三极管 BD139\BD140 组成了一个最简单的试验电路，该电路并不完美拿来学习是比较好的例子，下面就分析一下这个电路：

!["TL072-PNP-NPN-01"](/images/posts/TL072-PNP-NPN-01.png)

图中 R111 在这个电路中非常关键，如果缺少的化，在输入幅度较低时输出波形就会出现交越失真，加上这个电阻后就解决了输入小幅度时输出交越失真的问题。

缺少 R111 时三极管因为发射结需要0.5V左右的开启电压，所以在输入信号很小时，运放的输出达不到三极管发射结的开启电压，这就是交越失真的原因。当有 R111时，即使运放的输出很小也能通过该电阻直接输出，所以示波器看到的就越实战就消失了。

但是这种方法没有解决三极管发射结在小信号时的不导通状态（没有工作在 AB 类功放状态），最终还是需要让电路工作在 AB 状态。

下面是加电阻和不加电阻的波形：

![”有 R111 的波形“](/images/posts/TL072-PNP-NPN-waveform-1.png "有 R111 的波形")

![”无 R111 的波形“](/images/posts/TL072-PNP-NPN-waveform-2.png "无 R111 的波形")
