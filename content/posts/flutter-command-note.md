---
title: 'Flutter 开发日常常用命令'
date: '2026-08-15T14:13:17+08:00'
draft: false
lastmod: '2026-08-15T14:13:17+08:00'

# 摘要和描述
summary: ""
description: ""

categories: ['notes']
tags: ['Flutter']

cover:
  image: "/posts/cover.jpg"
  alt: "Flutter"
  caption: "Flutter"

# 作者
author: "zhang-shanfeng"

showtoc: true
comments: true

keywords: []
---

下面是我整理的一份 Flutter 命令使用的文档，涵盖日常开发中所有常用的命令。

## 全局选项

这些选项可以搭配任何命令使用：

```bash
flutter --help              # 查看所有可用命令
flutter --help --verbose    # 查看所有命令（含隐藏选项）
flutter --version           # 查看 Flutter SDK 版本信息
flutter -d <DEVICE_ID>      # 指定目标设备
flutter -v                  # 详细日志输出（调试时有用）
```

## 项目管理

### 创建项目

```bash
flutter create my_app                    # 创建新项目
flutter create --org com.example my_app  # 指定组织名
flutter create . --platforms web         # 为现有项目添加 Web 平台支持
flutter create . --platforms windows     # 为现有项目添加 Windows 平台支持
```

### 清理项目

```bash
flutter clean    # 删除 build/ 和 .dart_tool/ 目录，解决各种诡异构建问题
```

### 代码分析

```bash
flutter analyze                  # 分析整个项目的 Dart 代码
flutter analyze lib/main.dart    # 分析指定文件
```

### 代码格式化

```bash
flutter format lib/              # 格式化指定目录
flutter format lib/main.dart     # 格式化指定文件
```

## 运行与调试

### 运行应用

```bash
flutter run                      # 运行到默认设备（debug 模式）
flutter run -d chrome            # 运行到 Chrome 浏览器
flutter run -d edge              # 运行到 Edge 浏览器
flutter run -d windows           # 运行到 Windows 桌面
flutter run -d android           # 运行到 Android 设备
flutter run -d all               # 同时运行到所有可用设备
flutter run --release            # 以 release 模式运行
flutter run --profile            # 以 profile 模式运行（性能分析）
flutter run -d chrome --wasm     # 使用 WebAssembly 运行 Web 应用
```

## 运行中快捷键

运行 flutter run 后，终端支持以下快捷键：

| 快捷键 | 功能 |
|--------|------|
| `r` | 热重载（Hot Reload） |
| `R` | 热重启（Hot Restart） |
| `q` | 退出运行 |
| `d` | 分离（Detach），应用继续运行但断开调试 |
| `w` | 显示 Widget 树 |
| `t` | 显示渲染树 |
| `p` | 显示平台视图 |

### 附加到运行中的应用

```bash
flutter attach -d <DEVICE_ID>    # 连接到已在运行的 Flutter 应用
```

## 查看日志

```bash
flutter logs                     # 查看运行中应用的日志输出
```

## 截图

```bash
flutter screenshot               # 对连接设备上的应用截图
```

## 构建打包

```bash
flutter build apk                # 构建 Android APK
flutter build appbundle          # 构建 Android App Bundle（推荐上架 Google Play）
flutter build ios                # 构建 iOS 应用
flutter build web                # 构建 Web 应用
flutter build web --wasm         # 使用 WebAssembly 构建 Web 应用
flutter build windows            # 构建 Windows 桌面应用
flutter build linux              # 构建 Linux 桌面应用
flutter build macos              # 构建 macOS 桌面应用
```

## 安装到设备

```bash
flutter install -d <DEVICE_ID>   # 将已构建的应用安装到设备
```

## 依赖管理（pub）

```bash
flutter pub get                  # 下载 pubspec.yaml 中的所有依赖
flutter pub upgrade              # 升级所有依赖到最新版本
flutter pub outdated             # 查看哪些依赖有新版本可用
flutter pub add http             # 添加一个依赖包
flutter pub remove http          # 移除一个依赖包
flutter pub cache repair         # 修复损坏的包缓存
```

## 测试

```bash
flutter test                     # 运行所有单元测试和 Widget 测试
flutter test test/main_test.dart # 运行指定测试文件
flutter drive                    # 运行集成测试（Flutter Driver）
```

## 设备与模拟器

```bash
flutter devices                  # 列出所有已连接的设备
flutter emulators                # 列出所有可用模拟器
flutter emulators --launch <ID>  # 启动指定模拟器
flutter emulators --create       # 创建新模拟器
```

## SDK 管理

```bash
flutter upgrade                  # 升级 Flutter SDK 到最新版本
flutter downgrade                # 降级到当前渠道的上一个版本
flutter channel                  # 查看当前渠道
flutter channel stable           # 切换到 stable 渠道
flutter channel beta             # 切换到 beta 渠道
flutter channel master           # 切换到 master 渠道
flutter precache                 # 预缓存平台工具的二进制文件
```

## 配置

```bash
flutter config --build-dir=<DIR>          # 设置构建输出目录
flutter config --enable-web               # 启用 Web 支持
flutter config --enable-windows-desktop   # 启用 Windows 桌面支持
flutter config --enable-linux-desktop     # 启用 Linux 桌面支持
flutter config --no-analytics             # 关闭匿名数据上报
```

## 国际化

```bash
flutter gen-l10n               # 生成国际化（l10n）本地化文件
```

## 其它

```bash
flutter symbolize --input=<FILE>    # 解析 AOT 编译的堆栈跟踪信息
flutter bash-completion             # 输出 Shell 自动补全脚本
flutter custom-devices list         # 列出自定义设备
```

## 日常开发速查

| 场景 | 命令 |
|------|------|
| 新建项目 | `flutter create my_app` |
| 检查环境 | `flutter doctor` |
| 下载依赖 | `flutter pub get` |
| 运行调试 | `flutter run` |
| 代码检查 | `flutter analyze` |
| 运行测试 | `flutter test` |
| 清理缓存 | `flutter clean` |
| 打包 APK | `flutter build apk` |
| 打包 Web | `flutter build web` |
| 升级 SDK | `flutter upgrade` |

> [!Tip]
> 任何命令都可以通过 `flutter help <命令名>` 查看详细用法，例如 `flutter help run`、`flutter help build`。
