---
title: 'Rust 闭包：从基本语法到 Trait、Iterator 与 Fn/FnMut/FnOnce'
date: '2026-09-11T16:19:15+08:00'
draft: false
lastmod: '2026-09-11T16:19:15+08:00'

# 摘要和描述
summary: ""
description: ""

categories: ['notes',]
tags: ['Rust', 'Closure', 'Trait']

cover:
  image: "/posts/cover.jpg"
  alt: "cover"
  caption: "Rust Closure"

# 作者
author: "zhang-shanfeng"

showtoc: true
comments: true

keywords: ['Rust', 'Closure', 'Trait']
---


最近学习 Rust 的时候，我发现闭包是一个绕不开的东西。

刚开始接触闭包时，感觉它其实没有什么特别的：

```rust
let add = |x, y| x + y;
```

不就是一个没有名字的小函数吗？

但是继续学习 `Iterator`、`Trait` 以及一些实际的 Rust API 后，会发现闭包出现得非常频繁。

比如：

```rust
numbers
    .iter()
    .filter(|x| ...)
    .map(|x| ...)
    .collect();
```

这里的 `filter` 和 `map` 都在使用闭包。

再比如自己设计一个 Trait：

```rust
trait ComponentProcessor {
    fn count_if<F>(&self, predicate: F) -> usize
    where
        F: Fn(&Component) -> bool;
}
```

这时候闭包甚至成为了 API 的一部分。

所以这篇文章不准备单纯罗列闭包的语法，而是结合 `Trait` 和 `Iterator`，把闭包在 Rust 中到底是怎么使用的梳理一下。

---

## 一、闭包到底是什么？

最简单的闭包：

```rust
let add = |x, y| x + y;
```

使用：

```rust
println!("{}", add(1, 2));
```

输出：

```text
3
```

闭包的基本形式可以理解为：

```rust
|参数| 返回值
```

例如：

```rust
let double = |x| x * 2;

let result = double(10);

println!("{result}");
```

也可以写成多行：

```rust
let double = |x| {
    x * 2
};
```

和函数一样，如果最后一个表达式没有分号，它就是返回值：

```rust
let double = |x| {
    x * 2
};
```

如果写成：

```rust
let double = |x| {
    x * 2;
};
```

那么最后得到的就是 `()`。

---

## 二、闭包和普通函数有什么区别？

普通函数：

```rust
fn add(x: i32, y: i32) -> i32 {
    x + y
}
```

闭包：

```rust
let add = |x: i32, y: i32| -> i32 {
    x + y
};
```

表面上看，闭包最大的特点就是写起来比较方便。

但真正重要的区别并不是语法，而是：

> **闭包可以捕获它所在环境中的变量。**

例如：

```rust
let multiplier = 10;

let multiply = |x| x * multiplier;

println!("{}", multiply(5));
```

这里 `multiplier` 并不是闭包的参数：

```rust
|x|
```

但闭包依然可以使用它：

```rust
x * multiplier
```

所以输出：

```text
50
```

这就是闭包非常重要的一个特性。

---

## 三、闭包为什么可以使用外部变量？

可以把闭包想象成一个“带环境的小函数”。

例如：

```rust
let multiplier = 10;

let multiply = |x| x * multiplier;
```

它不只是：

```text
函数
```

还可以理解成：

```text
┌────────────────────┐
│ Closure            │
│                    │
│ multiplier = 10    │
│                    │
│ |x| x * multiplier │
└────────────────────┘
```

因此调用：

```rust
multiply(5)
```

时，它能够访问自己捕获的 `multiplier`。

这也是闭包和普通函数一个非常重要的区别。

---

## 四、闭包捕获变量时会涉及所有权

闭包和 Rust 的所有权系统联系得非常紧密。

例如：

```rust
let name = String::from("Rust");

let print_name = || {
    println!("{name}");
};

print_name();

println!("{name}");
```

这里闭包只是读取 `name`，所以可以理解成借用了它。

如果闭包需要修改外部变量：

```rust
let mut count = 0;

let mut increment = || {
    count += 1;
};

increment();
increment();

println!("{count}");
```

结果：

```text
2
```

这里闭包修改了外部的 `count`。

注意两个 `mut`：

```rust
let mut count = 0;
```

以及：

```rust
let mut increment = || {
    count += 1;
};
```

一个是外部变量需要可变，另一个是闭包本身需要以可变方式调用。

这两个 `mut` 后面理解 `FnMut` 时会再次遇到。

---

## 五、`move`：让闭包取得捕获变量的所有权

闭包默认会根据实际使用情况决定如何捕获外部变量。

如果希望明确让闭包取得变量所有权，可以使用 `move`：

```rust
let name = String::from("Rust");

let print_name = move || {
    println!("{name}");
};
```

这里：

```rust
move
```

表示闭包把捕获的变量移动进自己的环境。

所以：

```rust
let name = String::from("Rust");

let print_name = move || {
    println!("{name}");
};
```

之后，原来的 `name` 就不能再按照原来的方式使用了。

`move` 在多线程、异步编程以及需要让闭包独立拥有数据的场景中非常常见。

---

## 六、为什么 Iterator 中到处都是闭包？

学习 Iterator 的时候会经常看到这样的代码：

```rust
let numbers = [1, 2, 3, 4, 5];

let result: Vec<_> = numbers
    .iter()
    .filter(|x| **x % 2 == 0)
    .map(|x| **x * 2)
    .collect();
```

这里：

```rust
|x| **x % 2 == 0
```

是闭包。

以及：

```rust
|x| **x * 2
```

也是闭包。

可以简单理解成：

```text
Iterator
    ↓
不断提供元素

Closure
    ↓
告诉 Iterator 如何处理元素
```

例如：

```rust
.filter(|x| **x % 2 == 0)
```

相当于告诉 Iterator：

> 给我一个元素，我告诉你它应该保留还是丢弃。

而：

```rust
.map(|x| **x * 2)
```

则是：

> 给我一个元素，我告诉你应该把它转换成什么。

所以 Iterator 和闭包天然适合一起使用。

---

## 七、闭包作为函数参数

闭包真正开始变得有意思，是把它作为参数传给函数。

例如：

```rust
fn calculate<F>(x: i32, f: F) -> i32
where
    F: Fn(i32) -> i32,
{
    f(x)
}
```

使用：

```rust
let result = calculate(10, |x| x * 2);

println!("{result}");
```

结果：

```text
20
```

这里：

```rust
F
```

是泛型类型参数。

而：

```rust
F: Fn(i32) -> i32
```

表示：

> `F` 必须是一个可以接受 `i32`，并返回 `i32` 的闭包。

这也是理解 Rust 闭包非常关键的一步：

```text
闭包
 ↓
本身也是一种类型
 ↓
可以作为泛型参数
 ↓
通过 Fn / FnMut / FnOnce 约束
```

---

## 八、Fn、FnMut、FnOnce 到底是什么？

看到：

```rust
Fn
FnMut
FnOnce
```

一开始可能会觉得这是 Rust 又设计出来的三个复杂概念。

实际上可以先这样理解：

```text
Fn
    可以重复调用，并且调用时不需要修改捕获环境

FnMut
    可以重复调用，并且调用时可能修改捕获环境

FnOnce
    闭包可能在调用过程中消耗捕获的变量，因此只能调用一次
```

可以先不要急着研究底层细节。

先通过实际代码理解它们。

---

## 九、`Fn`：只读取捕获的环境

例如：

```rust
let name = String::from("Rust");

let print_name = || {
    println!("{name}");
};

print_name();
print_name();
```

这个闭包可以重复调用，也没有修改捕获的环境。

所以可以理解为：

```text
Fn
```

例如函数：

```rust
fn call<F>(f: F)
where
    F: Fn(),
{
    f();
    f();
}
```

调用：

```rust
let message = String::from("Hello");

call(|| {
    println!("{message}");
});
```

这里闭包可以被调用多次。

---

## 十、`FnMut`：闭包会修改捕获的环境

例如：

```rust
let mut count = 0;

let mut increment = || {
    count += 1;
};

increment();
increment();

println!("{count}");
```

这里闭包修改了：

```rust
count
```

因此它属于：

```text
FnMut
```

如果把闭包作为函数参数：

```rust
fn call<F>(mut f: F)
where
    F: FnMut(),
{
    f();
    f();
}
```

注意这里：

```rust
mut f
```

是需要的。

因为调用 `FnMut` 闭包本身需要以可变方式访问闭包。

---

## 十一、`FnOnce`：闭包消耗捕获的变量

例如：

```rust
let message = String::from("Hello");

let print_once = move || {
    println!("{message}");
};
```

如果闭包内部把捕获的值消耗掉：

```rust
let message = String::from("Hello");

let print_once = move || {
    drop(message);
};
```

调用一次之后，`message` 已经被消耗。

所以这种闭包属于：

```text
FnOnce
```

可以把三个 Trait 暂时记成：

```text
FnOnce
   ↑
FnMut
   ↑
Fn
```

更准确地说，它们存在继承关系：

```text
Fn: FnMut: FnOnce
```

一个实现 `Fn` 的闭包，也可以作为 `FnMut` 和 `FnOnce` 使用。

但反过来不行。

---

## 十二、为什么 `FnMut` 的参数经常需要写 `mut`？

这是学习闭包时非常容易产生的问题。

例如：

```rust
fn call<F>(mut f: F)
where
    F: FnMut(),
{
    f();
    f();
}
```

为什么：

```rust
f: F
```

不行，而：

```rust
mut f: F
```

可以？

因为 `FnMut` 的调用可能会修改闭包自身的状态。

可以类比普通变量：

```rust
fn foo(x: i32) {
    x += 1;
}
```

不允许。

需要：

```rust
fn foo(mut x: i32) {
    x += 1;
}
```

闭包也是类似的。

```rust
Fn
```

可以通过不可变方式调用。

而：

```rust
FnMut
```

调用时需要可变访问闭包。

所以函数内部通常需要：

```rust
mut f
```

---

## 十三、Trait 中为什么又不需要写 `mut f`？

这里是一个比较容易混淆的地方。

例如：

```rust
trait ComponentProcessor {
    fn modify<F>(&mut self, f: F)
    where
        F: FnMut(&mut Component);
}
```

这里不需要：

```rust
mut f
```

但实现的时候可以写：

```rust
fn modify<F>(&mut self, mut f: F)
where
    F: FnMut(&mut Component),
```

为什么？

因为 Trait 定义的是：

> **这个方法需要什么样的参数和能力。**

而：

```rust
mut
```

并不是类型的一部分。

这两个参数的类型其实都是：

```text
F
```

只是：

```rust
f: F
```

表示函数体内部不以可变绑定使用 `f`。

而：

```rust
mut f: F
```

表示函数体内部允许通过可变绑定使用 `f`。

所以：

```rust
trait ComponentProcessor {
    fn modify<F>(&mut self, f: F)
    where
        F: FnMut(&mut Component);
}
```

完全可以由：

```rust
impl ComponentProcessor for Vec<Component> {
    fn modify<F>(&mut self, mut f: F)
    where
        F: FnMut(&mut Component),
    {
        // ...
    }
}
```

来实现。

Trait 只规定：

```text
F 必须实现 FnMut
```

至于实现方法内部是否需要：

```text
mut f
```

属于具体实现自己的事情。

---

## 十四、用 Trait 做一个实际的闭包练习

为了真正理解这些东西，我给自己设计了一个简单的电子元件处理器。

先定义：

```rust
#[derive(Debug)]
struct Component {
    name: String,
    value: f64,
    price: f64,
}
```

然后设计一个 Trait：

```rust
trait ComponentProcessor {
    fn for_each<F>(&self, f: F)
    where
        F: Fn(&Component);

    fn count_if<F>(&self, predicate: F) -> usize
    where
        F: Fn(&Component) -> bool;

    fn modify<F>(&mut self, f: F)
    where
        F: FnMut(&mut Component);
}
```

这个 Trait 的三个方法分别练习了：

```text
for_each
    → Fn

count_if
    → Fn + 返回 bool

modify
    → FnMut + &mut Component
```

---

## 十五、实现 `for_each`

实现：

```rust
impl ComponentProcessor for Vec<Component> {
    fn for_each<F>(&self, f: F)
    where
        F: Fn(&Component),
    {
        self.iter().for_each(f);
    }

    // ...
}
```

调用：

```rust
components.for_each(|component| {
    println!(
        "{}: value={}, price={}",
        component.name,
        component.value,
        component.price
    );
});
```

这里的关系非常清楚：

```text
&self
 ↓
iter()
 ↓
&Component
 ↓
Fn(&Component)
 ↓
闭包读取数据
```

---

## 十六、实现 `count_if`

实现：

```rust
fn count_if<F>(&self, predicate: F) -> usize
where
    F: Fn(&Component) -> bool,
{
    self.iter()
        .filter(|c| predicate(c))
        .count()
}
```

使用：

```rust
let count = components.count_if(|component| {
    component.price > 0.15
});

println!("价格超过 0.15 的元件数量：{count}");
```

这里闭包：

```rust
|component| component.price > 0.15
```

本质上就是一个判断条件：

```text
Component
    ↓
Closure
    ↓
true / false
```

这种形式在实际代码中非常常见。

---

## 十七、实现 `modify`

这里是这次练习中最值得研究的一部分。

实现：

```rust
fn modify<F>(&mut self, mut f: F)
where
    F: FnMut(&mut Component),
{
    for mut item in self {
        f(&mut item);
    }
}
```

调用：

```rust
components.modify(|component| {
    component.price *= 1.1;
});

println!("{:#?}", components);
```

实测之后会发现：

> **原来的 `components` 中 `price` 确实发生了变化。**

这一点非常重要。

---

## 十八、为什么 `for mut item in self` 真的可以修改原来的元素？

这里需要仔细看类型。

在：

```rust
fn modify<F>(&mut self, mut f: F)
```

中：

```rust
self
```

是：

```text
&mut Vec<Component>
```

而：

```rust
for item in self
```

会通过 `IntoIterator` 对 `&mut Vec<Component>` 进行迭代。

得到的元素实际上是：

```text
&mut Component
```

所以可以理解成：

```rust
for item in self {
    // item: &mut Component
}
```

这也是为什么：

```rust
f(&mut item);
```

最终能够修改原来的 `Component`。

这里容易误解的一点是：

```rust
mut item
```

并不是把 `Component` 复制出来以后再修改一个无关的副本。

`item` 本身是指向 Vec 元素的可变引用：

```text
Vec
│
├── Component
├── Component ← item 指向这里
└── Component
```

所以通过这个可变引用修改：

```rust
component.price *= 1.1;
```

最终修改的就是 Vec 中真正存储的元素。

---

## 十九、不过这里其实可以写得更直接

虽然你的代码：

```rust
fn modify<F>(&mut self, mut f: F)
where
    F: FnMut(&mut Component),
{
    for mut item in self {
        f(&mut item);
    }
}
```

确实可以工作，但这里的：

```rust
mut item
```

和：

```rust
&mut item
```

并不是最直观的写法。

更推荐写成：

```rust
fn modify<F>(&mut self, mut f: F)
where
    F: FnMut(&mut Component),
{
    for item in self.iter_mut() {
        f(item);
    }
}
```

这样类型关系一眼就能看出来：

```text
self.iter_mut()
      ↓
&mut Component
      ↓
f(item)
      ↓
FnMut(&mut Component)
```

甚至可以直接使用 Iterator：

```rust
fn modify<F>(&mut self, mut f: F)
where
    F: FnMut(&mut Component),
{
    self.iter_mut().for_each(f);
}
```

我更推荐这个版本。

不是因为你原来的实现“错了”，而是因为 `iter_mut()` 更明确地表达了代码的意图：

> 我要遍历 Vec 中的元素，并取得每一个元素的可变引用。

---

## 二十、这里的 `mut item` 到底有什么作用？

如果写：

```rust
for item in self {
    f(&mut item);
}
```

通常会遇到借用相关的问题，因为：

```rust
&mut item
```

要求 `item` 这个绑定本身能够被可变借用。

所以你写：

```rust
for mut item in self {
```

以后：

```rust
&mut item
```

才可以成立。

但这里的 `mut` 是让：

```text
item 这个“引用变量”
```

本身可以被可变借用。

而：

```text
item
```

本身仍然指向 Vec 中的那个 `Component`。

所以不要把：

```rust
mut item
```

理解成：

> 把 Component 复制出来变成一个新的可变 Component。

它实际涉及的是：

```text
item: &mut Component
```

这个可变引用的再借用。

---

## 二十一、为什么 `iter_mut()` 更适合这个场景？

比较下面两种写法。

第一种：

```rust
for mut item in self {
    f(&mut item);
}
```

需要脑补：

```text
self
 ↓
&mut Vec<Component>
 ↓
IntoIterator
 ↓
&mut Component
 ↓
mut item
 ↓
&mut item
 ↓
再借用
```

第二种：

```rust
for item in self.iter_mut() {
    f(item);
}
```

就非常直接：

```text
self
 ↓
iter_mut()
 ↓
&mut Component
 ↓
f(item)
```

因此学习阶段我更建议第二种。

第一种可以作为一个很好的练习：

> 为什么它居然也能工作？

但写实际项目代码时，通常应该优先选择让意图更明显的写法。

---

## 二十二、这个练习真正学到的东西

这个小小的 `ComponentProcessor` 实际上把很多 Rust 知识串在了一起。

```text
                    ComponentProcessor
                           │
             ┌─────────────┼─────────────┐
             ↓             ↓             ↓
          for_each       count_if       modify
             │             │             │
             ↓             ↓             ↓
             Fn            Fn          FnMut
             │             │             │
             ↓             ↓             ↓
       &Component    &Component    &mut Component
             │             │             │
             └─────────────┼─────────────┘
                           ↓
                       Closure
                           ↓
                       Iterator
```

继续往下，又会碰到：

```text
Closure
   │
   ├── 捕获变量
   │
   ├── Ownership
   │
   ├── Borrowing
   │
   ├── move
   │
   ├── Fn
   ├── FnMut
   └── FnOnce
```

所以闭包并不是一个孤立的语法点。

它实际上是 Rust 中连接很多知识的重要一环。

---

## 二十三、学习闭包时不应该只背 `Fn / FnMut / FnOnce`

如果只是记：

```text
Fn      → 可以调用
FnMut   → 可以修改
FnOnce  → 只能调用一次
```

其实很容易学得很机械。

更好的方法是看闭包到底**捕获了什么，以及怎么使用这些捕获的变量**。

例如：

```rust
let x = 10;

let f = || {
    println!("{x}");
};
```

这里主要是读取。

再看：

```rust
let mut x = 10;

let mut f = || {
    x += 1;
};
```

这里修改了捕获的环境。

再看：

```rust
let x = String::from("Rust");

let f = move || {
    drop(x);
};
```

这里则是消耗捕获的变量。

从这三个例子出发，再去理解：

```text
Fn
FnMut
FnOnce
```

会自然很多。

---

## 二十四、闭包到底应该什么时候重点学习？

我觉得 Rust 的闭包不需要单独花很长时间学完所有细节。

比较适合的方式是：

```text
先学基本语法
      ↓
理解捕获变量
      ↓
学习 Iterator
      ↓
在 map/filter/find 中大量使用闭包
      ↓
遇到 Fn / FnMut / FnOnce
      ↓
再深入理解
      ↓
结合 Trait 自己设计 API
```

尤其是：

```rust
.iter()
.map(...)
.filter(...)
.collect()
```

这一类代码非常适合作为练习。

因为你可以同时理解：

```text
Iterator 做什么
Closure 做什么
Trait 做什么
Ownership 如何影响 Closure
```

---

## 二十五、最后总结一下

如果现在让我用一句话描述 Rust 闭包，我会这样理解：

> **闭包就是可以捕获所在环境中变量的一段可调用代码，而 Rust 又把这种“可调用的东西”抽象成了 `Fn`、`FnMut`、`FnOnce` 等 Trait。**

所以看到：

```rust
numbers.iter().map(|x| x * 2)
```

不要只理解成：

> `map` 里面写了一个 `|x| x * 2`。

而应该慢慢形成这样的认识：

```text
Iterator
    ↓
需要一个“处理元素的行为”
    ↓
传入 Closure
    ↓
Closure 是一种可调用的类型
    ↓
通过 Fn / FnMut / FnOnce 描述它的调用方式
```

而在自己设计 API 时，也会自然出现这种代码：

```rust
fn modify<F>(&mut self, mut f: F)
where
    F: FnMut(&mut Component),
```

这时候闭包就不再是一个“语法糖”，而真正成为 Rust 类型系统的一部分。

对目前的学习阶段来说，我觉得闭包最值得掌握的并不是几十种写法，而是下面这条线：

```text
闭包基本语法
    ↓
捕获外部变量
    ↓
move
    ↓
Fn / FnMut / FnOnce
    ↓
Iterator + Closure
    ↓
Trait + Closure
    ↓
自己设计接受闭包的 API
```

走完这条线之后，再去看 Rust 标准库里那些大量使用闭包的代码，就会容易很多。

而这也是我目前学习 Rust 时比较明显的一个感受：**很多 Rust 知识单独看起来零散，但一旦把 Trait、闭包、Iterator、所有权和借用放到一起，很多之前看不懂的代码突然就能串起来了。**
