---
layout: single
title:  "从零开始学命令行：看系统状态——CPU、内存、磁盘、网络"
date:   2026-07-09 13:00:00 +0800
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

电脑卡了？硬盘满了？网络不对劲？几个命令就能看到系统当前的状态。

---

## 看 CPU 和内存：`top`

```zsh
top
```

会实时刷新系统状态：

```
Processes: 423 total, 2 running, 421 sleeping
CPU usage: 12.5% user, 3.2% sys, 84.3% idle
Mem: 16 GB total, 12 GB used, 4 GB free
```

按 `q` 退出，按 `c` 看完整命令，按 `M` 按内存排序，按 `P` 按 CPU 排序。

更好用的版本是 `htop`：

```zsh
brew install htop   # macOS上装
htop
```

有颜色，可以上下滚动，还可以用鼠标点——`top` 的升级版。

---

## 看磁盘空间：`df`

```zsh
df -h
```

```
Filesystem      Size   Used   Avail   Capacity   Mounted on
/dev/disk1s1    233G   180G    45G    80%        /
```

`-h` 是 human-readable（用 G、M 显示，不用字节数）。

只关心某个挂载点：

```zsh
df -h /
```

---

## 看目录占了多少空间：`du`

```zsh
du -sh ~/Documents
```

```
12G    /Users/dax/Documents
```

- `-s` 只显示总计
- `-h` 人类可读

看当前目录下每个子目录的大小：

```zsh
du -sh */
```

```
4.0G    code
2.5G    Documents
1.2G    Downloads
800M    Pictures
```

找大文件利器。配合 `sort` 和 `head` 找出最大的几个：

```zsh
du -sh */ | sort -rh | head -5
```

---

## 看内存详情：`free`

Linux 上有 `free` 命令：

```zsh
free -h
```

macOS 上没有 `free`，用 `vm_stat` 代替：

```zsh
vm_stat
```

或者用 `top -l 1 | grep PhysMem`：

```zsh
top -l 1 -s 0 | grep PhysMem
```

---

## 看网络连接：`netstat` 和 `lsof`

```zsh
# 查看正在监听的端口
netstat -an | grep LISTEN

# 查看某个端口谁在用（macOS）
lsof -i :3000
```

`lsof -i` 特别实用——启动本地服务时端口被占用了，用它查谁在用那个端口。

---

## 综合监控：` glances`

`htop` 只看了 CPU 和内存，`glances` 什么都有——CPU、内存、磁盘、网络、进程：

```zsh
brew install glances
glances
```

一个窗口看全系统的状态。

---

## 快速排查场景

```zsh
# 电脑变慢了
top -l 1 | head -10

# 磁盘满了
df -h /

# 查谁在吃空间
du -sh ~/* | sort -rh | head -10

# 端口被占用了
lsof -i :8080

# 网络是否通
ping 8.8.8.8 -c 3
```

---

## 动手试试

```zsh
# 看磁盘
df -h

# 看当前目录空间
du -sh .
du -sh */ | sort -rh

# 看进程
top -l 1 -n 5   # 只看一次，前 5 个进程

# 看端口
lsof -i :22     # 看 SSH 端口谁在监听
```

---

## 小结

| 想知道什么 | 命令 |
|-----------|------|
| CPU/内存 | `top` 或 `htop` |
| 磁盘空间 | `df -h` |
| 目录大小 | `du -sh 目录` |
| 找大目录 | `du -sh */ \| sort -rh` |
| 端口占用 | `lsof -i :端口号` |
| 全状态 | `glances` |

平时记住 `df -h` 和 `du -sh */ | sort -rh` 这两个就够了——电脑卡顿十有八九是磁盘满了。

---

> [← 查看系列目录]({% link _pages/series-command-line.md %})
