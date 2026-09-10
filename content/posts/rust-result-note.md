---
title: 'Rust 中的 Result 与 `?`：main 函数到底应该返回哪个 Result？'
date: '2026-09-10T09:53:54+08:00'
draft: false
lastmod: '2026-09-10T09:53:54+08:00'

# 摘要和描述
summary: ""
description: ""

categories: ['notes']
tags: [Rust, Result]

cover:
  image: "/posts/cover.jpg"
  alt: "cover"
  caption: "Rust Result"

# 作者
author: "zhang-shanfeng"

showtoc: true
comments: true

keywords: [Rust, Result, main]
---

在学习 Rust 的过程中，经常会看到这样的代码：

```rust
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let value = some_function()?;

    println!("{value}");

    Ok(())
}
```

这里有两个值得注意的地方：

* `main` 函数不再是 `fn main()`，而是返回 `Result`
* 函数调用后面直接加了一个 `?`

刚开始接触 Rust 时，很容易把这两个语法联系在一起，但实际上它们解决的是两个不同的问题。

`Result` 用来表示操作可能成功，也可能失败；`?` 则是对 `Result` 进行错误传播的一种简写。

后来在使用 `rusqlite` 时，又会遇到：

```rust
fn main() -> rusqlite::Result<()> {
    // ...
}
```

这就产生了另一个疑问：

> Rust 标准库已经有 `Result` 了，为什么 `rusqlite` 还要提供自己的 `Result`？它们有什么区别？`main` 到底应该使用哪个？

## `?` 是什么？

先从 `?` 开始。

假设有一个函数：

```rust
use std::fs::File;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let file = File::open("hello.txt")?;

    Ok(())
}
```

`File::open()` 返回的是：

```rust
Result<File, std::io::Error>
```

也就是说，它的结果可能是：

```rust
Ok(file)
```

也可能是：

```rust
Err(error)
```

如果不用 `?`，可以使用 `match` 手动处理：

```rust
let file = match File::open("hello.txt") {
    Ok(file) => file,
    Err(error) => {
        return Err(error.into());
    }
};
```

而使用：

```rust
let file = File::open("hello.txt")?;
```

可以简单理解成：

> 如果操作成功，就把 `Ok` 里面的值取出来继续执行；如果失败，就把错误直接从当前函数返回。

因此，`?` 可以理解成一种**错误向上传播**的语法。

这也是为什么 `?` 不能随便出现在一个普通的 `fn main()` 中。

---

## 为什么 `main` 可以返回 `Result`？

如果写成：

```rust
fn main() {
    let file = File::open("hello.txt")?;
}
```

编译器会报错。

原因并不复杂。

`?` 遇到错误时，需要让当前函数返回一个 `Err`。但这里的 `main` 没有返回值：

```rust
fn main()
```

它没有办法返回这个错误。

因此需要让 `main` 本身返回 `Result`：

```rust
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let file = File::open("hello.txt")?;

    Ok(())
}
```

这里：

```rust
Result<(), Box<dyn std::error::Error>>
```

表示：

* 成功时返回 `Ok(())`
* 失败时返回 `Err(...)`

于是 `?` 就有了一个可以把错误传出去的地方。

---

## `Result` 本身并不复杂

标准库中的 `Result` 是：

```rust
std::result::Result<T, E>
```

它大致可以理解为：

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

其中：

* `T` 是成功时的值
* `E` 是失败时的错误

例如：

```rust
Result<String, std::io::Error>
```

表示：

```text
成功 -> String
失败 -> std::io::Error
```

所以 Rust 中的错误处理，本质上还是围绕这个 `Result<T, E>` 展开的。

---

# 那么 `rusqlite::Result` 又是什么？

使用 `rusqlite` 时，经常会看到：

```rust
fn main() -> rusqlite::Result<()> {
    // ...
    Ok(())
}
```

表面上看，似乎 `rusqlite` 自己定义了一个和标准库完全不同的 `Result`。

实际上通常并不是这样。

`rusqlite::Result<T>` 本质上是对标准库 `Result<T, E>` 的一个类型别名，可以理解为：

```rust
type Result<T> = std::result::Result<T, rusqlite::Error>;
```

所以：

```rust
rusqlite::Result<User>
```

实际上就是：

```rust
std::result::Result<User, rusqlite::Error>
```

区别只是 `rusqlite` 已经帮我们把错误类型固定成了：

```rust
rusqlite::Error
```

这样使用 `rusqlite` 的 API 时，就不用每次都重复写：

```rust
Result<T, rusqlite::Error>
```

而是直接写：

```rust
rusqlite::Result<T>
```

代码会简洁很多。

---

# 标准库也有类似的写法

这其实并不是 `rusqlite` 特有的设计。

例如标准库中也有：

```rust
std::io::Result<T>
```

它可以理解为：

```rust
std::result::Result<T, std::io::Error>
```

也就是说：

```rust
std::result::Result<T, E>
```

是通用的 `Result`，而：

```rust
std::io::Result<T>
```

只是把 `E` 固定成了 `std::io::Error`。

同样的思路也会出现在很多第三方库中。

例如：

```rust
rusqlite::Result<T>
serde_json::Result<T>
reqwest::Result<T>
```

这些类型通常都是在标准 `Result` 的基础上，把错误类型固定成各自库的错误类型。

因此看到：

```rust
xxx::Result<T>
```

时，可以先想到：

> 这个库很可能只是提供了一个方便使用的 `Result` 类型别名。

---

# 为什么库需要自己的 `Result`？

最直接的原因就是方便。

假设 `rusqlite` 没有提供自己的 `Result`，那么我们可能需要这样写：

```rust
fn query_user() -> Result<User, rusqlite::Error> {
    // ...
}
```

有了类型别名之后：

```rust
fn query_user() -> rusqlite::Result<User> {
    // ...
}
```

就清楚多了。

尤其是一个库的大部分函数都使用同一种错误类型时，这种写法很自然。

---

# `main` 应该返回哪个 `Result`？

这才是实际编程时真正需要考虑的问题。

其实没有一个规定说：

> `main` 必须返回 `std::result::Result`。

也没有规定：

> `main` 必须返回某个第三方库提供的 `Result`。

真正需要考虑的是：

> **这个 `main` 函数最终可能向外传播什么错误？**

## 只有一个主要错误来源

例如一个简单的 SQLite 程序：

```rust
fn main() -> rusqlite::Result<()> {
    let conn = rusqlite::Connection::open("users.db")?;

    conn.execute(
        "CREATE TABLE users (id INTEGER PRIMARY KEY)",
        [],
    )?;

    Ok(())
}
```

这里主要使用 `rusqlite`，产生的错误也是：

```rust
rusqlite::Error
```

因此直接使用：

```rust
rusqlite::Result<()>
```

非常合适。

把它展开其实就是：

```rust
fn main() -> std::result::Result<(), rusqlite::Error>
```

---

# 如果一个程序使用多个库呢？

情况就会发生变化。

例如：

```rust
fn main() -> rusqlite::Result<()> {
    let conn = rusqlite::Connection::open("users.db")?;

    let content = std::fs::read_to_string("config.toml")?;

    println!("{content}");

    Ok(())
}
```

这里有两个不同的错误来源：

```rust
rusqlite::Connection::open()
```

可能产生：

```rust
rusqlite::Error
```

而：

```rust
std::fs::read_to_string()
```

产生的是：

```rust
std::io::Error
```

但是：

```rust
rusqlite::Result<()>
```

实际上是：

```rust
Result<(), rusqlite::Error>
```

它只能接受 `rusqlite::Error`。

因此，这种情况下直接使用 `rusqlite::Result` 就不太合适了。

---

# `Box<dyn Error>` 就派上用场了

对于一些比较简单的程序，可以让 `main` 返回：

```rust
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let conn = rusqlite::Connection::open("users.db")?;

    let content = std::fs::read_to_string("config.toml")?;

    println!("{content}");

    Ok(())
}
```

这里的：

```rust
Box<dyn std::error::Error>
```

可以理解为：

> 一个能够装下不同错误类型的错误对象。

于是：

```text
rusqlite::Error
        ↓
Box<dyn Error>

std::io::Error
        ↓
Box<dyn Error>

serde_json::Error
        ↓
Box<dyn Error>
```

都可以统一作为 `main` 的错误返回出去。

对于学习项目、小型 CLI 程序来说，这种写法很方便。

---

# 大型项目通常会定义自己的错误类型

随着项目变复杂，如果所有错误最后都写成：

```rust
Box<dyn std::error::Error>
```

虽然方便，但错误的类型信息会变得比较模糊。

例如一个应用可能同时涉及：

```text
文件系统
数据库
网络
配置文件
业务逻辑
```

分别可能产生：

```text
io::Error
rusqlite::Error
reqwest::Error
serde_json::Error
```

这时可以定义自己的应用错误类型：

```rust
enum AppError {
    Io(std::io::Error),
    Database(rusqlite::Error),
    Network(reqwest::Error),
}
```

然后：

```rust
fn main() -> Result<(), AppError> {
    // ...
    Ok(())
}
```

这样整个应用就有了统一的错误类型。

实际项目中还经常使用 `thiserror` 等库来简化自定义错误类型的编写。

---

# `?` 和错误类型之间有什么关系？

到这里，可以把整个关系串起来：

```rust
fn main() -> Result<(), AppError> {
    let value = some_function()?;

    Ok(())
}
```

`some_function()` 返回：

```rust
Result<T, SomeError>
```

如果成功：

```text
Ok(T)
  ↓
继续执行
```

如果失败：

```text
Err(SomeError)
  ↓
?
  ↓
转换成 main 所需要的错误类型
  ↓
return Err(AppError)
```

这也是为什么以后学习 `From` trait 会非常重要。

`?` 并不只是简单地“发现错误就 return”。

当错误类型不完全相同时，它还涉及错误类型之间的转换，而这种转换通常就是通过 `From` 实现完成的。

---

# 最后总结一下

可以把 Rust 中这些 `Result` 的关系理解成：

```text
std::result::Result<T, E>
        │
        │ 通用 Result
        │
        ├── std::io::Result<T>
        │      └── E = std::io::Error
        │
        ├── rusqlite::Result<T>
        │      └── E = rusqlite::Error
        │
        ├── serde_json::Result<T>
        │      └── E = serde_json::Error
        │
        └── reqwest::Result<T>
               └── E = reqwest::Error
```

因此看到库提供的 `Result` 时，不必把它理解成“这个库重新发明了一套 Result”。

很多时候，它只是：

```rust
type Result<T> = std::result::Result<T, LibraryError>;
```

至于 `main` 应该选择哪一种，可以按照下面的思路判断：

```text
只有一个主要错误来源
        ↓
可以直接使用库提供的 Result
例如：
rusqlite::Result<()>


有多个不同错误来源
        ↓
可以考虑：
Result<(), Box<dyn Error>>


项目规模较大，需要明确的错误体系
        ↓
定义自己的 AppError
Result<(), AppError>
```

所以，`main` 并不关心“这个 `Result` 是标准库的还是第三方库的”。

真正重要的是：

> **`Result` 中的错误类型 `E` 是否能够表达这个函数可能发生的错误。**

而 `?` 做的事情，则可以简单概括为：

> **成功就继续，失败就交给上层。**
