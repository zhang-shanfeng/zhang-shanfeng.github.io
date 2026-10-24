---
title: 'Lvgl 主题与数据绑定简单示例_使用 lv_xx_bind_xx()绑定主题'
date: '2026-10-24T15:42:22+08:00'
draft: false
lastmod: '2026-10-24T15:42:22+08:00'

categories: ['electronics']
tags: ['LVGL', 'Subject', 'Bind']

cover: "/images/posts/lvgl-subject-data-bind-base.png"


showtoc: true
comments: true

keywords: ['LVGL', '数据绑定', 'Subject', '主题', '订阅者', 'Slider', 'Label']
---

介绍 `LVGL` 中**主题**与**数据绑定**相关的基础知识。

<!--more-->

一个主题可以被多个 LVGL 控件绑定，主题的值发生变化时所有的控件也都会跟着变化，如图中有四个控件和一个主题，上面的 Slider 和 Label 都绑定到了一个整型主题(Subject),下面是动图：

![](/images/posts/lvgl_subject_data_bind_demo_1.gif)

```c
/**************************************************************************************************
 * @file subject_observer_01_int.c
 * @author zhang-shanfeng (freerealmshn@gmail.com)
 * @brief 主题@订阅者 整型例子
 * @version V0.1.0
 * @date 2026-10-21
 *
 * @copyright Copyright (c) 2025 by zhang-shanfeng, All Rights Reserved.
 * @website https://zhang-shanfeng.github.io
 *
 *************************************************************************************************/
#include "subject_observer_01_int.h"

LV_FONT_DECLARE(alibaba_puhuiti_3_55_regular_18);

lv_obj_t* temp_label;
lv_subject_t temp_subj;

void btn_add_cb(lv_event_t* e)
{
    int32_t v = lv_subject_get_int(&temp_subj);
    lv_subject_set_int(&temp_subj, v + 1);
}

void btn_sub_cb(lv_event_t* e)
{
    int32_t v = lv_subject_get_int(&temp_subj);
    lv_subject_set_int(&temp_subj, v - 1);
}

void subject_observer_01_int(void)
{
    /* 获取默认显示器的活动屏幕 */
    lv_obj_t* scr = lv_screen_active();
    /* 设置布局为 Flex 布局 */
    lv_obj_set_layout(scr, LV_LAYOUT_FLEX);
    /* 设置屏幕大小宽高都为 100% */
    lv_obj_set_size(scr, lv_pct(100), lv_pct(100));
    /* 设置 Flex 为列布局，主轴方向为 Y 方向 */
    lv_obj_set_flex_flow(scr, LV_FLEX_FLOW_COLUMN);
    lv_obj_set_flex_align(
        scr, LV_FLEX_ALIGN_CENTER, LV_FLEX_ALIGN_CENTER, LV_FLEX_ALIGN_CENTER);

    /* 1.创建一个 subject */
    lv_subject_init_int(&temp_subj, 0);

    /* 创建一个 slider */
    lv_obj_t* slider = lv_slider_create(scr);
    lv_slider_bind_value(slider, &temp_subj);

    /* 创建 label */
    temp_label = lv_label_create(scr);

    /* 设置 label 样式 */
    lv_obj_set_style_text_color(temp_label,
                                lv_palette_lighten(LV_PALETTE_DEEP_ORANGE, 1),
                                LV_STATE_DEFAULT);
    lv_obj_set_style_text_font(
        temp_label, &alibaba_puhuiti_3_55_regular_18, LV_STATE_DEFAULT);
    lv_label_bind_text(temp_label, &temp_subj, "%d °C");

    /* 加按键 */
    lv_obj_t* btn1 = lv_button_create(scr);
    lv_obj_set_size(btn1, lv_pct(20), LV_SIZE_CONTENT);
    lv_obj_t* btn1_label = lv_label_create(btn1);
    lv_obj_set_align(btn1_label, LV_ALIGN_CENTER);
    lv_label_set_text(btn1_label, "+");
    lv_obj_add_event_cb(btn1, btn_add_cb, LV_EVENT_CLICKED, NULL);

    /* 减按键 */
    lv_obj_t* btn2 = lv_button_create(scr);
    lv_obj_set_size(btn2, lv_pct(20), LV_SIZE_CONTENT);
    lv_obj_t* btn2_label = lv_label_create(btn2);
    lv_obj_set_align(btn2_label, LV_ALIGN_CENTER);
    lv_label_set_text(btn2_label, "-");
    lv_obj_add_event_cb(btn2, btn_sub_cb, LV_EVENT_CLICKED, NULL);
}
```


例子中用 `lv_label_bind_text()` 和 `lv_slider_bind_value()` 分别给 label 和 slider 绑定主题，这两个函数属于顶层函数，另外我们还可以调用 `lv_subject_add_observer(subject, cb, user_data)` 手动订阅和 `lv_subject_add_observer_obj(subject, cb, widget, user_data)` 订阅并绑定Widget。

## Subject 支持的主题类型

|类型|枚举值|
|---|---|
|整数|`LV_SUBJECT_TYPE_INT`|
|浮点数|`LV_SUBJECT_TYPE_FLOAT`（需 `LV_USE_FLOAT`）|
|字符串|`LV_SUBJECT_TYPE_STRING`|
|指针|`LV_SUBJECT_TYPE_POINTER`|
|颜色|`LV_SUBJECT_TYPE_COLOR`|
|组|`LV_SUBJECT_TYPE_GROUP`|

创建主题：

```c
lv_subject_t * subject = lv_subject_create(LV_SUBJECT_TYPE_INT);
```

## lv_subject_init_int()

`lv_subject_init_int()` 初始化一个整数类型的 Subject。

原型声明：

```c
void lv_subject_init_int(lv_subject_t * subject, int32_t value);
```

## lv_subject_set_int()

`lv_subject_set_int()` 设置一个整数型“主题”（Subject）的值，并通知“观察者”（Observers）。

原型声明：

```c
void lv_subject_set_int(lv_subject_t * subject, int32_t value);
```

## lv_subject_get_int()

`lv_subject_get_int()` 获取整型 Subject 的当前值。

原型声明：

```c
int32_t lv_subject_get_int(lv_subject_t * subject);
```

## lv_label_bind_text()

`lv_label_bind_text()` 这个函数的作用是单向绑定，Subject 的值发生变化时，Label 的值自动刷新。

## lv_slider_bind_value()

`lv_slider_bind_value()` 这个函数的作用是双向绑定（**Two-way bind**）,它的双向同步机制如下：

- **Subject -> Slider**：当 Subject 的值被外部（如代码逻辑或其他控件）修改时，Slider 会自动同步更新，控制柄移动到对应位置。
- **Slider -> Subject**：当用户拖动控制柄时，Slider 会将新值写回 Subject，触发所有绑定该 Subject 的控件（如 Label）实时更新。

从代码层面看，`lv_slider_bind_value` 内部会创建一个 `lv_observer_t` 来监听 Subject 变化；同时在 Slider 上注册 `LV_EVENT_VALUE_CHANGED` 回调，把用户拖动的值写回 Subject，两者结合构成了完整的双向通道
