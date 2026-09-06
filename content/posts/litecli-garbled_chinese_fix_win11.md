---
title: 'Litecli Win11 查询结果中文乱码解决办法'
date: '2026-09-06T10:44:26+08:00'
draft: false
lastmod: '2026-09-06T10:44:26+08:00'

# 摘要和描述
summary: ""
description: ""

categories: ['notes']
tags: [Litecli]

cover:
  image: "/posts/cover.jpg"
  alt: "cover"
  caption: "cover"

# 作者
author: "zhang-shanfeng"

showtoc: true
comments: true

keywords: [Litecli, 乱码]
---

`Litecli` 是一款用于 `SQLite` 数据库的命令行客户端，具有自动补全和语法高亮功能。

官方仓库：https://github.com/dbcli/litecli

## 安装命令

```bash
pip install -U litecli
```

使用方法和 sqlite3 类似，只是提供了自动补全功能。

## 解决乱码问题

我用的终端是  `PowerShell 7`, 进入 litecli 交互界面后用 SQL 查询结果，中文显示的为乱码，下面是一下没有解决乱码时的命令输出：

```powershell
PS D:\sqlite_learn> chcp
活动代码页: 936
PS D:\sqlite_learn> $OutputEncoding

Preamble          : 
BodyName          : utf-8
EncodingName      : Unicode (UTF-8)
HeaderName        : utf-8
WebName           : utf-8
WindowsCodePage   : 1200
IsBrowserDisplay  : True
IsBrowserSave     : True
IsMailNewsDisplay : True
IsMailNewsSave    : True
IsSingleByte      : False
EncoderFallback   : System.Text.EncoderReplacementFallback
DecoderFallback   : System.Text.DecoderReplacementFallback
IsReadOnly        : True
CodePage          : 65001

PS D:\sqlite_learn> [Console]::InputEncoding

EncodingName      : Chinese Simplified (GB2312)
WebName           : gb2312
HeaderName        : gb2312
BodyName          : gb2312
WindowsCodePage   : 936
IsBrowserDisplay  : True
IsBrowserSave     : True
IsMailNewsDisplay : True
IsMailNewsSave    : True
Preamble          : 
IsSingleByte      : False
EncoderFallback   : System.Text.InternalEncoderBestFitFallback
DecoderFallback   : System.Text.InternalDecoderBestFitFallback
IsReadOnly        : True
CodePage          : 936

PS D:\sqlite_learn> [Console]::OutputEncoding

EncodingName      : Chinese Simplified (GB2312)
WebName           : gb2312
HeaderName        : gb2312
BodyName          : gb2312
WindowsCodePage   : 936
IsBrowserDisplay  : True
IsBrowserSave     : True
IsMailNewsDisplay : True
IsMailNewsSave    : True
Preamble          : 
IsSingleByte      : False
EncoderFallback   : System.Text.InternalEncoderBestFitFallback
DecoderFallback   : System.Text.InternalDecoderBestFitFallback
IsReadOnly        : False
CodePage          : 936
```


可以看到 编码不一致，解决办法是在 `PowerShell 7` 的配置文件 `$PROFILE：` 中添加一下内容：

用 code $PROFILE 打开配置文件，加入一下代码：

```ps
chcp 65001 > $null

[Console]::InputEncoding = [System.Text.UTF8Encoding]::new()
[Console]::OutputEncoding = [System.Text.UTF8Encoding]::new()
```

以后每次启动 PowerShell 7 都会自动执行。

执行以下代码验证：

```bash
chcp
$OutputEncoding
[Console]::InputEncoding
[Console]::OutputEncoding
```

```powershell
PS D:\sqlite_learn> chcp                                                                      
Active code page: 65001                                                                       
PS D:\sqlite_learn> $OutputEncoding                                                           
                                                                                              
Preamble          :                                                                           
BodyName          : utf-8                                                                     
EncodingName      : Unicode (UTF-8)                                                           
HeaderName        : utf-8                                                                     
WebName           : utf-8                                                                     
WindowsCodePage   : 1200                                                                      
IsBrowserDisplay  : True                                                                      
IsBrowserSave     : True
IsMailNewsDisplay : True
IsMailNewsSave    : True
IsSingleByte      : False
EncoderFallback   : System.Text.EncoderReplacementFallback
DecoderFallback   : System.Text.DecoderReplacementFallback
IsReadOnly        : True
CodePage          : 65001

PS D:\sqlite_learn> [Console]::InputEncoding

Preamble          : 
BodyName          : utf-8
EncodingName      : Unicode (UTF-8)
HeaderName        : utf-8
WebName           : utf-8
WindowsCodePage   : 1200
IsBrowserDisplay  : True
IsBrowserSave     : True
IsMailNewsDisplay : True
IsMailNewsSave    : True
IsSingleByte      : False
EncoderFallback   : System.Text.EncoderReplacementFallback
DecoderFallback   : System.Text.DecoderReplacementFallback
IsReadOnly        : False
CodePage          : 65001

PS D:\sqlite_learn> [Console]::OutputEncoding

Preamble          : 
BodyName          : utf-8
EncodingName      : Unicode (UTF-8)
HeaderName        : utf-8
WebName           : utf-8
WindowsCodePage   : 1200
IsBrowserDisplay  : True
IsBrowserSave     : True
IsMailNewsDisplay : True
IsMailNewsSave    : True
IsSingleByte      : False
EncoderFallback   : System.Text.EncoderReplacementFallback
DecoderFallback   : System.Text.DecoderReplacementFallback
IsReadOnly        : False
CodePage          : 65001
```
