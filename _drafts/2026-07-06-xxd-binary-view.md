---
layout: single
title:  "从零开始学命令行：xxd——用二进制看文件的本来面目"
date:   2026-07-06 12:50:00 +0800
categories:
  - Command Line
tags:
  - 命令行
  - xxd
  - 二进制
  - hexdump
  - 文件格式
---

> 💡 这篇还没写完，先记着几个有意思的发现。

## 起源

随手用 `xxd /bin/ls` 看了一眼，看到了 `cafebabe`——这才知道 macOS 上的命令文件是通用二进制格式。

然后又试了 `xxd /usr/bin/cd`，看到的是一段 shell 脚本——以前一直以为 `cd` 是系统底层的东西，没想到它就是个 Shell 函数。

## 什么是 xxd

`xxd` 是一个命令行工具，用来把文件以十六进制（hex）的形式展示出来。每个字节对应两个十六进制数字，旁边还印着对应的 ASCII 字符，方便对照。

安装：macOS 自带，不需要额外装。

## 基础用法

```zsh
xxd /bin/ls | head -20
```

输出长这样：

```
00000000: cafe babe 0000 0002 0000 000c 0000  ....  ................
00000010: 2b18 0000 0020 0000 0005 0000 0020  +....  ..............
...
```

最左边是文件偏移量，中间是十六进制字节，最右边是 ASCII 对照。

## `cafebabe`——这不是咖啡

看到 `cafebabe` 不要慌。这是 macOS 通用二进制格式（Universal Binary）的魔数（Magic Number）。

- Intel Mac 和 Apple Silicon Mac 的指令集不一样
- 通用二进制就是把两套架构的可执行文件塞到一个文件里
- `cafebabe` 就是标识这种格式的「签名」

用 `file /bin/ls` 确认一下：

```zsh
file /bin/ls
```

输出：

```
/bin/ls: Mach-O universal binary with 2 architectures: [x86_64:Mach-O 64-bit x86_64 executable] [arm64e:Mach-O 64-bit arm64e executable]
```

一个文件里包了两份代码，系统根据当前架构选择加载哪一份。代价是文件体积翻倍——但 macOS 上无所谓，磁盘现在便宜。

## `xxd /usr/bin/cd`——cd 竟然是个脚本？

```zsh
xxd /usr/bin/cd | head -20
```

输出的第一个字节不是 `cafebabe` 之类的魔数，而是 `23`——ASCII 的 `#`，说明这是一个文本文件（shell 脚本）：

```
00000000: 2321 2f62 696e 2f73 680a 2320 2446 7265  #!/bin/sh.# $Fre
00000010: 6542 5344 240a 2320 5468 6973 2066 696c  eBSD$.# This fil
00000020: 6520 6973 2069 6e20 7468 6520 7075 626c  e is in the publ
00000030: 6963 2064 6f6d 6169 6e2e 0a62 7569 6c74  ic domain..built
00000040: 696e 2060 6563 686f 2024 7b30 2323 2a2f  in `echo ${0##*/
00000050: 7d20 7c20 7472 205c 5b3a 7570 7065 723a  } | tr \[:upper:
00000060: 5d20 5c5b 3a6c 6f77 6572 3a5d 6020 247b  ] \[:lower:]` ${
00000070: 312b 2224 4022 7d0a                         1+"$@"}.
```

读一下 ASCII 列——`#!/bin/sh`、`public domain`、`builtin`、`echo`、`tr`。这是一个 POSIX shell 脚本，做的事情是把命令名转成小写，然后调内置命令。

为什么 `cd` 不是一个二进制程序？因为 `cd` 必须是一个 Shell **内置命令**（builtin）——它要改变当前 Shell 进程的工作目录。如果 `cd` 是外部二进制程序，它只能改变自己进程的目录，对调用它的 Shell 没影响。这个文件只是一个**兼容脚本**，给那些写死了 `/usr/bin/cd` 路径的旧脚本用的。

所以 `cd` 其实不是「用 C 写的系统命令」——`cd` 必须是一个 Shell 内置命令，因为它要改变当前 Shell 进程的工作目录。如果是外部命令，它只能改变自己进程的目录，对调用它的 Shell 没影响。

---

## xxd 还能做什么

### 查看图片文件头

```zsh
xxd -l 64 logo.png
```
```
00000000: 8950 4e47 0d0a 1a0a 0000 000d 4948 4452  .PNG........IHDR
```

PNG 文件的魔数是 `8950 4e47`（即 `‰PNG`）。

### 查看文本文件的编码

```zsh
echo "你好" | xxd
```

```
00000000: e4bd a0e5 a5bd 0a                        ........
```

每个汉字占 3 个字节——这是 UTF-8 编码。

### 修改文件内容（危险操作）

```zsh
printf 'hello' > test.txt
xxd test.txt | sed 's/68656c6c6f/48454c4c4f/' | xxd -r > test_upper.txt
```

把文件里的小写 `hello` 改成大写 `HELLO`。`xxd -r` 是反向操作——把 hex dump 转回二进制。

日常不太用得上，但偶尔救急的时候知道有这个功能就够了。

---

## 小结

`xxd` 的核心价值不是「记住它的选项」，而是**让你能用眼睛看到文件的底层结构**。看到一个二进制文件，不再是一片黑箱——至少能读出开头的魔数、区分文本和二进制、知道文件格式是怎么组织起来的。
