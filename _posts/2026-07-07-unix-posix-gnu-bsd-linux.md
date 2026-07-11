---
layout: single
title:  "从零开始学命令行：Unix、POSIX、GNU、BSD、Linux——这些名字到底在说什么"
date:   2026-07-07 14:00:00 +0800
categories:
  - Command Line
tags:
  - 命令行
  - Unix
  - POSIX
  - GNU
  - BSD
  - Linux
  - 历史
---

用过命令行一段时间之后，肯定会碰到这些词：Unix、POSIX、GNU、BSD、Linux。它们经常出现在各种地方——软件介绍里、文档里、论坛回复里。

但这些名字到底是什么意思？它们之间是什么关系？

这篇就来把它们理清楚。

---

## Unix：一切的起点

时间回到 1969 年，美国贝尔实验室。Ken Thompson 和 Dennis Ritchie 写了一个操作系统，取名叫 Unix。

今天的眼光看，那个系统简陋得不行。但它确立了一些**至今仍在用**的设计思想，比如：

- 一切皆文件
- 每个程序只做一件事，做好它
- 用文本作为程序间通信的接口

这些思想后来被称为"Unix 哲学"。

Unix 一开始是用汇编写的，后来 Dennis Ritchie 发明了 C 语言，然后用 C 重写了 Unix。这一步很关键——用 C 写的 Unix 可以轻松移植到不同的硬件上，不用从头重写。

Unix 在 70、80 年代在学术界和工业界广泛使用。但后来 AT&T 收回了 Unix 的版权，不再允许自由分发。封闭导致了两件事：催生了自由软件运动，也催生了 BSD。

```
1969: Unix 诞生 (贝尔实验室)
1973: 用 C 重写
1980s: 商业 Unix 盛行
1990s: 版权收紧，各方出走
```

---

## POSIX：让大家别再吵了

80 年代，各种 Unix 变体越来越多——Sun 的 Solaris、IBM 的 AIX、HP 的 HP-UX……每个都号称是"Unix"，但命令、接口、系统调用之间总有细微差别。

写一个程序，得针对不同的"Unix"做不同的适配。

IEEE（电气电子工程师学会）看不下去了，牵头制定了一个标准：**POSIX**（Portable Operating System Interface，可移植操作系统接口）。

POSIX 定义了：
- 系统调用长什么样（比如 `fork()`、`open()`）
- 命令行的行为应该是什么（比如 `ls` 应该有 `-l`、`-a` 这些选项）
- C 语言标准库怎么与系统交互

只要一个系统声称"POSIX 兼容"，就意味着你可以用同样的方式在上面写程序、敲命令。

这就是为什么今天在 Linux、macOS、各种 BSD 上，`ls -l` 的行为基本一致——它们都遵循同一本标准。

POSIX 本身不是操作系统，它是**一纸标准**。任何系统都可以照着实现。Linux 不是 Unix，但它是 POSIX 兼容的。

---

## GNU：自由软件运动

1983 年，Richard Stallman 在 MIT 发起了一个项目：**GNU**（"GNU's Not Unix"——这是个递归缩写）。

他为什么做这件事？因为当时的软件世界正在走向封闭——Unix 不让免费使用和修改，其他软件也越来越多地变成专有软件。Stallman 认为这是不对的。

GNU 的目标是创建一个**完全自由**的 Unix 兼容操作系统。自由的意思是：

- 可以自由运行
- 可以查看源码
- 可以修改
- 可以分发修改后的版本

GNU 项目开发了大量 Unix 兼容的工具：GCC（编译器）、Emacs（编辑器）、Coreutils（`ls`、`cp`、`mv` 等核心命令）、Bash（Shell）……

到 90 年代初，GNU 已经有了操作系统需要的所有组件——**除了一个东西：内核**。

它需要一个内核来驱动硬件、管理进程和内存。而就在这时，一个芬兰大学生搞出了一个内核。

---

## Linux：最后一块拼图

1991 年，Linus Torvalds 在赫尔辛基大学发布了他写的一个操作系统内核，叫 **Linux**。

内核是操作系统的核心：管理 CPU、内存、硬盘、网络……没有内核，系统就无法运转。

Linus 用 GPL 许可证发布了 Linux——GNU 项目的许可证。这意味着 Linux 可以跟 GNU 工具链无缝结合。

于是 GNU 的工具 + Linux 的内核 = 一个完整的、自由的操作系统。

很多人坚持叫它 **GNU/Linux**，以承认 GNU 的贡献。但大部分人就简称为 Linux。

这就是为什么：
- 你装的"Linux 系统"里，大部分命令其实是 GNU 的工具
- `ls --version` 通常会显示 `ls (GNU coreutils)`——你用的 `ls` 是 GNU 写的，不是 Linux 写的
- 在 macOS（BSD 系）和 Linux（GNU 系）上，同样一个命令的行为可能有细微差别

```
GNU 工具链 + Linux 内核 = 完整的操作系统
     ↑                            ↑
   1983                         1991
```

---

## BSD：另一个 Unix 分支

BSD（Berkeley Software Distribution）源于加州大学伯克利分校。1977 年开始，伯克利给 Unix 用户分发包含自己开发的增强工具的软件包。

最开始它只是对 AT&T Unix 的补充，后来越来越独立。到了 4.4 BSD 版本，伯克利重写了几乎所有原本来自 AT&T 的代码，BSD 成了一个**不含 AT&T 代码的、自由的 Unix 变体**。

分支演化出了几个主要的现代 BSD 系统：
- **FreeBSD** — 最流行的 BSD，注重性能和易用性
- **OpenBSD** — 注重安全性，号称"六年默认安装只发现两个远程漏洞"
- **NetBSD** — 注重可移植性，能跑在各种奇怪的硬件上

macOS 的内核（XNU）里包含了大量 FreeBSD 的代码。这就是为什么在 macOS 上敲命令，行为经常跟 FreeBSD 更像，而不是跟 Linux 一样。

比如 `ls -l` 在 macOS 上输出的是 `-rw-r--r--`，用 `--version` 呢？

```zsh
ls --version
```

macOS 上会报错——因为 macOS 的 `ls` 来自 BSD，不支持 `--version`。Linux 上则能正常输出版本信息，因为用的是 GNU coreutils 的 `ls`。

```
AT&T Unix
  ├── System V (商业 Unix 分支)
  ├── BSD (伯克利分支)
  │    ├── FreeBSD
  │    ├── OpenBSD
  │    ├── NetBSD
  │    └── macOS (继承了 BSD 用户态工具)
  └── GNU/Linux (完全独立的实现，POSIX 兼容)
```

---

## 放在一起看

| 名字 | 是什么 | 关键人物 | 诞生年份 |
|------|--------|----------|----------|
| **Unix** | 第一个现代操作系统 | Ken Thompson, Dennis Ritchie | 1969 |
| **POSIX** | 操作系统接口标准 | IEEE | 1988 |
| **GNU** | 自由软件工具链 | Richard Stallman | 1983 |
| **Linux** | 开源操作系统内核 | Linus Torvalds | 1991 |
| **BSD** | Unix 的自由分支 | 加州大学伯克利分校 | 1977 |

它们的关系：

```
Unix 创造了一种设计范式
  └── POSIX 把这种范式写成标准
       ├── GNU 实现了这个标准（工具链）
       └── BSD 继承了 Unix 的血统（并在 macOS 中延续）
```

**所以当我们说"学命令行"的时候，到底学的是什么？**

不是 Unix 本身，不是 POSIX 标准，而是它们共同确立的那套思维方式和操作习惯。不管在哪个系统上，`ls` 就是列文件，`|` 就是管道，`grep` 就是搜索——这些概念穿越了半个世纪，一直用到现在。

---

## 动手试试

```zsh
# 看看你的 ls 是从哪来的
which ls
ls --version          # macOS 上会报错——因为不是 GNU 的
/usr/bin/ls --version # 一样

# 看看系统的 uname
uname -a              # 能看到内核信息

# 看看系统说自己是什么
uname -s              # Darwin = macOS 的内核名
uname -o              # macOS 不支持这个选项

# 如果你的系统上装了 GNU 工具
# brew install coreutils 之后
gls --version         # 前面加 "g" 的通常是 GNU 版本
```

---

> [← 查看系列目录]({% link _pages/series-command-line.md %})
