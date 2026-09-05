---
title: 'Windows 下 SQLite 学习快速入门'
date: '2026-09-05T21:00:47+08:00'
draft: false
lastmod: '2026-09-05T21:00:47+08:00'

# 摘要和描述
summary: ""
description: ""

categories: ['notes',]
tags: ['SQLite']

cover:
  image: "/posts/cover.jpg"
  alt: "cover"
  caption: "cover"

# 作者
author: "zhang-shanfeng"

showtoc: true
comments: true

keywords: ['Windows', 'SQLite', '数据库']
---

记录一下在 Windows 下安装搭建 SQLite 的学习环境，包括下载安装、使用 GUI 管理数据库和基础的使用练习。

## 下载安装

第一步先安装 SQLite，去官网下载二进制预编译文件：https://sqlite.org/download.html

分别下载 `Precompiled Binaries for Windows` 下面的 `sqlite-dll-win-x64-3530400.zip` 和 `sqlite-tools-win-x64-3530400.zip` 将两者分别解压后放入自定义目录中，如：d:\sqlite,目录结果如下：

```bash
PS E:\bin\sqlite> ls

    Directory: E:\bin\sqlite

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a---            1970/1/1     8:00        3243520 sqldiff.exe
-a---            1970/1/1     8:00        4484608 sqlite3_analyzer.exe
-a---            1970/1/1     8:00        3179520 sqlite3_rsync.exe
-a---           2026/7/25     3:33           8722 sqlite3.def
-a---           2026/7/25     3:33        3285504 sqlite3.dll
-a---            1970/1/1     8:00        4022272 sqlite3.exe
```

将 `d:\sqlite` 目录添加到 `Path` 环境变量中，就可以在终端中使用 `sqlite3` 命令来学习了。

## 基础命令

`SQLite` 的命令行工具 `sqlite3` 主要通过 **两类** 命令进行操作：**普通 SQL** 语句和以点 **(.) 开头**的元命令。前者用于处理数据，后者则用于管理数据库和格式化输出。

### 启动和退出命令

- `sqlite3 test.db` ：打开或创建数据库文件，如果文件不存在则为创建数据库文件。
- `sqlite3` ：不指定文件名，则创建一个临时的内存数据库，退出即消失。
- `.exit` 或 `.quit` ：退出 `sqlite3` 交互界面。

### 数据库与表管理

这些元命令用于查看和修改数据库结构。

- `.databases` : 列出当前会话中所有附加的数据库及其文件路径
- `.tables` : 显示当前数据库中的所有表
- `.schema 表名` : 显示表或整个数据库的 CREATE 语句（表结构）
- `.dump 表名` : 将数据库或特定表导出为 SQL 文本格式的脚本，用于备份
- `.read 文件名` : 执行指定文件中的 SQL 语句，常用于导入备份或运行脚本
- `.backup 数据库名 文件名` : 将数据库（默认为 main）备份到指定文件

### 查询输出格式

这些命令控制查询结果在屏幕上的显示方式，非常实用。

- `.mode 模式` : 设置查询结果的输出模式，如：`csv`, `column`, `json`, `list` 等
- `.headers on/off` : 控制是否显示查询结果的列名, 如：`on` 时显示，`off` 时隐藏
- `.nullvalue <字符串>` : 设置查询结果中 NULL 值的显示文本, 如：`.nullvalue NULL`
- `.output <文件名>` : 将后续所有查询结果重定向输出到指定文件，而非屏幕, 如：`.output result.txt`
- `.once <文件名>` : 仅将下一次查询的结果输出到指定文件，如：`.once report.csv`

### 数据导入导出

配合输出格式，这些命令可以进行数据迁移。

- `.import <文件名> <表名>` : 将数据文件（如 CSV）导入到指定表中。通常需先设置 .mode csv，如：`.import data.csv users`
- `.output stdout` : 将输出目标重新指回屏幕（标准输出），停止写入文件，如：`sqlite> .output stdout`


## SQL 基础

### 创建数据表

```sql
CREATE TABLE
    user_profile (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        user_name TEXT UNIQUE,
        age INTEGER NOT NULL,
        gender TEXT CHECK (gender IN ('M', 'F', 'X', 'N'))
    );
```

### 插入数据

```sql
INSERT INTO
    user_profile (user_name, age, gender)
VALUES
    ('张志龙', 27, 'M');
```

### 更新一个字段

基础语法：

`UPDATE 表名 SET 字段名 = 新值 WHERE 条件;`

```sql
sqlite> UPDATE user_profile SET user_name = "张三丰" WHERE id = 1;
```

#### 更新多个字段：

```sql
UPDATE user_profile
SET
    user_name = "张无忌",
    age = 38
WHERE
    id = 1;

-- 所有用户的年龄加 1
UPDATE user_profile
SET
    age = age + 1;
```

查询结果：

```bash
sqlite> .read queries/query.sql
1|张无忌_vip|41|M
```
在用户名的后面加上了 `_vip` 后缀。

#### 字符串拼接

```sql
UPDATE user_profile
SET
    user_name = user_name || '_vip'
WHERE
    id = 1;
```

#### 最重要的安全提醒

```sql
-- ❌ 危险！没有 WHERE 条件会更新整张表
UPDATE users SET age = 26;  -- 所有用户的年龄都变成 26！

-- ✅ 安全！必须加 WHERE 条件
UPDATE users SET age = 26 WHERE id = 1;
```

#### 实际使用步骤

1. 先查询确认要更新的数据
2. 执行更新
3. 验证更新结果

### 查询语句

查询指定字段：

```sql
SELECT
    user_name,
    age,
    gender
FROM
    user_profile;
```

重命名指定查询列：

```sql
SELECT
    user_name as 用户名,
    age as 年龄,
    gender as 性别
FROM
    user_profile;
```

### 删除语句

```sql
DELETE FROM user_profile
WHERE
    id = 4;
```

### 新增字段

```sql
ALTER TABLE user_profile
ADD COLUMN site TEXT DEFAULT 'None';
```

增加了 `site` 网站字段，并设置默认值为 `None`。

## 配置文件

.sqliterc 文件为配置文件，放到用户根目录 `~/`

```
.headers on # 显示表头
.mode column # 列对齐，查询时显示比较好看
```
