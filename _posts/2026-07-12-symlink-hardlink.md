---
layout: single
title:  "从零开始学命令行：软链接和硬链接——文件的两个分身"
date:   2026-07-12 13:00:00 +0800
categories:
  - Command Line
tags:
  - 命令行
  - ln
  - 软链接
  - 硬链接
  - 文件系统
---

文件系统里有一个很容易搞混的概念：**链接**。我们在文件权限那篇提到过 `ls -l` 输出里会看到链接数，现在就来把这个彻底搞清楚。

链接分两种：
- **硬链接（hard link）**——同一个文件的不同名字
- **软链接（symbolic link / symlink）**——指向另一个文件的快捷方式

---

## 先搞清楚文件是怎么存的

在理解链接之前，得先知道文件系统是怎么存文件的。简化一下：

```text
文件名 → inode（索引节点） → 磁盘上的数据
```

- **文件名**是你看到的，比如 `readme.md`
- **inode** 是文件的"身份证"，存了权限、所有者、大小、时间戳这些元数据
- **数据块**是文件实际内容存放的地方

`ls -i` 能看到文件的 inode 编号：

```zsh
ls -i readme.md
```

```text
12345678 readme.md
```

创建文件的时候，系统做了一件事：**给文件名分配一个 inode，inode 指向数据块**。

链接就是在这个关系上做文章。

---

## 硬链接：同一个 inode，多个名字

```zsh
echo "hello" > original.txt
ln original.txt hardlink.txt
ls -i original.txt hardlink.txt
```

```text
12345678 original.txt
12345678 hardlink.txt
```

两个文件名指向**同一个 inode**。也就是说，它们是同一个文件的两个名字。

修改其中一个：

```zsh
echo "world" >> hardlink.txt
cat original.txt
```

```text
hello
world
```

改一个，另一个也会变——因为本来就是同一个文件。

删除呢？

```zsh
rm original.txt
cat hardlink.txt
```

```text
hello
world
```

文件还在。硬链接的原理是：**每个 inode 里有一个引用计数**，记录有多少个文件名指向它。`rm` 只是把计数减 1，计数变成 0 的时候才真正删掉数据。

```zsh
ls -l hardlink.txt
```

输出里第二列的数字就是链接数。

**硬链接的限制：**
- 不能跨文件系统（不同分区/硬盘不行，因为 inode 是分区独立的）
- 不能链接目录（防止循环引用）

---

## 软链接：指向路径的快捷方式

```zsh
echo "hello" > original.txt
ln -s original.txt softlink.txt
ls -li
```

```text
12345678 -rw-r--r--  1 dax  staff  6 Jul 10 12:00 original.txt
12345679 lrwxr-xr-x  1 dax  staff  11 Jul 10 12:00 softlink.txt -> original.txt
```

软链接有自己的 inode（12345679），它的内容就是一个路径字符串 `original.txt`。

**权限那列的开头是 `l`**（link），表示这是一个链接。后面的权限 `rwxr-xr-x` 没有实际意义——软链接的权限由目标文件决定。

读取软链接的时候，系统自动跳转到目标文件：

```zsh
cat softlink.txt
```

```text
hello
```

但删掉原文件后：

```zsh
rm original.txt
cat softlink.txt
```

```text
cat: softlink.txt: No such file or directory
```

软链接断了——变成了一个"死链接"（dangling symlink）。

`ls` 能看到它，但 `cat`、`open` 这些操作都会失败。

**软链接的优点：**
- 可以跨文件系统
- 可以链接目录
- 没有硬链接的限制

---

## 对比

| | 硬链接 | 软链接 |
|------|------|------|
| 本质 | 多个名字指向同一个 inode | 一个指向路径的特殊文件 |
| `ls -l` 显示 | 普通文件 | `l` 开头，`->` 指向目标 |
| 跨文件系统 | ❌ | ✅ |
| 链接目录 | ❌ | ✅ |
| 删除原文件后 | 文件还在 | 链接失效 |
| 修改同步 | ✅ | ✅ |

---

## 日常怎么用

**软链接最常见：**

```zsh
# 把配置目录链接到方便的位置
ln -s ~/Dropbox/config/nvim ~/.config/nvim

# 版本切换
ln -sf ~/tools/node-v18/bin/node ~/local/bin/node
```

`-f` 是 force，如果目标已存在就覆盖。

**硬链接在备份场景有用：**

```zsh
# 用 cp -l 创建硬链接而不是复制数据
cp -l file1.txt backup/file1.txt
```

省空间，因为不复制数据。

---

## 动手试试

```zsh
mkdir -p ~/cmd_practice/links
cd ~/cmd_practice/links

# 硬链接
echo "这是原始文件" > original.txt
ln original.txt hard.txt
ls -li
cat hard.txt

# 删除原始文件
rm original.txt
cat hard.txt    # 还在
ls -li          # hard.txt 还在

# 软链接
echo "另一个文件" > source.txt
ln -s source.txt sym.txt
ls -li

# 断链
rm source.txt
cat sym.txt     # 报错，死链了
```

---

## 小结

- **硬链接** = 同一个文件的不同名字，删一个不影响另一个
- **软链接** = 指向路径的快捷方式，原文件没了就失效
- `ln` 不带参数创建硬链接，`ln -s` 创建软链接
- `ls -li` 看 inode 编号，`ls -l` 看链接类型

硬链接在日常操作中很少直接用，但理解它能帮你搞懂文件系统的底层机制。软链接天天用——管理配置、版本切换、组织文件都离不了它。

---

> [← 查看系列目录]({% link _pages/series-command-line.md %})
