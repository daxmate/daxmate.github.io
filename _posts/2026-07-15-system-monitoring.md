---
layout: single
title:  "从零开始学命令行：看系统状态——CPU、内存、磁盘、网络"
date:   2026-07-15 16:53:00 +0800
categories:
  - Command Line
tags:
  - 命令行
  - top
  - htop
  - df
  - du
  - 监控
  - 系统状态
---

电脑突然变慢了。风扇呼呼转，点什么都像在泥里走路。这时候怎么办？

大多数人会打开活动监视器（macOS）或任务管理器（Windows），看看谁在捣乱。但命令行里有更轻量、更灵活的方式来看系统状态——而且不用离开终端。

## 看 CPU 和内存：`top`

```zsh
top
```

终端马上变成一张实时刷新的表格：

```
Processes: 423 total, 2 running, 421 sleeping
CPU usage: 12.5% user, 3.2% sys, 84.3% idle
Mem: 16 GB total, 12 GB used, 4 GB free
```

按 `q` 退出。在 `top` 的界面里，还能按 `c` 看完整命令路径，按 `M` 按内存排序，按 `P` 按 CPU 排序。

`top` 是 1984 年一个叫 **William LeFebvre** 的研究生在 Rice University 写的。当时大家共用一个 Unix 服务器，经常有人跑了个程序把 CPU 吃满，其他人连不上。LeFebvre 写了个工具，能一眼看到谁在吃资源——这就是 top。最初代码只有几百行，到今天它几乎是每台 Unix/Linux/macOS 机器上的标配。

### 更好看的版本：`htop`

`top` 够用，但不怎么好看。`htop` 是它的升级版，有颜色、支持鼠标、能上下滚动进程列表：

```zsh
brew install htop   # macOS 上先安装
htop
```

一眼能看到颜色条表示的 CPU 和内存使用率，按 F6 可以按任意字段排序。用习惯了就回不去了。

`htop` 是巴西开发者 **Hisham Muhammad** 在 2004 年做的。他觉得 `top` 的交互方式太原始，就在自己做的 GoboLinux 发行版里写了一个替代品。后来这个替代品成了整个 Linux/macOS 生态里最受欢迎的系统监控工具之一。

---

## 看磁盘空间：`df`

`df` 是 "Disk Free" 的缩写——看磁盘还剩多少空间：

```zsh
df -h
```

```
Filesystem      Size   Used   Avail   Capacity   Mounted on
/dev/disk1s1    233G   180G    45G    80%        /
```

`-h` 是 human-readable，用 G、M 显示，不用看一串数字。

只看系统盘：

```zsh
df -h /
```

`df` 是 Unix 最古老的命令之一，可以追溯到 70 年代贝尔实验室的早期 Unix。它的设计极度简单：不加参数看所有磁盘，加 `-h` 让人看懂。一个命令管了五十年的需求。

---

## 看目录占了多少空间：`du`

`df` 告诉你磁盘快满了，但谁在吃空间？用 `du`（Disk Usage）。

```zsh
du -sh ~/Documents
```

```
12G    /Users/dax/Documents
```

- `-s` 只显示总计
- `-h` 人类可读

找出当前目录下最大的子目录：

```zsh
du -sh */
```

```
4.0G    code
2.5G    Documents
1.2G    Downloads
800M    Pictures
```

配合 `sort` 和 `head` 可以快速定位大文件：

```zsh
du -sh */ | sort -rh | head -5
```

这个组合大概是排查"磁盘满了"最常用的命令。

---

## 看内存详情

Linux 上有专门的 `free` 命令：

```zsh
free -h
```

macOS 上没有 `free`，用 `vm_stat`：

```zsh
vm_stat
```

不过大部分人不用单独看内存——`top` 或 `htop` 打开的瞬间就已经显示了内存占用。

---


## 快速排查场景

```zsh
# 电脑变慢了——先看谁在吃 CPU
top -l 1 | head -10

# 磁盘满了——确认一下
df -h /

# 谁在吃空间——找大目录
du -sh ~/* | sort -rh | head -10

# 网络通不通
ping 8.8.8.8 -c 3
```

---

## 动手试试

```zsh
# 看磁盘空间
df -h

# 看当前目录的大小
du -sh .
du -sh */ | sort -rh

# 看一次系统状态，只看前 5 个进程
top -l 1 -n 5

# 看一眼系统整体状况——全都要
htop
```

---

## 小结

| 想知道什么 | 命令 |
|-----------|------|
| CPU、内存谁在用 | `top` 或 `htop` |
| 磁盘还剩多少 | `df -h` |
| 目录占了多少 | `du -sh 目录` |
| 谁在吃空间 | `du -sh */ \| sort -rh` |

记住 `df -h` 和 `du -sh */ | sort -rh` 这两个就足够应付大部分"磁盘满了"的场面。

---

> [← 查看系列目录]({% link _pages/series-command-line.md %})
