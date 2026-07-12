---
layout: single
title:  "从零开始学命令行：文件搜索"
date:   2026-07-12 16:16:00 +0800
categories:
  - Command Line
tags:
  - 命令行
  - find
  - locate
  - mdfind
  - 文件搜索
---

文件操作学了一圈，创建、查看、复制、移动、删除都搞定了。但还有一个高频需求没解决——**找文件**。

文件藏在哪个目录了？某个配置文件改过，记不清放哪了？这篇文章就把搜索文件这件事搞明白。

---

## 准备工作

老规矩，先在练习目录里建一些文件来搜：

```zsh
mkdir -p ~/cmd_practice/search-demo/subdir/deep
cd ~/cmd_practice/search-demo

touch readme.txt notes.txt script.sh
echo "hello world" > hello.txt
echo "hello world" > subdir/hello.txt
echo "hello world" > subdir/deep/hello.txt
```

三个 `hello.txt` 散落在不同深度的目录里，后面用来试搜索。

---

## 最精确的搜索：`find`

`find` 是文件搜索的瑞士军刀。它最早由 Dick Haight 在 **1974 年**的 Version 5 Unix 里写出来，和 `cpio`（一个归档工具）是一对搭档。五十多年过去了，`find` 依然是 Unix 世界里最通用的搜索命令。

基本结构是：

```zsh
find 从哪里找 按什么条件找 找到后干什么
```

### 按文件名找

```zsh
find . -name "hello.txt"
```

```
./hello.txt
./subdir/hello.txt
./subdir/deep/hello.txt
```

`.` 表示当前目录。`-name` 后面跟文件名。它会列出**所有匹配的文件路径**，包括子目录里的。

`-name` 支持通配符：

```zsh
find . -name "*.txt"
```

```
./readme.txt
./notes.txt
./hello.txt
./subdir/hello.txt
./subdir/deep/hello.txt
```

所有 `.txt` 文件都找到了。

### 大小写不敏感：`-iname`

有时候不记得文件名是大写还是小写：

```zsh
find . -iname "HELLO.txt"
```

`-iname` 忽略大小写，`HELLO.txt` 和 `hello.txt` 都能匹配。

### 按类型找

只找目录：

```zsh
find . -type d
```

```
.
./subdir
./subdir/deep
```

只找普通文件：

```zsh
find . -type f
```

`d` 表示目录，`f` 表示普通文件。这是最常用的两个类型。

### 按时间找

`find` 能按文件的修改时间来找，排查问题的时候特别好用。比如「最近 24 小时内改过的文件」：

```zsh
find . -type f -mtime -1
```

`-mtime` 是 modification time（修改时间）。`find` 把时间除以 24 小时取整（地板除），数字 `n` 指的是「`n` 个完整 24 小时周期之前」：
  - `-1` 表示不到 1 个周期 = **最近 24 小时内**
  - `+7` 表示超过 7 个周期 = **至少 8 天前**（注意不是 7 天前）
  - `7` 表示恰好过了 7 个周期 = **7~8 天前**这个区间

更精确的用 `-mmin`（按分钟）：

```zsh
find . -type f -mmin -60   # 最近一小时内改过的
```

### 按大小找

```zsh
find . -type f -size +1M   # 大于 1MB 的文件
find . -type f -size -100k  # 小于 100KB 的文件
```

`+` 大于，`-` 小于，不带符号就是精确匹配。

### 按内容搜：`-exec grep`

`find` 本身只能搜文件名和属性。要在文件**内容**里搜关键词，得配合 `grep`：

```zsh
find . -name "*.txt" -exec grep "hello" {} \;
```

```
hello world
hello world
hello world
```

这句的含义：找到所有 `.txt` 文件，对每一个（`{}` 是占位符），执行 `grep "hello"`。`\;` 是结束标记。

如果想同时显示文件名，给 `grep` 加上 `-H`：

```zsh
find . -name "*.txt" -exec grep -H "hello" {} \;
```

```
./hello.txt:hello world
./subdir/hello.txt:hello world
./subdir/deep/hello.txt:hello world
```

也可以用 `+` 代替 `\;`——前者会尽量把多个文件打包传给 `grep` 一次执行，性能更好：

```zsh
find . -name "*.txt" -exec grep -H "hello" {} +
```

效果一样，但跑起来更快。

`find -exec grep` 这个组合在排查配置文件的时候特别实用——你知道某个配置项的值，但不记得写在哪个文件里了。

---

## 闪电搜索：`locate`

`find` 的缺点是慢——它在文件系统里**现场翻找**。要搜整个硬盘，得等一会儿。

`locate` 不一样。它查的是一个**预先建好的索引数据库**，出结果几乎是瞬间的事：

```zsh
locate hello.txt
```

`locate` 是 James A. Woods 在 **1983 年**发明的。那年他在 USENIX 上发了一篇论文叫 *Finding Files Fast*，提出用预建数据库来解决文件查找慢的问题。这个思路后来被 BSD Unix 采纳，4.3BSD-Reno 开始正式以独立的 `locate` 命令发布。

不过代价也很明显——数据库不是实时更新的。macOS 上默认**每周更新一次**（通过 `launchd` 调度），所以刚创建的文件 `locate` 可能找不到。

手动更新数据库：

```zsh
sudo /usr/libexec/locate.updatedb
```

跑完之后，`locate` 就能搜到新文件了。

`locate` 适合的场景：知道文件名，搜全盘。`find` 适合：在当前目录或特定目录下，按各种条件精确搜。

---

## macOS 专属：`mdfind`

macOS 有一个藏在命令行下的搜索神器——`mdfind`。它调用的是 Spotlight 的索引引擎，不光能搜文件名，还能搜文件内容，按各种元数据搜：

```zsh
mdfind "hello world"
```

不只是 `hello.txt`，所有包含 "hello world" 这个短语的文档、邮件、笔记，都会列出来。

按文件名搜：

```zsh
mdfind "kMDItemFSName == 'hello.txt'"
```

限定在当前目录：

```zsh
mdfind -onlyin . "hello"
```

`mdfind` 的速度比 `find` 快得多，因为它和 Spotlight 一样走索引。适合快速定位文档。

---

## 更现代的替代品：`fd`

`find` 的语法确实有点古早味。如果你想要更友好的体验，可以装一个叫 `fd` 的工具：

```zsh
brew install fd
```

语法简洁得多：

```zsh
fd hello     # 默认搜文件名，正则匹配，忽略大小写
fd -e txt    # 只搜 .txt 文件
fd hello -x cat {}   # 找到后执行命令
```

`fd` 是 2017 年流行的 Rust 重写大潮中的一员，默认忽略 `.gitignore` 里的文件，在 git 仓库里也会自动跳过 `.git` 目录。日常用比 `find` 顺手，但 `find` 胜在**哪台 Unix 机器上都有**，不用额外装。

---

## 动手试试

```zsh
cd ~/cmd_practice/search-demo

# 按文件名找
find . -name "hello.txt"
find . -name "*.sh"

# 大小写不敏感
find . -iname "HELLO.TXT"

# 只找目录
find . -type d

# 按时间
touch new-file.txt
find . -type f -mmin -5   # 5 分钟内改过的

# 配合 grep 搜内容
find . -name "*.txt" -exec grep -H "hello" {} \;

# mdfind（macOS）
mdfind -onlyin . "hello"

# locate（如果数据库是新的）
locate hello.txt
```

---

## 小结

| 命令 | 场景 | 特点 |
|------|------|------|
| `find` | 按文件名、类型、时间、大小搜 | 功能最强，到处都有，但慢 |
| `find -exec grep` | 按文件内容搜 | 排查配置文件的利器 |
| `locate` | 全盘按文件名搜 | 快如闪电，但依赖索引更新 |
| `mdfind` | macOS 下搜内容和元数据 | 调用 Spotlight 索引，又快又全 |
| `fd` | 日常快速搜索 | 语法更友好，需要 `brew install` |

总之搜索文件这件事，`find` 是基本功，`locate` 是快速备选，`mdfind` 是 macOS 的隐藏利器，`fd` 则是更现代的替代。多试试就能找到最适合自己工作流的组合。

> [← 查看系列目录]({% link _pages/series-command-line.md %})
