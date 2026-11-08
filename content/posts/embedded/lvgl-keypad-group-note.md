---
title: 'Lvgl 键盘与组的使用笔记'
date: '2026-11-05'
draft: false
lastmod: '2026-11-05'

categories: ['electronics']
tags: ['LVGL', 'Group', 'Keypad']

cover: "/images/posts/lvgl_keypad_spinbox_group_01.png"


showtoc: true
comments: true

keywords: ['LVGL', 'Group', 'Keypad']
---

介绍 `LVGL` 中**keypad**输入设备与**Group**相关的基础知识。

<!--more-->

## 基本描述

首先说说我要实现的功能，我想要实现用 5 个实体按键来完成LVGL 中的：焦点移动（前/后）、数值调整（加/减）、左移/右移（如：spinbox 控件中指示当前编辑哪一位的光标）、OK按键（Enter)，这里有七个功能，分别对应 LVGL 中的按键标志：`LVGL_KEY_PREV`、`LVGL_KEY_NEXT`、`LVGL_KEY_UP`、`LVGL_KEY_DOWN`、`LVGL_KEY_LEFT`、`LVGL_KEY_RIGHT`和`LVGL_KEY_Enter`。

> [!tip]
> 注意我用的 LVGL 输入设备类型为 `LV_INDEV_TYPE_KEYPAD`
> ```c
> lv_indev_set_type(keypad_indev, LV_INDEV_TYPE_KEYPAD);
> ```

## 创建输入设备并配置读取回调函数

```c
keypad_indev = lv_indev_create();
lv_indev_set_type(keypad_indev, LV_INDEV_TYPE_KEYPAD);
lv_indev_set_read_cb(keypad_indev, keypad_read_cb);
```

```c
/**************************************************************************************************
 * @brief 给 LVGL 用的函数
 *
 * @param indev
 * @param data
 * @date 2026-10-30
 *************************************************************************************************/
void keypad_read_cb(lv_indev_t* indev, lv_indev_data_t* data)
{
    xSemaphoreTake(g_key_mutex, portMAX_DELAY);
    if (g_current_lv_key != 0)
    {
        data->key = g_current_lv_key;
        data->state = LV_INDEV_STATE_PRESSED;
    }
    else
    {
        data->state = LV_INDEV_STATE_RELEASED;
    }
    xSemaphoreGive(g_key_mutex);
}
```


## 按键回调函数（ESP-IDF 官方的按键组件）

我用的是 ESP-IDF 官方的按键组件：[espressif/button](https://components.espressif.com/components/espressif/button/versions/4.2.3/readme?language=en)

用 5 个按键实现7个LVGL 功能，光标移动和数值加减需要动态复用，使用一个环境变量来实现，下面时按键回调函数：

```c
static void button_event_cb(void* arg, void* usr_data)
{
    int btn_id = (int)(intptr_t)usr_data;
    button_event_t event = iot_button_get_event((button_handle_t)arg);
    uint32_t lv_key = 0;

    switch (btn_id)
    {
        case 1:
            lv_key = is_edited_mode ? LV_KEY_UP : LV_KEY_PREV;
            break;
        case 2:
            lv_key = is_edited_mode ? LV_KEY_DOWN : LV_KEY_NEXT;
            break;
        case 3:
            lv_key = LV_KEY_ENTER;
            break;
        case 4:
            lv_key = LV_KEY_LEFT;
            break;
        case 5:
            lv_key = LV_KEY_RIGHT;
            break;
        default:
            break;
    }

    if (event == BUTTON_PRESS_DOWN)
    {
        xSemaphoreTake(g_key_mutex, portMAX_DELAY);
        g_current_lv_key = lv_key;
        xSemaphoreGive(g_key_mutex);
    }
    else if (event == BUTTON_PRESS_UP || event == BUTTON_PRESS_END)
    {
        xSemaphoreTake(g_key_mutex, portMAX_DELAY);
        if (g_current_lv_key == lv_key) { g_current_lv_key = 0; }
        xSemaphoreGive(g_key_mutex);
    }
}
```

`is_edited_mode` 是一个全局变量，用量控制两个按键对应 LVGL 中的哪个功能。

实物截图：

下图为聚焦点在电流输入框，默认为非编辑状态（聚焦轮廓为蓝色）

![](/images/posts/lvgl_keypad_spinbox_group_01.png)

下图为聚焦点在电流输入框，并且处于编辑状态(`lv_obj_set_edited(u, is_edited_mode);`,同时移除蓝色块隐藏的样式。

![](/images/posts/lvgl_keypad_spinbox_group_02.png)

## 隐藏 spinbox 光标的方法

各个设置 spinbox 光标的这3个属性即可隐藏光标的显示：

```c
lv_style_set_bg_opa(&style_spinbox_cursor, 0);
lv_style_set_text_opa(&style_spinbox_cursor, 0);
lv_style_set_border_width(&style_spinbox_cursor, 0);
```

### 使用方式（控制spinbox 光标的隐藏与显示）

下面这个函数是图中三个 spinbox 控件的事件处理函数，`u` 就是当前我们点击的 spinbox 这个控件的指针，当点击了 spinbox （LV_EVENT_CLICKED)时调用该函数，执行 is_edited_mode 的取反（是否为编辑模式）用在 `button_event_cb` 函数中控制哪两个按键是移动焦点的功能，还是数值加减的功能。

还要手动控制 spinbox 光标的显示与隐藏，当然我们也可以定制光标的样式，默认为蓝色的方块。

`lv_obj_add_style()` 最后一个参数（选择器）需要选择 `LV_PART_CURSOR`,代表这个样式用于 spinbox 的光标部分。

```c
static void spinbox_handle(lv_event_t* e)
{
    lv_obj_t* u = lv_event_get_target_obj(e);

    is_edited_mode = !is_edited_mode;
    lv_obj_set_edited(u, is_edited_mode);
    if (!is_edited_mode)
    {
        lv_obj_add_style(u, &style_spinbox_cursor, LV_PART_CURSOR);
    }
    else
    {
        lv_obj_remove_style(u, &style_spinbox_cursor, LV_PART_CURSOR);
    }
}
```

## 包含 Label 和 Spinbox 的行生成函数

下面是`Label` 和 `Spinbox` 的行生成函数要用的的配置参数结构体：

```c
#pragma once

#include "lvgl.h"

struct ui_spinbox_cfg_t
{
    const lv_font_t* font;
    int32_t min_value;    /* spinbox 最小值 */
    int32_t max_value;    /* spinbox 最大值 */
    uint32_t digit_count; /* 不含小数点和正负号的数字位数 */
    uint32_t sep_pos;     /* 小数点前的数字位数。如果为0，则不显示小数点 */
    int32_t radius;       /* spinbox 圆角大小 */
    int32_t pad_ver;      /* 上下内边距 */
    int32_t pad_hor;      /* 左右内边距 */
    lv_color_t value_color;
    bool has_value_color;
    int32_t width;
    int32_t height;
};

struct ui_label_cfg_t
{
    const char* title;
    const lv_font_t* font;
    lv_color_t color;
    bool has_color;
    int32_t width;
    int32_t height;
    int32_t pad_ver; /* 上下内边距 */
    int32_t pad_hor; /* 左右内边距 */
};

struct ui_kv_row_spinbox_cfg_t
{
    int32_t width;   /* row 宽度 */
    int32_t height;  /* row 高度 */
    int32_t pad_ver; /* row 上下内边距 */
    int32_t pad_hor; /* row 左右内边距 */
    struct ui_label_cfg_t label;
    struct ui_spinbox_cfg_t spinbox;
};
```

下面是生成ui_kv_row_spinbox_cfg_t 配置信息的工厂函数：

```c
struct ui_kv_row_spinbox_cfg_t make_row_spinbox_kv_cfg(const char* title)
{
    return (struct ui_kv_row_spinbox_cfg_t){
        .width = lv_pct(100),
        .height = LV_SIZE_CONTENT,
        .pad_hor = 6,
        .pad_ver = 6,
        .label =
            {
                .title = title,
                .width = LV_SIZE_CONTENT,
                .height = LV_SIZE_CONTENT,
                .has_color = true,
                .color = lv_color_white(),
                .pad_hor = 0,
                .pad_ver = 0,
                .font = &alibaba_puhuiti_regular_24_4bpp,
            },
        .spinbox = {
            .min_value = 0,
            .max_value = 9999,
            .digit_count = 4,
            .sep_pos = 1,
            .width = LV_SIZE_CONTENT,
            .height = LV_SIZE_CONTENT,
            .font = &ibm_plex_mono_medium_24_2bpp,
            .has_value_color = true,
            .value_color = lv_color_black(),
            .radius = 8,
            .pad_hor = 4,
            .pad_ver = 4,
        }};
}
```

下面是创建包含 Label 和 spinbox 控件的行

```c
/**************************************************************************************************
 * @brief 创建一行，内部放两个控件，左侧 Label,右侧 spinbox
 *
 * @note 备注
 * @param parent
 * @return lv_obj_t*
 * @date 2026-11-04
 *************************************************************************************************/
lv_obj_t* ui_create_kv_row_spinbox(lv_obj_t* parent,
                                   const struct ui_kv_row_spinbox_cfg_t* cfg)
{
    lv_obj_t* cont = lv_obj_create(parent);
    lv_obj_remove_style_all(cont);
    lv_obj_set_size(cont, cfg->width, cfg->height);
    lv_obj_set_flex_flow(cont, LV_FLEX_FLOW_ROW);
    lv_obj_set_flex_align(cont,
                          LV_FLEX_ALIGN_SPACE_BETWEEN,
                          LV_FLEX_ALIGN_CENTER,
                          LV_FLEX_ALIGN_START);
    if (cfg->pad_ver) { lv_obj_set_style_pad_ver(cont, cfg->pad_ver, 0); }
    if (cfg->pad_hor) { lv_obj_set_style_pad_hor(cont, cfg->pad_hor, 0); }

    lv_obj_t* title = lv_label_create(cont);
    lv_obj_set_style_text_font(title, cfg->label.font, 0);
    if (cfg->label.has_color)
    {
        lv_obj_set_style_text_color(title, cfg->label.color, 0);
    }
    lv_obj_set_size(title, cfg->label.width, cfg->label.height);
    lv_label_set_text(title, cfg->label.title);

    lv_obj_t* spinbox = lv_spinbox_create(cont);
    lv_spinbox_set_range(
        spinbox, cfg->spinbox.min_value, cfg->spinbox.max_value);
    lv_spinbox_set_digit_format(
        spinbox, cfg->spinbox.digit_count, cfg->spinbox.sep_pos);
    lv_spinbox_step_prev(spinbox);
    if (cfg->spinbox.pad_ver)
    {
        lv_obj_set_style_pad_ver(spinbox, cfg->spinbox.pad_ver, 0);
    }
    if (cfg->spinbox.pad_hor)
    {
        lv_obj_set_style_pad_hor(spinbox, cfg->spinbox.pad_hor, 0);
    }
    lv_obj_set_style_text_font(spinbox, &ibm_plex_mono_medium_24_2bpp, 0);
    lv_obj_set_style_radius(spinbox, cfg->spinbox.radius, 0);
    lv_obj_set_size(spinbox, cfg->spinbox.width, cfg->spinbox.height);

    return spinbox;
}
```

## demo 函数

```c
static void demo1(lv_obj_t* parent)
{
    static bool inited = false;
    if (!inited)
    {
        lv_style_init(&style_spinbox_cursor);
        lv_style_set_bg_opa(&style_spinbox_cursor, 0);
        lv_style_set_text_opa(&style_spinbox_cursor, 0);
        lv_style_set_border_width(&style_spinbox_cursor, 0);
        inited = true;
    }

    lv_obj_set_style_bg_color(parent, lv_color_black(), 0);
    lv_obj_set_style_bg_opa(parent, LV_OPA_COVER, 0);

    lv_obj_set_layout(parent, LV_LAYOUT_FLEX);
    lv_obj_set_flex_flow(parent, LV_FLEX_FLOW_COLUMN);

    struct ui_kv_row_spinbox_cfg_t row_spinbox_cfg =
        make_row_spinbox_kv_cfg("电流");
    lv_obj_t* current_spinbox =
        ui_create_kv_row_spinbox(parent, &row_spinbox_cfg);
    lv_obj_add_style(current_spinbox, &style_spinbox_cursor, LV_PART_CURSOR);

    row_spinbox_cfg = make_row_spinbox_kv_cfg("电压");
    lv_obj_t* voltage_spinbox =
        ui_create_kv_row_spinbox(parent, &row_spinbox_cfg);
    lv_obj_add_style(voltage_spinbox, &style_spinbox_cursor, LV_PART_CURSOR);

    row_spinbox_cfg = make_row_spinbox_kv_cfg("功率");
    lv_obj_t* power_spinbox = ui_create_kv_row_spinbox(parent, &row_spinbox_cfg);
    lv_obj_add_style(power_spinbox, &style_spinbox_cursor, LV_PART_CURSOR);

    lv_obj_add_event_cb(current_spinbox, spinbox_handle, LV_EVENT_CLICKED, NULL);
    lv_obj_add_event_cb(voltage_spinbox, spinbox_handle, LV_EVENT_CLICKED, NULL);
    lv_obj_add_event_cb(power_spinbox, spinbox_handle, LV_EVENT_CLICKED, NULL);
}
```

## Spinbox 有哪些状态

Spinbox 本身没有定义一套独占的特殊状态枚举，它作为基于 Text Area 的衍生控件，通用对象状态均适用。在实际开发中，最常用来让 Spinbox 显示不同样式的状态主要有：

|状态|含义|典型用途|
|---|---|---|
|`LV_STATE_DEFAULT`|默认/空闲|普通显示样式|
|`LV_STATE_FOCUSED`|获得焦点|编码器或键盘导航选中该控件时，高亮边框或背景|
|`LV_STATE_EDITED`|编辑模式|进入数字调整状态时，改变光标或边框样式以示区分|
|`LV_STATE_PRESSED`|被按下|触摸或按键触发时的反馈|
|`LV_STATE_DISABLED`|禁用|变灰或降低透明度|

此外，`LV_STATE_CHECKED` 等通用状态在 Spinbox 上较少使用，但语法上同样可以绑定样式。
