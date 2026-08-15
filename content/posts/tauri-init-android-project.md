---
title: 'Init a Tauri Android Project'
date: '2026-08-14T23:39:58+08:00'
draft: true
lastmod: '2026-08-14T23:39:58+08:00'

# 摘要和描述
summary: ""
description: ""

categories: ['notes']
tags: ['Tauri', 'Android']

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

在已有的 Tauri 项目中添加 Android 支持怎么做？下面就记录下怎么在一个已有的 Tauri 项目中初始化并允许一个 Android 项目。

首先我们要有一个 Tauri 项目，我们可以用 `pnpm create tauri-app`  命令来创建一个项目，创建后怎么添加 `Android` 支持，其实很简单只要运行初始化命令即可：

```bash
PS D:\Product\Software\sfeekit> pnpm tauri android init
$ tauri "android" "init"
        Info Using installed NDK: E:\AndroidSDK\ndk\30.0.15729638
Generating Android Studio project...
        Info "D:\\Product\\Software\\sfeekit\\src-tauri" relative to "D:\\Product\\Software\\sfeekit\\src-tauri\\gen/android\\app" is "..\\..\\..\\"
victory: Project generated successfully!
    Make cool apps! 🌻 🐕 🎉
```

## 运行 android 

```bash
PS D:\Product\Software\sfeekit> pnpm tauri android dev
$ tauri "android" "dev"
        Info Using installed NDK: E:\AndroidSDK\ndk\30.0.15729638
 (KKG-AN70) with target "aarch64-linux-android" Max
        Info Using 198.18.0.1 to access the development server.
        Info Replacing devUrl host with 198.18.0.1. If your frontend is not listening on that address, try configuring your development server to use the `TAURI_DEV_HOST` environment variable or 0.0.0.0 as host.
     Running BeforeDevCommand (`pnpm dev`)
$ vite
zsf:0.0.0.0
0.0.0.0
android
        Warn Waiting for your frontend dev server to start on http://198.18.0.1:1420/...

  VITE v8.1.5  ready in 3264 ms

  ➜  Local:   http://localhost:1420/
  ➜  Network: http://172.20.240.1:1420/
  ➜  Network: http://192.168.1.29:1420/
```

经过一段时间后在手机上打开了我们的 App，但是显示”网页无法打开“，报错信息：位于 http://tauri.localhost 的网页无法加载，因为：net:ERR_CONNECTION_REFUSED。

运行 `pnpm tauri android dev` 是关闭 ClashVerge 中的虚拟网卡模式，修改 `tauri.conf.json` 中的 `identifier` 标识符为需要的值如： `"identifier": "io.zhangshanfeng.xxx"`

## 修改 WebSocket 连接错误

## 添加 core-js
