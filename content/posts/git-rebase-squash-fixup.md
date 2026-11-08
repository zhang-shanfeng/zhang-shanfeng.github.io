---
title: 'Git Rebase 交互式合并提交完全指南：从 squash 到 fixup'
date: '2026-11-08'
draft: false

categories: ['notes']
tags: ['git', 'rebase']

cover: "/images/posts/pexels-kassiamelox-39970432.jpg"


showtoc: true
comments: true

keywords: ['git', 'rebase', 'squash', 'fixup']
---

介绍 `git` 中 **Rebase** 合并提交命令的使用，以及交互式提交 `-i`怎么用。

<!--more-->

在日常开发中，我们经常会遇到这样的情况：为了修复一个 bug 或者调整一个功能，提交了一堆零碎的 commit。等到要合并到主分支时，这些"WIP"、"修复拼写错误"、"再改一下"的提交记录显得杂乱无章。

这时候，`git rebase -i`（交互式变基）就是你的救星。它能够让你像编辑文档一样，重新整理提交历史，把多个零碎提交合并成一个干净、有意义的提交。本文将带你从零开始掌握这个强大工具。

## 一、Rebase 是什么？

`git rebase` 的字面意思是"重新设定基准"。它的核心作用是**把当前分支的提交"移植"到另一个基准点上**，从而改写提交历史。

与 `git merge` 不同：

- `merge` 会保留完整的分叉历史，产生一个合并提交
- `rebase` 则会把你的提交"接"到目标分支的最新提交之后，形成一条**线性、干净的历史**

而加上 `-i`（interactive，交互式）参数后，Git 会打开一个编辑器，让你逐条决定每个提交的命运：保留、合并、修改、删除、重排……

## 二、准备工作：先理解你的提交历史

在动手之前，先用 `git log --oneline` 看看你要处理的范围：

```bash
$ git log --oneline
e5f6g7h (HEAD -> feature) 修复登录页拼写错误
d4e5f6g 再改一下样式
c3d4e5f 添加登录功能
b2c3d4e 初始化项目
a1b2c3d 第一个提交
```

假设你想把 `c3d4e5f`、`d4e5f6g`、`e5f6g7h` 这三个提交整理成一个干净的"添加登录功能"。

## 三、启动交互式变基

关键命令是：

```bash
git rebase -i <基准提交>
```

这里的 `<基准提交>` 指的是**你不想改动的最早提交**。也就是说，它之后的提交才会进入交互界面。

以上面的例子，要把三个提交合并，基准应该选 `b2c3d4e`：

```bash
git rebase -i b2c3d4e
```

> 💡 **小技巧**：也可以用 `HEAD~3` 表示"最近 3 个提交"，效果一样。

执行后，编辑器会打开，显示类似这样的内容：

```
pick c3d4e5f 添加登录功能
pick d4e5f6g 再改一下样式
pick e5f6g7h 修复登录页拼写错误

# Rebase b2c3d4e..e5f6g7h onto b2c3d4e (3 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup <commit> = like "squash", but discard this commit's log message
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
```

**注意：提交的顺序是从上到下，最早的在最上面。**

## 四、常用命令详解

### 1. `pick`（保留）

默认命令，表示保留这个提交不动。如果不做任何修改，rebase 的结果和原来一样。

### 2. `reword`（改信息）

保留提交内容，但重新编辑提交信息。适合修正写错的 commit message。

```
pick   c3d4e5f 添加登录功能
reword d4e5f6g 再改一下样式    ← 会弹出编辑器让你重写信息
```

### 3. `squash`（合并并保留信息）

把当前提交**合并到上一个提交**，并且**保留两者的提交信息**。Git 会打开编辑器让你整合信息。

```
pick   c3d4e5f 添加登录功能
squash d4e5f6g 再改一下样式
squash e5f6g7h 修复登录页拼写错误
```

执行后，Git 会打开一个编辑器，内容大致是：

```
# This is a combination of 3 commits.
# This is the 1st commit message:

添加登录功能

# This is the commit message #2:

再改一下样式

# This is the commit message #3:

修复登录页拼写错误
```

你可以编辑、删除、整合这些内容，最终留下一个你满意的提交信息。

### 4. `fixup`（合并并丢弃信息）

和 `squash` 类似，但**直接丢弃当前提交的信息**，只保留目标提交的原始信息。适合那些"修个小错"的琐碎提交。

```
pick  c3d4e5f 添加登录功能
fixup d4e5f6g 再改一下样式
fixup e5f6g7h 修复登录页拼写错误
```

执行后，最终的提交信息就是：

```
添加登录功能
```

干净利落，`d4e5f6g` 和 `e5f6g7h` 的信息直接消失。

### 5. `drop`（删除）

删除某个提交。直接删掉那一行，或者写成 `drop` 都行。

```
pick c3d4e5f 添加登录功能
drop d4e5f6g 再改一下样式      ← 这个提交被彻底移除
pick e5f6g7h 修复登录页拼写错误
```

### 6. 调整顺序

想重新排列提交？直接**调整行的顺序**即可。比如把 `e5f6g7h` 提到 `c3d4e5f` 前面：

```
pick e5f6g7h 修复登录页拼写错误
pick c3d4e5f 添加登录功能
pick d4e5f6g 再改一下样式
```

> ⚠️ **注意**：调换顺序可能引发冲突，Git 会暂停 rebase 让你手动解决。

## 五、squash vs fixup：如何选择？

这是本文标题里的核心问题，也是实践中最高频的决策点。

| 特性 | `squash` | `fixup` |
|------|----------|---------|
| 保留提交内容 | ✅ | ✅ |
| 保留提交信息 | ✅（手动整合） | ❌（直接丢弃） |
| 是否打开编辑器 | ✅ | ❌ |
| 适用场景 | 多个提交都有价值，需整合描述 | 后续提交只是琐碎修补 |

**判断法则**：

- 如果后续提交的说明文字**有价值、想合并进去** → 用 `squash`
- 如果后续提交只是"修个小错"，说明文字**无关紧要** → 用 `fixup`

比如："添加登录功能" + "修复拼写错误"，显然拼写错误的说明没有保留价值，用 `fixup` 更合适。

## 六、进阶技巧：`--autosquash`

手动编辑交互界面的提交列表虽然直观，但提交一多就很繁琐。Git 提供了自动化方案。

### 步骤 1：用特殊前缀提交

当你在开发中发现前一个提交有问题，想修复它时，不要直接 `git commit`，而是：

```bash
# 修复提交，稍后想合并进 <commit-hash> 且丢弃自身信息
git commit --fixup=<commit-hash>

# 修复提交，稍后想合并进 <commit-hash> 且保留信息
git commit --squash=<commit-hash>
```

Git 会自动生成 `fixup! 原提交信息` 或 `squash! 原提交信息` 格式的提交信息。

### 步骤 2：自动排序

之后执行：

```bash
git rebase -i --autosquash <基准提交>
```

Git 会**自动识别 `fixup!` 和 `squash!` 前缀**，把它们移动到目标提交后面，并预先标记为 `fixup` 或 `squash`。你只需确认保存即可，效率大大提升。

### 步骤 3：设置默认开启（可选）

如果你觉得这个功能太香，想让每次 `rebase -i` 都默认自动排序：

```bash
git config --global rebase.autoSquash true
```

## 七、冲突处理与中止

Rebase 过程中如果遇到冲突，Git 会暂停并提示你：

```
error: could not apply c3d4e5f...
```

此时按以下步骤处理：

1. **解决冲突**：手动编辑冲突文件
2. **标记为已解决**：`git add <file>`
3. **继续 rebase**：`git rebase --continue`

如果搞砸了，想退出：

```bash
git rebase --abort
```

这会让分支回到 rebase 开始前的状态，一切归零。

## 八、安全须知：改写历史的代价

`rebase` 会**改写提交的 SHA-1 哈希值**，因此：

- ⚠️ **永远不要对已推送到公共分支（如 main/master）的提交执行 rebase**
- ⚠️ 如果 rebase 过的分支已经推送，需要用 `git push --force-with-lease` 强制推送
- ✅ 推荐使用 `--force-with-lease` 而不是 `--force`，它会在远程有别人新提交时拒绝覆盖，更安全

## 九、完整实战示例

假设你的提交历史是：

```
e5f6g7h 修复登录页拼写错误
d4e5f6g 再改一下样式
c3d4e5f 添加登录功能
b2c3d4e 初始化项目
```

你想把上面三个提交整理成两个：一个"添加登录功能"（合并样式修改），一个"修复拼写错误"（保留信息）。

```bash
$ git rebase -i b2c3d4e
```

编辑器打开后修改为：

```
pick   c3d4e5f 添加登录功能
fixup  d4e5f6g 再改一下样式
pick   e5f6g7h 修复登录页拼写错误
```

保存退出，Git 会：

1. 把 `d4e5f6g` 合并进 `c3d4e5f`，丢弃"再改一下样式"的信息
2. 保留 `e5f6g7h` 作为独立提交

最终历史：

```
xxxxxxx 修复登录页拼写错误
yyyyyyy 添加登录功能
b2c3d4e 初始化项目
```

清爽多了！

## 十、速查表

| 需求 | 命令 |
|------|------|
| 交互式合并最近 3 个提交 | `git rebase -i HEAD~3` |
| 合并并保留信息 | 在交互界面用 `squash` |
| 合并并丢弃信息 | 在交互界面用 `fixup` |
| 自动合并 fixup 提交 | `git rebase -i --autosquash <base>` |
| 创建 fixup 提交 | `git commit --fixup=<hash>` |
| 创建 squash 提交 | `git commit --squash=<hash>` |
| 中止 rebase | `git rebase --abort` |
| 继续 rebase | `git rebase --continue` |
| 安全强推 | `git push --force-with-lease` |

## 结语

`git rebase -i` 是 Git 中最强大也最需要小心的工具之一。掌握 `squash` 和 `fixup` 的区别，能让你在整理提交历史时事半功倍：**有价值的描述用 squash 整合，琐碎的修补用 fixup 丢弃**。

再配合 `--autosquash` 的自动化，你可以把"提交-修补-整理"这条流程变得异常顺滑。

最后再强调一遍铁律：**只对尚未共享的本地提交做 rebase**。历史一旦推送出去，就让它保持原样吧。

祝你 rebase 顺利，历史永远干净整洁！
