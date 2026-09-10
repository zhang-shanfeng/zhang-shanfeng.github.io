---
title: 'Sqlite 外键学习笔记'
date: '2026-09-06T13:22:47+08:00'
draft: true
lastmod: '2026-09-06T13:22:47+08:00'

# 摘要和描述
summary: ""
description: ""

categories: ['notes']
tags: ['SQL', 'SQLite']

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

外键（`Foreign Key`） 是关系数据库中用于建立表与表之间关联的字段，它引用另一张表的主键或唯一键。

它的核心作用有两点：

- 参照完整性：确保子表（从表）中的数据在父表（主表）中必须有对应的有效值。
- 约束数据：防止插入不存在的关联数据，或删除被引用的父表数据。

基本语法：

```sql
CREATE TABLE 子表 (
    列名 数据类型,
    外键列名 数据类型,
    FOREIGN KEY (外键列名) REFERENCES 父表名 (父表主键列)
        ON DELETE 操作
        ON UPDATE 操作
);
```

## 例子

```sql
-- 2. 创建部门表（父表）
CREATE TABLE departments (
    dept_id INTEGER PRIMARY KEY,
    name TEXT NOT NULL
);

-- 3. 创建员工表（子表），dept_id 引用 departments 的 dept_id
CREATE TABLE employees (
    emp_id INTEGER PRIMARY KEY,
    name TEXT NOT NULL,
    dept_id INTEGER,
    FOREIGN KEY (dept_id) REFERENCES departments (dept_id)
        ON DELETE CASCADE
        ON UPDATE CASCADE
);
```

验证外键：

```sql
-- 插入部门
INSERT INTO departments (dept_id, name) VALUES (1, '技术部');
INSERT INTO departments (dept_id, name) VALUES (2, '市场部');

-- 插入员工，dept_id=1 在父表存在 → 成功
INSERT INTO employees (emp_id, name, dept_id) VALUES (101, '张三', 1);

-- 插入员工，dept_id=3 在父表不存在 → 失败！报错：FOREIGN KEY constraint failed
INSERT INTO employees (emp_id, name, dept_id) VALUES (102, '李四', 3);
```

## 外键行为规则

这是最常用且必须掌握的知识点，主要是在创建外键时，通过 ON DELETE 和 ON UPDATE 子句来定义当父表数据变动时，子表该如何响应。

|行为规则|效果说明|最适用的场景|
|---|---|---|
|**`ON DELETE CASCADE`**|父表行被删除时，子表中所有引用该行的记录被**自动删除**。|订单与订单明细：删除订单时，明细自然不再需要。|
|**`ON DELETE SET NULL`**|父表行被删除时，子表中的外键列被**设为 `NULL`**。|员工与部门：部门解散了，员工暂时没有部门，但员工记录保留。|
|**`ON DELETE RESTRICT`/`NO ACTION`**|**阻止删除**。如果子表有引用，父表行无法被删除。这是**默认行为**。|用户与订单：一个有订单的用户不能被删除，保证数据完整。|
|**`ON UPDATE CASCADE`**|父表主键值更新时，子表外键值**同步更新**。|当主键ID需要修改时（虽然很少见），保持关联一致。|

## 查询外键

如果你已经有 employees 表中的外键值（比如 dept_id），想查出这个外键具体对应 departments 表中的哪一行数据，那么这正是 JOIN 的经典用途。

目标：查出 employees 表中每个员工的名字和他/她所属的部门名称。

```sql
SELECT 
    employees.name AS 员工姓名,
    departments.name AS 部门名称
FROM employees
JOIN departments ON employees.dept_id = departments.id;
```

## 反向查询

（已知员工查部门）
```sql
-- 查询员工及其所属部门名称
SELECT employees.name, departments.name
FROM employees
JOIN departments ON employees.dept_id = departments.id;
```

反向查询（已知部门查员工）:

```sql
-- 查询“技术部”下的所有员工姓名
SELECT employees.name
FROM employees
JOIN departments ON employees.dept_id = departments.id
WHERE departments.name = '技术部';
```

或者，你也可以换一种写法，先从部门表出发：

```sql
SELECT employees.name
FROM departments
JOIN employees ON departments.id = employees.dept_id
WHERE departments.name = '技术部';
```
