---
title: 'St7789 方向控制笔记'
date: '2026-09-14T00:24:48+08:00'
draft: false
lastmod: '2026-09-14T00:24:48+08:00'

# 摘要和描述
summary: ""
description: ""

categories: ['electronics']
tags: [ST7789]

cover:
  image: "/posts/cover.jpg"
  alt: "cover"
  caption: "ST7789 显示方向控制"

# 作者
author: "zhang-shanfeng"

showtoc: true
comments: true

keywords: [ST7789, '方向控制']
---

ST7789 是一款通用控制器，设计上支持最大 240×320 的 GRAM。但面板厂商可以：

- 做 240×320 的全尺寸面板 → 1:1 对应
- 做 172×320 的小尺寸面板 → 只用 GRAM 的一部分
- 做 240×240 的方形面板 → 只用一个子区域

```txt
┌─────────────────────────────────────────┐
│  ST7789 芯片内部 GRAM                    │
│  - 由控制器硅片决定                       │
│  - 固定：240 × 320 × 18bit              │
│  - 不可变（硬件定死）                     │
│  - 地址空间：列 0~239，行 0~319           │
└─────────────────────────────────────────┘
              ↕ 通过驱动电路扫描输出
┌─────────────────────────────────────────┐
│  物理面板（玻璃）                         │
│  - 实际可见的像素阵列                     │
│  - 尺寸：172 × 320（你的屏）             │
│  - 由面板厂商决定，与 GRAM 尺寸无关        │
└─────────────────────────────────────────┘
```

## 方向控制 MADCTL

无论怎么旋转，你传入的 (x0, y0) 永远代表用户看到的左上角。旋转由 MADCTL 寄存器自动完成映射。

ST7789 用一个字节控制方向，关键位：

|位|名称|含义|
|---|---|---|
|D7|MY|行地址方向（Page Address Order）|
|D6|MX|列地址方向（Column Address Order）|
|D5|MV|行列交换（Row/Column Exchange）|
|D4|ML|垂直刷新方向|
|D3|RGB|BGR/RGB 顺序|
|D2|MH|水平刷新方向|

旋转 = MX + MY + MV 三个位的组合：

|旋转|MX|MY|MV|十六进制|
|---|---|---|---|---|
|0°（竖屏）|0|0|0|0x00|
|90°（横屏）|1|0|1|0x60|
|180°（竖屏倒置）|1|1|0|0xC0|
|270°（横屏倒置）|0|1|1|0xA0|

（实际还要叠加 RGB/BGR 位，常见值：0x00 / 0x60 / 0xC0 / 0xA0）

## 为什么需要 X_OFFSET / Y_OFFSET？

你的面板是 172×320，但 ST7789 控制器内部显存是 240×320！

![st7789_01.png](/images/posts/st7789_01.png)

### 竖屏 0° 时

- 逻辑 X（0\~171）→ 物理列（34~205）
- `X_OFFSET = 34`（你的代码写 0，可能有问题！）
- `Y_OFFSET = 0`

### 横屏 90° 时

- 逻辑坐标系被旋转，X/Y 互换
- 原来列方向的偏移现在变成了行方向
- 所以 Y_OFFSET = 34，X_OFFSET = 0


## 方向控制的函数

```c
void spilcd_set_rotation(uint8_t rot)
{
    // /// X_OFFSET/Y_OFFSET/spilcd_width/spilcd_height 要在 spilcd_set_window 函数中使用
    spilcd_write_cmd(0x36);  // MADCTL
    switch (rot) {
        case 0:   // 竖屏 0°
            spilcd_write_data(0x00);
            X_OFFSET = 34; Y_OFFSET = 0;
            spilcd_width = 172; spilcd_height = 320;
            break;
        case 1:   // 横屏 90°
            spilcd_write_data(0x60);
            X_OFFSET = 0; Y_OFFSET = 34;
            spilcd_width = 320; spilcd_height = 172;
            break;
        case 2:   // 竖屏 180°
            spilcd_write_data(0xC0);
            X_OFFSET = 34; Y_OFFSET = 0;
            spilcd_width = 172; spilcd_height = 320;
            break;
        case 3:   // 横屏 270°
            spilcd_write_data(0xA0);
            X_OFFSET = 0; Y_OFFSET = 34;
            spilcd_width = 320; spilcd_height = 172;
            break;
    }
}
```
