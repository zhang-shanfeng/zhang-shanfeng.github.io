---
title: 'Kotlin 类委托学习练习'
date: '2026-11-19T22:05:20+08:00'
draft: false
lastmod: '2026-11-19T22:05:20+08:00'

categories: ['Kotlin']
tags: ['Kotlin', '类委托']

cover: "/images/posts/pexels-abdullahoguk-35519304.jpg"

showtoc: true
comments: true

keywords: []
---

学习 `Kotlin` 的类委托，了解类委托的工作原理。

<!--more-->

委托（Delegation）听起来很抽象，有必要了解它的工作原理，下面是委托的本质：

> [!important]
> **「我不亲自干活，我把活交给别人干，但对外我仍声称我会。」**

## 不用委托的例子

下面代码定义了一个 `Logger` 接口，然后我们定义的 `Robot` 类需要用日志：

```kotlin
interface Logger {
    fun log(msg: String)
}

class ConsoleLogger : Logger {
    override fun log(msg: String) = println("[LOG] $msg")
}
```

### 传统的做法，手动转发

`Robot` 类：

```kotlin
class Robot : Logger {
    private val logger = ConsoleLogger()

    override fun log(msg: String) {
        logger.log(msg)   // ← 自己啥也没干，转手给了 logger
    }
}
```

为了使用实现了 `Logger` 接口的  `ConsoleLogger` 类，我们需要给 `Robot` 类添加一个 ConsoleLogger 对象。

 `Robot` 中重写的 log 直接调用了 ConsoleLogger 对象。

用这种方法实现某个接口已实现类代码的方法很繁琐，委托就是解决这个问题的好方法。

## 使用委托

kotlin 中委托用关键字 `by`，下面将 `Robot` 的 Logger 的实现委托给 ConsoleLogger 对象：

```kotlin
class Robot : Logger by ConsoleLogger()
```

> [!tip]
> 注意 `ConsoleLogger` 后面是带 `()` 的，不能缺少。

验证委托方式：

```kotlin
class Robot : Logger by ConsoleLogger()

fun main() {
    val r = Robot()
    r.log("启动")   // 输出: [LOG] 启动  ← 实际是 ConsoleLogger 在打印
}
```

## 委托作为类的构造参数

用构造参数作为委托更灵活，可以给类指定不同的委托对象，来实现不同的实现接口。

```kotlin
class Robot(logger: Logger) : Logger by logger
```

使用：

```kotlin

interface Logger {
    fun log(msg: String)
}

class ConsoleLogger : Logger {
    override fun log(msg: String) = println("控制台：$msg")
}

class FileLogger : Logger {
    override fun log(msg: String) = println("写入文件：$msg")
}

val r1 = Robot(ConsoleLogger())
val r2 = Robot(FileLogger())
```

这样 `Robot` 就拥有了多种 Logger 接口实现，需要什么实现就传入对应的实现对象即可。
