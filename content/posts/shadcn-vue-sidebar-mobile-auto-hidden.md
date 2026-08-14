---
title: 'Shadcn-vue 移动端抽屉（Sidebar） 点击链接后自动关闭“抽屉”功能的实现方案'
date: '2026-08-14T16:54:33+08:00'
draft: false
lastmod: '2026-08-14T16:54:33+08:00'

# 摘要和描述
summary: ""
description: ""

categories: ['frontend']
tags: []

cover:
  image: "/posts/cover.jpg"
  alt: "cover"
  caption: "cover"

# 作者
author: "zhang-shanfeng"

showtoc: true
comments: true

keywords: ["shadcn-vue", "vue", "Sidebar", "Mobile", "auto hidden"]
---

`Shadcn-vue` 的 `Sidebar` 移动端抽屉打开后，点击导航链接不会自动关闭抽屉，需要点击空白区域才关闭，这篇文章就是记录怎么实现点击链接后自动关闭“抽屉”的。

关闭抽屉我们可以调用 `setOpenMobile(false)`，那么方法一就是在每一个 Vue 的路由链接上调用该方法就能实现。

## 方案一（推荐）

方案一就是直接在 `<router-link :to="subItem.to" @click="handleNavLink">` 上用 `@click` 调用触发函数。

代码例子：

```vue
<script setup lang="ts">

import {
    useSidebar
} from "@/components/ui/sidebar";


const { isMobile, setOpenMobile } = useSidebar()

function handleNavLink() {
    if (isMobile) {
        setOpenMobile(false)
    }
}

</script>

<template>
    ...
    <SidebarMenuSubItem v-for="subItem in item.items" :key="subItem.title">
        <SidebarMenuSubButton as-child>
            <router-link :to="subItem.to" @click="handleNavLink">
                <span>{{ subItem.title }}</span>
            </router-link>
        </SidebarMenuSubButton>
    </SidebarMenuSubItem>
    ...
</template>
```

这里只列出了部分代码，注意用与方便阅读，我们在 `<router-link :to="subItem.to" @click="handleNavLink">` 上使用 `vue` 的 `@clink` 来调用 `setOpenMobile()` 执行关闭移动端抽屉。

## 方案二

方案二为使用 Vue 的 watch 监听路由变化，代码如下：

```vue
<script setup lang="ts">
import {
    useSidebar
} from "@/components/ui/sidebar";

import { watch } from 'vue';
import { useRoute } from 'vue-router';


const { isMobile, setOpenMobile } = useSidebar()


const route = useRoute()

watch(
    () => route.path,
    () => {
        if (isMobile) {
            setOpenMobile(false)
        }
    }
)

</script>

<template>
   ...
   <SidebarMenuSubItem v-for="subItem in item.items" :key="subItem.title">
       <SidebarMenuSubButton as-child>
           <router-link :to="subItem.to">
               <span>{{ subItem.title }}</span>
           </router-link>
       </SidebarMenuSubButton>
   </SidebarMenuSubItem>
   ...
</template>

<style lang="scss" scoped></style>

```

这个方法有一个缺点就是点击当前页面的**路由链接**不会自动关闭 <mark>抽屉</mark>，因为路由没有发生变化。正因为如此，所以推荐优先使用方案一。
