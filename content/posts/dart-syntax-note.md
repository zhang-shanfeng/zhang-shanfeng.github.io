---
title: 'Dart Syntax 语法笔记'
date: '2026-08-15T11:30:25+08:00'
draft: true
lastmod: '2026-08-15T11:30:25+08:00'

# 摘要和描述
summary: ""
description: ""

categories: ['notes']
tags: ['Dart']

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

这是一篇 Dart 语言语法笔记，用于时常回顾与加强 Drat 语法的记忆。

## 基础变量类型

```dart
void varTest(){
  var name = 'Voyager I'; // 字符串
  var year = 1977; // 数字
  var antennaDiameter = 3.7; // 浮点数
  var flybyObjects = ['Jupiter', 'Saturn', 'Uranus', 'Neptune']; // 数组
  var image = {
    'tags': ['saturn'],
    'url': '//path/to/saturn.jpg',
  }; // 字典对象
  
  // 流程控制
  if(year >= 2001){
    print('21st century');
  } else if(year >= 1901) {
    print('20th century');
  }
  
  for(final object in flybyObjects){
    print(object);
  }
  
  for(int month=1; month <= 12; month++){
    print(month);
  }
  
  while(year<2016) {
    year+=1;
  }
}
```

## 函数的定义

Dart 的函数定义需要明确返回类型和参数类型，例子：

```dart
int fibonacci(int n) {
  if(n==0 || n==1) return n;
  return fibonacci(n-1) + fibonacci(n-2);
}
```

## class 类定义

```dart
class Spacecraft{
  String name;
  DateTime? launchDate;
  
  // Read-only non-final property
  int? get launchYear => launchDate?.year;
  
  Spacecraft(this.name, this.launchDate) {
    
  }
  
  Spacecraft.unlaunched(String name) : this(name, null);
  
  void describe() {
    print('Spacecraft: $name');
    var launchDate = this.launchDate;
    if(launchDate != null) {
      int years = DateTime.now().difference(launchDate).inDays ~/ 365;
      print('Launched: $launchYear ($years years ago)');
    } else {
      print('Unlaunched');
    }
  }
}
```

## Dot shorthands 点符号速记

https://dart.dev/language/dot-shorthands

点符号是 Dart 提供的一种简写语法，如下：

```dart
// Use dot shorthand syntax on enums:
enum Status { none, running, stopped, paused }

Status currentStatus = .running; // Instead of Status.running

// Use dot shorthand syntax on a static method:
int port = .parse('8080'); // Instead of int.parse('8080')

// Uses dot shorthand syntax on a constructor:
class Point {
  final int x, y;
  Point(this.x, this.y);
  Point.origin() : x = 0, y = 0;
}

Point origin = .origin(); // Instead of Point.origin()
```

只要是编译器能够推断出来的场景都可以使用”点符号“语法。更多请看官方文档。
