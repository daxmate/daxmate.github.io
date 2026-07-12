---
layout: single
title:  "从零开始学命令行：压缩与归档"
date:   2026-07-12 15:47:00 +0800
categories:
  - Command Line
tags:
  - 命令行
  - tar
  - gzip
  - zip
  - 压缩
  - 归档
---

有没有遇到过这种情况——想发几份文件给别人，一个一个发太麻烦，打包成一个又太大发不动。或者从网上下了一个 `.tar.gz` 文件，不知道怎么解开。

压缩与归档就是干这个的。两个概念先分清：

- **归档**：把一堆文件捆成一个，体积不变，方便传输。比如 `tar`。
- **压缩**：把文件变小，节省空间。比如 `gzip`。

日常往往两个一起用：先归档成一个文件，再压缩——最后得到一个 `.tar.gz`。

---

## 准备工作

```zsh
mkdir -p ~/cmd_practice/compress-demo
cd ~/cmd_practice/compress-demo

# 建几个文件和目录
echo "This is file one" > file1.txt
echo "This is file two" > file2.txt
echo "This is file three" > file3.txt
mkdir subdir
echo "Deep file" > subdir/deep.txt
```

---

## 最常用的组合：`tar`

`tar` 的名字来自 **Tape Archive**——磁带归档。这玩意儿的历史可以追溯到 1970 年代的 Unix Version 7。那时候的存储介质是磁带，你不可能把几百个小文件一个个往磁带上写，所以先把它们捆成一个再写上去。后来压缩功能加上去之后，`tar` 本身就变成了一个能打包还能压缩的瑞士军刀。

不过话说回来，这些早期 Unix 命令的选项设计比较随性——`-czf`、`-xzf` 这种组合，不常用的话根本记不住。所以这篇只是带你认识一下，混个脸熟就行。用的时候回来查一下，完全正常。

### 打包 + 压缩

```zsh
tar -czf myarchive.tar.gz file1.txt file2.txt file3.txt subdir/
```

四个核心选项拆开看：

| 选项 | 含义 |
|------|------|
| `-c` | create，创建归档 |
| `-z` | gzip，用 gzip 压缩 |
| `-f` | file，指定归档文件名（必须放最后） |
| `-v` | verbose，显示处理了哪些文件（可选） |

加 `-v` 能看到过程：

```zsh
tar -czvf myarchive.tar.gz file1.txt file2.txt file3.txt subdir/
```

```text
a file1.txt
a file2.txt
a file3.txt
a subdir
a subdir/deep.txt
```

### 解压

```zsh
tar -xzf myarchive.tar.gz
```

| 选项 | 含义 |
|------|------|
| `-x` | extract，解压 |
| `-z` | gzip 解压 |
| `-f` | 指定文件 |

解压到指定目录：

```zsh
tar -xzf myarchive.tar.gz -C /path/to/dest/
```

`-C` 指定目标目录，得确保这个目录已经存在。

### 只看不拆：`-t`

有时候只想看看压缩包里有什么，不想真的解出来：

```zsh
tar -tzf myarchive.tar.gz
```

```text
file1.txt
file2.txt
file3.txt
subdir/
subdir/deep.txt
```

`-t` 是 list（列出），不拆包。

### 记不住选项？有规律

`tar` 的选项看起来杂乱，其实有规律：

| 动作 | 选项 |
|------|------|
| 打包 | `-c` (create) |
| 拆包 | `-x` (extract) |
| 查看 | `-t` (list) |

配上一个压缩格式（`-z` 是 gzip，`-j` 是 bzip2），再加上 `-f` 指定文件名。

日常最常用的就两句：

```zsh
tar -czf 名字.tar.gz 要打包的文件或目录   # 打包
tar -xzf 名字.tar.gz                     # 拆包
```

---

## 单文件压缩：`gzip` / `gunzip`

如果只需要压缩一个文件，不需要先打包，直接用 `gzip`：

```zsh
gzip file1.txt
```

`file1.txt` 变成了 `file1.txt.gz`，原文件没了。

解压：

```zsh
gunzip file1.txt.gz
```

又变回了 `file1.txt`，`.gz` 文件消失。

保留原文件：

```zsh
gzip -k file1.txt   # -k 就是 keep
```

`gzip` 只处理单个文件。要打包多个文件还得用 `tar`。

---

## 跨平台通用：`zip` / `unzip`

`tar.gz` 在 Unix 世界是老大，但要发给 Windows 用户，`zip` 更省事——Windows 自带支持，不用装额外软件。

```zsh
zip myarchive.zip file1.txt file2.txt file3.txt
```

打包目录要加 `-r`（递归）：

```zsh
zip -r myarchive.zip subdir/
```

解压：

```zsh
unzip myarchive.zip
```

解压到指定目录：

```zsh
unzip myarchive.zip -d /path/to/dest/
```

查看内容（不拆包）：

```zsh
unzip -l myarchive.zip
```

### ZIP 压缩是怎么做到的？

ZIP 背后的故事其实挺有意思。80 年代末有个人叫 Phil Katz，他写了个 PKARC 去兼容当时流行的 ARC 压缩格式。结果被 ARC 的开发者告了，说他侵犯版权。

Katz 一怒之下搞了个自己的格式——**ZIP**，而且把规格完全公开，任何人都可以免费使用。格式公开之后，各种操作系统都开始支持 ZIP，很快就成了跨平台压缩的标配。

ZIP 用的压缩算法叫 **DEFLATE**，核心是两个步骤：

1. **LZ77——找重复**。扫描文件，标记出重复出现的数据段。比如一段文本里"compression"这个词出现了三次，第二次开始就不存完整内容了，而是记"往回找 N 个字节，从这里复制 X 个字节"。就像老师在批作业时发现同一个错误出现多次，第二次开始就写"同上"。

2. **Huffman 编码——高频短码**。统计每个字节出现的频率，出现越多的用越短的编码。这不新鲜——摩尔斯电码就这么干的：最常用的字母 E 是 `.`，不常用的 Q 是 `--.-`。

这两步合在一起就是 DEFLATE——实际上 gzip 也用的同一个算法，只是文件格式不同。ZIP 的容器可以存多个文件，每个文件独立压缩；gzip 只能压单个流，所以常见的是先 tar 归档再 gzip 压缩。

大致记住就好：**自己用或发给 Linux/Mac 用户用 `tar.gz`，发给 Windows 用户或者需要自解压时用 `zip`**。

---

## 压缩率对比：`bzip2`

除了 `gzip`，`bzip2` 压得更狠，但也更慢。在 `tar` 里用 `-j` 代替 `-z`：

```zsh
tar -cjf myarchive.tar.bz2 file1.txt file2.txt
tar -xjf myarchive.tar.bz2
```

日常小文件用 `gzip` 就够了。要极致压缩才考虑 `bzip2`（或者更现代的 `xz`，用 `-J`）。

---

## 动手试试

```zsh
cd ~/cmd_practice/compress-demo

# tar 打包 + 压缩
tar -czf demo.tar.gz file1.txt file2.txt file3.txt subdir/

# 看看包里有什么
tar -tzf demo.tar.gz

# 建一个新目录解压进去
mkdir extracted
tar -xzf demo.tar.gz -C extracted/

# 对比目录
ls extracted/

# zip 操作
zip -r demo.zip file1.txt file2.txt subdir/
unzip -l demo.zip

# 单文件 gzip
gzip -k file1.txt
ls -lh file1.txt*

# 解压单文件
gunzip file1.txt.gz
```

按这个顺序跑一遍，基本就搞清楚了。

---

## 小结

| 命令 | 作用 | 常用写法 |
|------|------|----------|
| `tar -czf` | 打包并 gzip 压缩 | `tar -czf name.tar.gz files/` |
| `tar -xzf` | 解压 tar.gz | `tar -xzf name.tar.gz` |
| `tar -tzf` | 查看 tar.gz 内容 | `tar -tzf name.tar.gz` |
| `gzip` / `gunzip` | 单文件压缩/解压 | `gzip file` → `gunzip file.gz` |
| `zip -r` | 将文件夹打包为 zip | `zip -r name.zip dir/` |
| `unzip` | 解压 zip | `unzip name.zip` |

`tar -czf` 打包，`tar -xzf` 拆包——两句用个十次八次就刻在肌肉里了。

---

> [← 查看系列目录]({% link _pages/series-command-line.md %})
