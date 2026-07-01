---
layout: single
title:  "从零开始学命令行：文件搜索"
date:   2026-07-01 09:00:00 +0800
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

文件藏在哪个目录了？某个配置文件改过，但忘了在哪了？这个需求太日常了，这篇文章就把搜索文件这件事搞明白。

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

`find` 是文件搜索的瑞士军刀。基本的结构是：

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

`.` 表示当前目录。`-name` 后面跟文件名。它会把**所有匹配的文件路径**列出来，包括子目录里的。

`-name` 是精确匹配文件名，但支持通配符：

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

`d` 是目录，`f` 是普通文件。这是最常用的两个类型。

### 按时间找

`find` 能按照文件的修改时间来找，这在排查问题的时候特别有用。比如「最近 24 小时内改过的文件」：

```zsh
find . -type f -mtime -1
```

`-mtime` 是 modification time（修改时间）。`-1` 表示「1 天以内」；`+7` 表示「7 天以前」；`7` 表示「正好 7 天前」。

更精确的可以用 `-mmin`（按分钟）：

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

`find` 本身只能搜文件名和属性。如果要在文件**内容**里搜关键词，需要配合 `grep`：

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

这个组合在排查配置文件的时候特别实用——你知道某个配置项的值，但不记得写在哪个文件里了。

---

## 闪电搜索：`locate`

`find` 的缺点是慢——它在文件系统里**现场翻找**。如果你要搜整个硬盘，得等一会儿。

`locate` 不一样。它查的是一个**预先建好的索引数据库**，瞬间出结果：

```zsh
locate hello.txt
```

但这个数据库不是实时更新的——macOS 默认每周更新一次。如果文件刚创建，`locate` 可能找不到。

手动更新数据库：

```zsh
sudo /usr/libexec/locate.updatedb
```

跑完之后，`locate` 就能搜到新文件了。

`locate` 适合的场景：知道文件名，搜全盘。`find` 适合：在当前目录或特定目录下，按各种条件精确搜。

---

## macOS 专属：`mdfind`

macOS 有一个藏在命令行下的搜索神器——`mdfind`。它调用的是 Spotlight 的索引引擎，不光能搜文件名，还能搜文件内容、按各种元数据搜：

```zsh
mdfind "hello world"
```

不只是搜到 `hello.txt`，所有包含 "hello world" 这个短语的文档、邮件、笔记，都会列出来。

按文件名搜：

```zsh
mdfind "kMDItemFSName == 'hello.txt'"
```

限定在当前目录：

```zsh
mdfind -onlyin . "hello"
```

`mdfind` 的速度比 `find` 快得多，因为它和 Spotlight 一样用索引。适合快速定位文档。

---

## 更现代的替代品：`fd`

如果觉得 `find` 的语法不够直观，可以装一个叫 `fd` 的工具：

```zsh
brew install fd
```

语法简洁得多：

```zsh
fd hello     # 默认搜文件名，正则匹配，忽略大小写
fd -e txt    # 只搜 .txt 文件
fd hello -x cat {}   # 找到后执行命令
```

`fd` 默认忽略 `.gitignore` 里的文件，而且在 git 仓库里会自动跳过 `.git` 目录。日常用比 `find` 顺手，但 `find` 胜在**哪台 Unix 机器上都有**，不需要额外安装。

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

# mdfind (macOS)
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

初学阶段先练好 `find`——把 `find . -name` 变成肌肉记忆。等用熟了再学 `mdfind` 和 `fd`，日常效率会明显提升。

下一篇聊怎么打包和解压——`tar`、`gzip`、`zip`，把一堆文件压成一个再解开。

---

> [← 查看系列目录]({% link _pages/series-command-line.md %})
