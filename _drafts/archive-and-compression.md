---
layout: single
title:  "从零开始学命令行：压缩与归档"
date:   2026-07-01 14:00:00 +0800
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

有没有遇到过这种情况——想发几份文件给别人，一个一个发太麻烦，打包成一个又太大。或者从网上下了一个 `.tar.gz` 文件，不知道怎么解开。

这就是压缩与归档要解决的问题。两个概念先分清楚：

- **归档**：把一堆文件捆成一个，大小不变，方便传输。比如 `tar`。
- **压缩**：把文件变小，节省空间。比如 `gzip`。

日常操作通常是两个一起用：先归档成一个大文件，再压缩——最后得到一个 `.tar.gz`。

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

`tar` 的名字来自 **Tape Archive**——磁带归档。它的本职是把多个文件和目录打成一个包（`tar` 文件），不压缩。后来添加了压缩功能，所以现在一个命令就能搞定归档 + 压缩。

### 打包 + 压缩

```zsh
tar -czf myarchive.tar.gz file1.txt file2.txt file3.txt subdir/
```

四个选项拆开看：

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

```
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

`-C` 指定目标目录，得先确保这个目录已经存在。

### 只看不拆：`-t`

有时候只想看看压缩包里有什么，不想真的解出来：

```zsh
tar -tzf myarchive.tar.gz
```

```
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

不删原文件的压缩：

```zsh
gzip -k file1.txt   # 保留原文件
```

`gzip` 只处理单个文件。要打包多个文件还得用 `tar`。

---

## 跨平台通用：`zip` / `unzip`

`tar.gz` 是 Unix 世界的老大，但如果要发给 Windows 用户，`zip` 更友好——Windows 系统自带支持，不用装额外软件。

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

日常简单记：**自己用或发给 Linux/Mac 用户用 `tar.gz`，发给 Windows 用户用 `zip`**。

---

## 压缩率对比：`bzip2`

除了 `gzip`，还有一种叫 `bzip2` 的压缩算法，压得更狠，但更慢。在 `tar` 里用 `-j` 代替 `-z`：

```zsh
tar -cjf myarchive.tar.bz2 file1.txt file2.txt
tar -xjf myarchive.tar.bz2
```

日常中小文件用 `gzip` 就够了。只有需要极致压缩的时候才考虑 `bzip2`（或者更现代的 `xz`，用 `-J`）。

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

按照这个顺序跑一遍，基本就搞清楚了。

---

## 小结

| 命令 | 作用 | 常用写法 |
|------|------|----------|
| `tar -czf` | 打包并 gzip 压缩 | `tar -czf name.tar.gz files/` |
| `tar -xzf` | 解压 tar.gz | `tar -xzf name.tar.gz` |
| `tar -tzf` | 查看 tar.gz 内容 | `tar -tzf name.tar.gz` |
| `gzip` / `gunzip` | 单文件压缩/解压 | `gzip file` → `gunzip file.gz` |
| `zip -r` | 打包为 zip | `zip -r name.zip dir/` |
| `unzip` | 解压 zip | `unzip name.zip` |

`tar -czf` 打包，`tar -xzf` 拆包——这两句用个十次八次就刻在肌肉里了。

下一篇聊命令行的一个爽功能——历史记录。敲过的命令不用再敲一遍，`Ctrl+R` 一搜就出来。

---

> [← 查看系列目录]({% link _pages/series-command-line.md %})
