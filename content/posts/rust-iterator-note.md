---
title: 'Rust Iterator：从理解迭代器到掌握常用方法'
date: '2026-09-10T14:20:04+08:00'
draft: false
lastmod: '2026-09-10T14:20:04+08:00'

# 摘要和描述
summary: ""
description: ""

categories: ['notes']
tags: ['Rust', 'Iterator']

cover:
  image: "/posts/cover.jpg"
  alt: "cover"
  caption: "Rust Iterator"

# 作者
author: "zhang-shanfeng"

showtoc: true
comments: true

keywords: ['Rust', 'Iterator']
---

在学习 Rust 的过程中，`Iterator` 是一个很容易遇到的东西。

刚开始看到：

```rust
let result: Vec<_> = numbers
    .iter()
    .filter(|x| **x % 2 == 0)
    .map(|x| **x * 2)
    .collect();
```

可能会觉得这只是一种比较“花哨”的写法。换成 `for` 循环似乎也完全可以。

但随着代码量增加，会发现 Rust 标准库和第三方库中大量 API 都会返回 `Iterator`。特别是在处理集合、查询数据库、字符串、文件等数据时，迭代器几乎无处不在。

因此，与其把 `map`、`filter`、`collect` 一个个记下来，不如先把 `Iterator` 本身理解清楚。

---

## 一、Iterator 到底是什么？

Rust 中的 `Iterator` 是一个 trait。

它最核心的定义其实并不复杂：

```rust
trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;
}
```

这里最重要的是 `next()`：

```rust
fn next(&mut self) -> Option<Self::Item>;
```

可以先把 Iterator 理解成：

> **一个能够不断产生下一个元素的对象。**

例如：

```rust
let numbers = vec![10, 20, 30];

let mut iter = numbers.iter();

println!("{:?}", iter.next());
println!("{:?}", iter.next());
println!("{:?}", iter.next());
println!("{:?}", iter.next());
```

结果：

```text
Some(10)
Some(20)
Some(30)
None
```

整个过程就是：

```text
第一次 next() → Some(10)
第二次 next() → Some(20)
第三次 next() → Some(30)
第四次 next() → None
```

这其实就是 Iterator 最核心的工作方式。

---

## 二、为什么 `next()` 返回 `Option`？

因为 Iterator 不知道什么时候会结束。

所以 Rust 使用：

```rust
Option<Self::Item>
```

来表示：

```text
Some(value) → 还有元素
None        → 没有元素了
```

这和 Rust 中 `Option` 的设计正好对应。

例如：

```rust
let mut iter = [1, 2, 3].iter();

while let Some(value) = iter.next() {
    println!("{value}");
}
```

这里实际上就是不断调用：

```rust
iter.next()
```

直到返回：

```rust
None
```

所以 `for` 循环并不是什么特殊的“魔法”。

---

## 三、`for` 循环其实也在使用 Iterator

例如：

```rust
let numbers = vec![1, 2, 3];

for number in numbers {
    println!("{number}");
}
```

可以把它理解成 Iterator 在不断调用 `next()`。

大致可以想象成：

```rust
let mut iter = numbers.into_iter();

loop {
    match iter.next() {
        Some(number) => println!("{number}"),
        None => break,
    }
}
```

当然，实际的 `for` 循环还涉及 `IntoIterator` 等机制，但从理解 Iterator 的角度，这样看已经足够了。

所以：

> **Iterator 并不是为了替代 `for` 循环而存在的，它是 Rust 中遍历和处理序列数据的一种基础抽象。**

---

# 四、`iter()`、`iter_mut()` 和 `into_iter()`

这是学习 Iterator 时非常容易混淆的一部分。

它们最大的区别其实和 Rust 的所有权有关。

## 4.1 `iter()`：借用元素

```rust
let numbers = vec![1, 2, 3];

for number in numbers.iter() {
    println!("{number}");
}

println!("{numbers:?}");
```

`iter()` 得到的是：

```text
&Item
```

也就是说，它只是借用集合中的元素。

因此遍历结束以后，`numbers` 仍然可以使用。

---

## 4.2 `iter_mut()`：可变借用元素

如果需要修改集合中的元素，可以使用：

```rust
let mut numbers = vec![1, 2, 3];

for number in numbers.iter_mut() {
    *number *= 2;
}

println!("{numbers:?}");
```

结果：

```text
[2, 4, 6]
```

这里得到的是：

```text
&mut Item
```

因此可以通过解引用修改原来的元素。

---

## 4.3 `into_iter()`：取得元素所有权

```rust
let numbers = vec![1, 2, 3];

for number in numbers.into_iter() {
    println!("{number}");
}
```

这里 Iterator 会取得集合中元素的所有权。

对于：

```rust
Vec<T>
```

来说，迭代得到的是：

```text
T
```

而不是：

```text
&T
```

因此：

```rust
let names = vec![
    String::from("Alice"),
    String::from("Bob"),
];

for name in names.into_iter() {
    println!("{name}");
}
```

遍历之后，`names` 就不能再使用了，因为它的所有权已经被消费掉。

---

## 五、Iterator 的方法可以分成两类

Iterator 方法很多，但没有必要一开始全部背下来。

可以先把它们分成两大类：

```text
Iterator
├── 适配器（Adapter）
│   ├── map
│   ├── filter
│   ├── enumerate
│   ├── zip
│   ├── take
│   └── skip
│
└── 消费器（Consumer）
    ├── collect
    ├── count
    ├── sum
    ├── find
    ├── any
    ├── all
    └── fold
```

一个非常重要的区别是：

> **适配器通常不会立即消费 Iterator，而消费者会真正把 Iterator“跑起来”。**

---

# 六、`map()`：把每个元素转换成另一个值

`map()` 是 Iterator 中最常用的方法之一。

例如：

```rust
let numbers = [1, 2, 3, 4];

let result: Vec<_> = numbers
    .iter()
    .map(|x| x * 2)
    .collect();

println!("{result:?}");
```

结果：

```text
[2, 4, 6, 8]
```

可以把：

```rust
.map(|x| x * 2)
```

理解成：

```text
1 → 2
2 → 4
3 → 6
4 → 8
```

它不会修改原来的集合，而是产生一个新的 Iterator。

---

## `map()` 也可以改变类型

例如：

```rust
let numbers = [1, 2, 3];

let strings: Vec<String> = numbers
    .iter()
    .map(|x| x.to_string())
    .collect();
```

这里：

```text
&integer → String
```

所以 `map()` 不只是“数学上的转换”，更一般地说，它是：

> **把 Iterator 中的每一个元素映射成另一个值。**

---

# 七、`filter()`：筛选元素

`filter()` 用来保留满足条件的元素。

例如：

```rust
let numbers = [1, 2, 3, 4, 5, 6];

let result: Vec<_> = numbers
    .iter()
    .filter(|x| **x % 2 == 0)
    .collect();

println!("{result:?}");
```

结果：

```text
[2, 4, 6]
```

可以理解成：

```text
1 → 丢弃
2 → 保留
3 → 丢弃
4 → 保留
5 → 丢弃
6 → 保留
```

---

# 八、`map()` 和 `filter()` 连起来使用

Iterator 最有意思的地方之一，就是可以把多个操作串起来。

例如：

```rust
let numbers = 1..=10;

let result: Vec<_> = numbers
    .filter(|x| x % 2 == 0)
    .map(|x| x * x)
    .collect();

println!("{result:?}");
```

结果：

```text
[4, 16, 36, 64, 100]
```

执行过程可以理解成：

```text
1..=10
   ↓
filter
   ↓
2 4 6 8 10
   ↓
map
   ↓
4 16 36 64 100
   ↓
collect
   ↓
Vec
```

这也是 Iterator 很适合数据处理的原因。

---

# 九、`collect()`：把 Iterator 收集成集合

`collect()` 也是非常常见的方法。

例如：

```rust
let numbers = 1..=5;

let result: Vec<_> = numbers
    .map(|x| x * 2)
    .collect();
```

得到：

```text
[2, 4, 6, 8, 10]
```

这里需要注意：

```rust
.collect()
```

本身并不能直接告诉 Rust 最终要收集成什么类型。

所以通常需要类型推导提供信息：

```rust
let result: Vec<_> = iterator.collect();
```

或者：

```rust
let result = iterator.collect::<Vec<_>>();
```

两种写法都很常见。

---

# 十、`collect()` 不一定只能生成 `Vec`

例如：

```rust
let numbers = 1..=5;

let result: Vec<i32> = numbers.collect();
```

也可以收集成其他集合：

```rust
let result: std::collections::HashSet<_> =
    [1, 2, 2, 3].into_iter().collect();
```

结果中的重复元素会被去掉。

因此可以把 `collect()` 理解为：

> **把 Iterator 中产生的数据，按照目标类型收集起来。**

具体收集成什么，由目标类型决定。

---

# 十一、`enumerate()`：遍历时同时获得索引

有时候遍历数据时，还需要知道当前是第几个元素。

传统写法可能是：

```rust
let names = ["Alice", "Bob", "Charlie"];

for i in 0..names.len() {
    println!("{}: {}", i, names[i]);
}
```

使用 `enumerate()` 后：

```rust
let names = ["Alice", "Bob", "Charlie"];

for (index, name) in names.iter().enumerate() {
    println!("{index}: {name}");
}
```

结果：

```text
0: Alice
1: Bob
2: Charlie
```

`enumerate()` 会把：

```text
Item
```

转换成：

```text
(index, Item)
```

---

# 十二、`zip()`：把两个 Iterator 配对

例如有两个数组：

```rust
let names = ["Alice", "Bob", "Charlie"];
let scores = [90, 85, 95];
```

可以使用：

```rust
for (name, score) in names.iter().zip(scores.iter()) {
    println!("{name}: {score}");
}
```

结果：

```text
Alice: 90
Bob: 85
Charlie: 95
```

可以把 `zip()` 理解成：

```text
Alice    90
Bob      85
Charlie  95
```

如果两个 Iterator 长度不同，`zip()` 会在较短的那个结束时停止。

---

# 十三、`take()` 和 `skip()`

## `take()`

只取前几个元素：

```rust
let numbers = 1..=100;

let result: Vec<_> = numbers
    .take(5)
    .collect();

println!("{result:?}");
```

结果：

```text
[1, 2, 3, 4, 5]
```

---

## `skip()`

跳过前几个元素：

```rust
let numbers = 1..=5;

let result: Vec<_> = numbers
    .skip(2)
    .collect();

println!("{result:?}");
```

结果：

```text
[3, 4, 5]
```

两者也可以组合：

```rust
let result: Vec<_> = (1..=100)
    .skip(10)
    .take(5)
    .collect();
```

得到：

```text
[11, 12, 13, 14, 15]
```

---

# 十四、`find()`：查找第一个满足条件的元素

例如：

```rust
let numbers = [1, 3, 5, 8, 10];

let result = numbers
    .iter()
    .find(|x| **x % 2 == 0);

println!("{result:?}");
```

结果：

```text
Some(8)
```

如果找不到：

```rust
let numbers = [1, 3, 5];

let result = numbers
    .iter()
    .find(|x| **x % 2 == 0);

println!("{result:?}");
```

结果：

```text
None
```

所以 `find()` 返回：

```rust
Option<Self::Item>
```

这和前面 `next()` 返回 `Option` 的思想是一致的。

---

# 十五、`any()` 和 `all()`

这两个方法适合判断一组数据是否满足某种条件。

## `any()`

只要有一个满足条件，就返回 `true`：

```rust
let numbers = [1, 3, 5, 8];

let has_even = numbers
    .iter()
    .any(|x| *x % 2 == 0);

println!("{has_even}");
```

结果：

```text
true
```

---

## `all()`

必须全部满足条件才返回 `true`：

```rust
let numbers = [2, 4, 6, 8];

let all_even = numbers
    .iter()
    .all(|x| *x % 2 == 0);

println!("{all_even}");
```

结果：

```text
true
```

例如：

```rust
let numbers = [2, 4, 5, 8];

let all_even = numbers
    .iter()
    .all(|x| *x % 2 == 0);
```

结果就是：

```text
false
```

---

# 十六、`count()`：统计元素数量

```rust
let numbers = [10, 20, 30, 40];

let count = numbers
    .iter()
    .count();

println!("{count}");
```

结果：

```text
4
```

也可以结合 `filter()`：

```rust
let count = (1..=100)
    .filter(|x| x % 2 == 0)
    .count();

println!("{count}");
```

结果：

```text
50
```

---

# 十七、`sum()` 和 `product()`

如果 Iterator 中的元素可以进行求和：

```rust
let numbers = [1, 2, 3, 4, 5];

let sum: i32 = numbers
    .iter()
    .copied()
    .sum();

println!("{sum}");
```

结果：

```text
15
```

`product()` 则用于求乘积：

```rust
let numbers = [1, 2, 3, 4, 5];

let product: i32 = numbers
    .iter()
    .copied()
    .product();

println!("{product}");
```

结果：

```text
120
```

这里出现了一个值得注意的小细节：

```rust
.copied()
```

因为 `iter()` 产生的是：

```text
&i32
```

而我们最终想处理的是：

```text
i32
```

对于实现了 `Copy` 的类型，可以使用 `copied()` 把引用转换成值。

---

# 十八、`min()` 和 `max()`

例如：

```rust
let numbers = [5, 2, 8, 1, 9];

let min = numbers.iter().min();
let max = numbers.iter().max();

println!("{min:?}");
println!("{max:?}");
```

结果：

```text
Some(1)
Some(9)
```

注意它们返回的是：

```rust
Option
```

因为 Iterator 可能是空的。

例如：

```rust
let numbers: [i32; 0] = [];

let min = numbers.iter().min();

println!("{min:?}");
```

结果：

```text
None
```

这也是 Rust API 中经常使用 `Option` 的一个典型例子。

---

# 十九、`fold()`：理解 Iterator 的一个关键方法

`fold()` 一开始可能不太容易理解，但它很值得学习。

例如求和：

```rust
let numbers = [1, 2, 3, 4, 5];

let sum = numbers
    .iter()
    .fold(0, |acc, x| acc + x);
```

这里：

```rust
fold(0, ...)
```

中的 `0` 是初始值。

可以把执行过程理解成：

```text
初始值：0

0 + 1 = 1
1 + 2 = 3
3 + 3 = 6
6 + 4 = 10
10 + 5 = 15
```

最终：

```text
15
```

所以 `fold()` 可以理解为：

> **从一个初始值开始，依次使用 Iterator 中的元素更新这个结果。**

---

## `fold()` 不只是用来求和

例如统计字符串长度：

```rust
let words = ["Rust", "Iterator", "Trait"];

let total_length = words
    .iter()
    .fold(0, |total, word| total + word.len());

println!("{total_length}");
```

也可以进行更加复杂的累积计算。

不过在实际代码中，如果已经存在更直接的方法，例如 `sum()`，通常优先使用更直观的方法。

---

# 二十、Iterator 是惰性的

这是 Iterator 中另一个非常重要的概念。

例如：

```rust
let numbers = vec![1, 2, 3];

let iter = numbers
    .iter()
    .map(|x| {
        println!("处理 {x}");
        x * 2
    });
```

仅仅执行到这里，并不会打印：

```text
处理 1
处理 2
处理 3
```

因为：

```rust
map()
```

只是创建了一个新的 Iterator。

真正开始消费它，例如：

```rust
let result: Vec<_> = iter.collect();
```

这时候才会真正执行。

因此可以记住：

> **很多 Iterator 方法是惰性的，只有当 Iterator 被消费时，前面的操作才真正执行。**

例如：

```rust
let result: Vec<_> = numbers
    .iter()
    .filter(|x| **x > 1)
    .map(|x| x * 2)
    .collect();
```

真正的数据处理是在 `collect()` 等消费者调用时发生的。

---

# 二十一、Iterator 方法为什么可以一直链起来？

例如：

```rust
let result: Vec<_> = (1..=10)
    .filter(|x| x % 2 == 0)
    .map(|x| x * 2)
    .filter(|x| *x > 10)
    .collect();
```

每一个适配器都返回另一个 Iterator。

因此可以继续调用：

```text
Iterator
   ↓
filter()
   ↓
Iterator
   ↓
map()
   ↓
Iterator
   ↓
filter()
   ↓
Iterator
   ↓
collect()
   ↓
Vec
```

这就是 Iterator 链式调用的基础。

---

# 二十二、Iterator 和闭包为什么经常一起出现？

如果使用 Iterator，经常会看到：

```rust
.map(|x| x * 2)
```

```rust
.filter(|x| x > 10)
```

```rust
.find(|x| *x == target)
```

这里的：

```rust
|x| x * 2
```

就是 Rust 的闭包。

Iterator 负责：

> **一个一个提供元素。**

闭包负责：

> **告诉 Iterator 每个元素应该怎么处理。**

所以两者组合起来非常自然：

```text
Iterator → 提供数据
Closure  → 定义处理方式
```

这也是为什么学习 Iterator 的过程中，闭包会频繁出现。

---

# 二十三、Iterator 和 `Result` 结合起来

在实际 Rust 项目中，一个非常有价值的场景就是：

```rust
Iterator<Item = Result<T, E>>
```

例如数据库查询。

假设查询结果中的每一行都可能出错，那么 Iterator 可能产生：

```text
Result<Person, Error>
Result<Person, Error>
Result<Person, Error>
...
```

这时可以：

```rust
let persons = stmt
    .query_map([], |row| {
        Ok(Person {
            id: row.get(0)?,
            name: row.get(1)?,
            data: row.get(2)?,
        })
    })?
    .collect::<Result<Vec<Person>>>()?;
```

这里：

```rust
.collect::<Result<Vec<Person>>>()?
```

非常值得注意。

它可以把：

```text
Iterator<Item = Result<Person, Error>>
```

转换成：

```text
Result<Vec<Person>, Error>
```

也就是说：

```text
全部成功
    ↓
Ok(Vec<Person>)

中间出现错误
    ↓
Err(Error)
```

这也是 Rust 中 Iterator 和错误处理结合的一个非常典型的场景。

---

# 二十四、Iterator 与传统 `for` 循环应该怎么选？

并不是用了 Rust Iterator 就不能写 `for`。

实际上，两者都有自己的适用场景。

例如简单遍历：

```rust
for number in numbers {
    println!("{number}");
}
```

非常清楚，也没有必要为了“函数式编程”强行改成：

```rust
numbers
    .iter()
    .for_each(|number| println!("{number}"));
```

如果只是简单执行一些操作，`for` 循环往往更容易阅读。

而这种场景：

```rust
let result: Vec<_> = numbers
    .iter()
    .filter(...)
    .map(...)
    .collect();
```

Iterator 的表达能力就比较强。

因此不应该把它理解成：

```text
Iterator > for
```

更合适的理解是：

```text
简单控制流程       → for
数据转换 / 筛选    → Iterator
复杂流程控制       → for
多个数据处理步骤   → Iterator
```

最终还是以代码的可读性为准。

---

# 二十五、学习 Iterator 时，我认为最应该掌握什么？

Iterator 的方法非常多，不需要一次全部记住。

如果刚开始学习，我会优先掌握下面这些：

### 第一层：核心概念

```text
Iterator
next()
Option
Item
```

先搞清楚：

> Iterator 到底是什么，以及 `next()` 是怎么工作的。

---

### 第二层：三个最常用的方法

```text
iter()
map()
filter()
collect()
```

尤其要理解：

```rust
iter()
    .filter(...)
    .map(...)
    .collect()
```

这种数据处理流程。

---

### 第三层：常用组合

```text
enumerate()
zip()
take()
skip()
find()
any()
all()
count()
sum()
min()
max()
```

这些在实际项目中都很容易遇到。

---

### 第四层：进一步理解

```text
fold()
```

然后继续学习：

```text
Iterator + Closure
Iterator + Option
Iterator + Result
Iterator + Ownership
Iterator + Generic
```

到这个阶段，Iterator 就不只是一个 API，而会逐渐变成 Rust 编程中的一种思维方式。

---

# 二十六、最后再回头看 Iterator trait

最开始看到：

```rust
trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;
}
```

可能觉得它只是标准库里一个普通的 trait。

但把前面的内容串起来之后，会发现它其实连接了很多 Rust 的核心知识：

```text
                    Iterator
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
        Trait       Generic       Option
          │
          ↓
       next()
          │
          ↓
      产生元素
          │
     ┌────┴────┐
     ↓         ↓
   map       filter
     │         │
     └────┬────┘
          ↓
       Iterator
          │
          ↓
       collect
          │
          ↓
       最终结果
```

而在实际代码中，它还会进一步和：

```text
Closure
Ownership
Borrowing
Result
Error Handling
```

联系起来。

这也是为什么我认为 `Iterator` 是 Rust 中非常值得认真学习的 trait。

---

# 总结

如果只是记住几个方法，很容易变成：

```text
map 是干什么的？
filter 是干什么的？
collect 是干什么的？
```

过一段时间可能又忘了。

但如果理解了 Iterator 的核心：

> **Iterator 是一个能够通过 `next()` 逐个产生元素的对象。**

那么后面的很多方法其实就比较容易理解了。

```rust
map()
```

就是改变每个元素。

```rust
filter()
```

就是筛选元素。

```rust
enumerate()
```

就是给元素增加索引。

```rust
zip()
```

就是把两个 Iterator 配对。

```rust
take()
```

和：

```rust
skip()
```

用于控制取哪些元素。

而：

```rust
collect()
sum()
count()
find()
any()
all()
fold()
```

则是在某个阶段真正消费 Iterator，得到最终结果。

所以学习 Iterator 的重点并不是**把所有方法背下来**，而是先建立这样的认识：

```text
数据
 ↓
Iterator
 ↓
一个一个产生元素
 ↓
map / filter / ...
 ↓
继续产生新的 Iterator
 ↓
collect / sum / find / ...
 ↓
最终结果
```

等真正写代码的时候，再根据 IDE 的自动补全去查具体方法，反而更有效。

对 Rust 来说，`Iterator` 值得早一点学，但更重要的是**把它和 Trait、闭包、Option、Result、所有权这些概念联系起来一起理解**。这样以后再看到标准库或者第三方库返回 Iterator，就不会觉得陌生了。
