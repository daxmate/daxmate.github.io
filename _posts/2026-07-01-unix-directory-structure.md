---
layout: single
title:  "从零开始学命令行：Unix 目录结构"
date:   2026-07-01 13:45:00 +0800
categories:
  - Command Line
tags:
  - 命令行
  - 文件系统
  - Unix
  - 目录结构
---

在前面几篇里，我们一直在 `~`（home 目录）和 `Downloads`、`Documents` 这些目录里打转。但如果往上走几级，到根目录 `/`，会看到一大堆名字奇怪的文件夹——`bin`、`etc`、`usr`、`var`……

每个都有明确的职责。把这些目录认一遍——知道了它们各管什么，以后遇到问题知道该往哪儿找。

---

## 先看一眼

打开终端，看看根目录：

```zsh
ls /
```

```
Applications  System  Users  Volumes
bin           dev     home   sbin
etc           tmp     usr    var
```

macOS 的根目录比 Linux 多了一些 Apple 特有的目录，但核心的 `bin`、`etc`、`usr`、`var` 在两者身上是一样的。这篇讲的是这些 Unix 通用的部分。

---

## 逐一说

### `/bin` — 基础命令

**bin** 是 **binaries**（二进制程序）的缩写。这里放着系统启动和单用户模式下必须用到的命令：`ls`、`cp`、`mv`、`rm`、`cat`、`echo`……

没有 `/bin`，系统连开机都开不利索。

### `/sbin` — 系统管理命令

和 `/bin` 类似，但这儿放的是系统管理员用的命令：`mount`、`fsck`、`shutdown`、`reboot`……普通用户一般用不上，但系统维护离不开。

### `/usr` — 大部分软件的「家」

`/usr` 是 **Unix System Resources** 的缩写（最初的意思是 User，现在早不是了）。这是整个系统里最胖的目录之一——绝大部分用户用的软件都在这里：

```zsh
/usr/bin        # 更多的命令（和 /bin 的区别已经越来越模糊）
/usr/local      # 自己装的软件（Intel Mac 上 Homebrew 装在这里）
/usr/local/bin  # Intel Mac 上 Homebrew 的命令行工具
/usr/share      # 共享数据（文档、字体、图标）
```

为什么要把 `/usr/bin` 和 `/bin` 拆开？历史原因——早期硬盘太小，分两套：系统启动必需的放 `/bin`，不那么急的放 `/usr/bin`。现在硬盘早大了，但这个传统还在。

### `/etc` — 配置文件

`/etc` 是 et cetera 的缩写。最初是放杂七杂八东西的杂项目录，后来渐渐变成了**配置文件**的专用目录。

```zsh
/etc/hosts       # 域名映射
/etc/passwd      # 用户信息
/etc/shells      # 系统装了哪些 Shell
```

这个目录下的文件基本都是纯文本，可以用 `cat` 直接看。

### `/var` — 经常变动的数据

`/var` 是 **variable**（可变的）的缩写。里面的数据会经常变化——日志、缓存、临时文件：

```zsh
/var/log         # 系统日志
/var/tmp         # 重启不删的临时文件
```

服务器上 `/var/log` 很重要——网站错误、登录记录全在里面。

### `/tmp` — 临时文件

和 `/var/tmp` 不同，`/tmp` 里的东西**重启就清空**。任何用户都可以往这里写。适合放临时的、不重要的东西。

### `/dev` — 设备文件

Unix 把一切当成文件处理，硬件也不例外：

```zsh
/dev/disk0      # 硬盘（macOS 用 disk0、disk1...，Linux 用 sda、sdb...）
/dev/tty         # 终端
/dev/null        # 黑洞——写进去就消失
/dev/random      # 随机数生成器
```

这就是为什么前几篇我们敲 `tty` 会输出 `/dev/ttys001`——它真的是一个文件。

### `/home` — 用户的家

这我们最熟了。每个用户一个目录，放自己的文档、下载、配置。`~` 就是这个目录的简写。

macOS 上叫 `/Users`，Linux 上叫 `/home`。作用一样。

### `/opt` — 可选软件

一些不是系统自带的独立软件会放在这里。在 Apple Silicon（M1/M2/M3）Mac 上，**Homebrew 就装在 `/opt/homebrew/`**，里面结构跟 `/usr/local/` 一样——`/opt/homebrew/bin` 放命令行工具，`/opt/homebrew/Cellar` 放具体软件包。

所以如果你用的是 Apple Silicon Mac，用的其实是 `/opt/homebrew/bin` 而不是 `/usr/local/bin`。可以用 `which brew` 确认一下。

Intel Mac 和 Linux 上 Homebrew 则放在 `/usr/local/`。

---

## macOS 特有的几个

macOS 的根目录还有几个别的：

| 目录 | 干什么的 |
|------|---------|
| `/Applications` | 所有用户共享的应用（跟用户自己的 `~/Applications` 不一样） |
| `/System` | 系统核心文件，别碰 |
| `/Volumes` | 挂载的磁盘和网络盘会出现在这 |

---

## 为什么要知道这些

初学阶段不需要记住每一个，知道下面这几个就能解决大部分问题了：

- 装了个命令行工具不知道装哪儿了 → `/usr/local/bin`（Intel Mac）或 `/opt/homebrew/bin`（Apple Silicon）
- 某个服务跑不起来想看日志 → `/var/log`
- 想确认一个配置文件的内容 → `/etc` 里找
- 工具起了冲突不知道用哪个 → `which` 查路径，看是不是 Homebrew 的覆盖了系统自带的

---

> [← 查看系列目录]({% link _pages/series-command-line.md %})
