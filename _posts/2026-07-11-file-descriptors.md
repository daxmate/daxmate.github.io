---
layout: single
title:  "从零开始学命令行：文件描述符——0、1、2 背后是什么"
date:   2026-07-11 15:30:00 +0800
categories:
  - Command Line
tags:
  - 文件描述符
  - 重定向
---

在之前的多篇内容里，我们反复看到过「stdin（0）」「stdout（1）」「stderr（2）」这些编号，以及 `>`、`<`、`2>&1` 这些写法。但 0、1、2 到底是什么？

整数。确切地说，是**文件描述符**。

---

## 一个整数背后

打开一个文件：

```c
int fd = open("notes.txt", O_RDONLY);
char buf[1024];
read(fd, buf, sizeof(buf));
close(fd);
```

看不懂 C 没关系，只看第一行就好：`open` 返回一个整数。后面所有读写操作都靠这个整数来指代「那个文件」。

进程启动时，内核维护一张**文件描述符表**：

```text
文件描述符表（内核里）
┌─────┬──────────────┐
│ fd  │  指向什么     │
├─────┼──────────────┤
│  0  │  /dev/ttys000 │
│  1  │  /dev/ttys000 │
│  2  │  /dev/ttys000 │
│  3  │  ~/notes.txt  │
│ ... │  ...          │
└─────┴──────────────┘
```

表里每一项记录「这个 fd 指向内核中哪个打开的文件」。`read(fd)` 就是告诉内核：从 fd 指向的那个地方读。

文件描述符就是一个**数组索引**，每个索引对应内核中的一个打开文件记录。所有系统调用都通过这个索引来工作。

---

## 为什么是 0、1、2

进程启动时，shell 替它打开三样东西：

| 编号 | 名称 | 干什么用 |
|------|------|----------|
| 0 | stdin | 读输入 |
| 1 | stdout | 写输出 |
| 2 | stderr | 写错误消息 |

为什么从 0 开始？因为 `open()` 返回的是**当前可用的最小整数**。进程启动时表是空的，第一个打开的（stdin）拿到 0，然后 1，然后 2。

这个约定从早期 Unix 开始，后来写进 POSIX 标准。Linux、macOS、各种 BSD 都遵守。上一个不遵守的程序早就被淘汰了。

Shell 的「重定向」就是在替换这张表的条目。不是改程序，是改 table。

---

## 死记硬背不如看一次

`ls -l /dev/fd/` 就能看到当前 shell 的 fd 表。

Linux 上输出很清晰——每个 fd 都以符号链接的形式显示它指向的目标。macOS 上输出的类型码各有不同（`c` 是字符设备，`p` 是管道，`s` 是 socket），但本质一样：

```console
$ ls -l /dev/fd/
total 0
crw--w----  1 dax  tty    0x10000001 Jul 11 15:18 0
p-w--w----  0 dax  staff           0 Jul 11 15:18 1
crw--w----  1 dax  tty    0x10000001 Jul 11 15:18 2
dr-------- 19 dax  staff         608 Jul 11 10:49 3
```

0、1、2 都在。你的输出可能跟这个不完全一样——终端类型、shell、系统版本都会影响——但能确信的是：三个默认 fd 永远存在，重定向就是在换它们的指向。

---

## 重定向就是修改这张表

写了这么久的 `> file`，背后发生了什么？

以 `command > out.txt` 为例，大致是这样：

1. shell fork 出子进程（子进程继承了 shell 的 fd 表）
2. 在子进程中，`open("out.txt", O_WRONLY | O_CREAT | O_TRUNC)`
3. 得到一个新 fd（比如 3）
4. `dup2(3, 1)` —— 把 fd 1 的表项指向 `out.txt` 的打开记录
5. `close(3)` —— 关掉临时 fd
6. `exec(command)` —— 替换成目标程序，但 fd 表不动

子进程的 fd 1 现在指向 `out.txt`。程序自己不知道——它只管往 fd 1 `write`。写到了文件里，不是屏幕上。

---

## 一个细节：`> file 2>&1` 和 `2>&1 > file` 为什么不同

刚学重定向的时候，这个区别让人困惑过——但搞懂了 fd 表就很好解释。

### `command > file 2>&1`

从左到右执行：

```
步骤 1: > file     → 把 fd 1 指向 file
步骤 2: 2>&1       → 把 fd 2 指向「当前 fd 1 指向的位置」
                      ——也就是 file
结果：fd 1 和 fd 2 都指向 file ✓
```

### `command 2>&1 > file`

从左到右执行：

```
步骤 1: 2>&1       → 把 fd 2 指向「当前 fd 1 指向的位置」
                      ——当前 fd 1 指向屏幕（终端）
步骤 2: > file     → 把 fd 1 指向 file
结果：fd 1 → file，fd 2 → 屏幕 ✗
```

画出来更清楚：

```text
正确写法：
         fd 1 ──→ file ←── fd 2

错误写法：
   步骤 1: fd 2 ──→ fd 1 ──→ 屏幕
   步骤 2: fd 2 ──→ 屏幕（不变）
           fd 1 ──→ file（走远了）
```

重定向是按照**从左到右的顺序依次修改 fd 表**。每次 `N>&M` 取的是**当前** M 指向的目标，不是最终的目标。

自己验证一下。让 stderr 有输出：

```zsh
$ ls /nonexistent > file 2>&1
$ cat file
ls: /nonexistent: No such file or directory
```

错误信息进了文件。换过来：

```zsh
$ ls /nonexistent 2>&1 > file
```

屏幕上会看到错误信息，文件里是空的。

---

## 操作自定义描述符

Shell 允许直接操作自己的 fd 表，不需要外部命令。用 `exec`。

### 打开写入

```zsh
exec 3> log.txt
```

fd 3 指向了 `log.txt`。往 fd 3 写：

```zsh
echo "first line" >&3
echo "second line" >&3
```

### 打开读取

```zsh
exec 3< config.txt
read -u 3 line
echo $line
```

`-u 3` 告诉 `read` 从 fd 3 读，不是默认的 fd 0。

### 关闭

```zsh
exec 3>&-
```

`>&-` 是关闭 fd 的意思。从 fd 表里移除。

### 临时保存和恢复

```zsh
# 把当前 stdout 备份到 fd 5
exec 5>&1

# 暂时让 stdout 写到文件
exec > output.txt
echo "this goes to file"

# 恢复 stdout
exec 1>&5
exec 5>&-

echo "this goes back to screen"
```

脚本的输出：第一句去文件，第二句回屏幕。

---

## 三个特殊文件

`/dev/null`、`/dev/zero`、`/dev/fd`——最常用的三个伪设备。它们展示了一件事：在 Unix 里，「文件」不一定是磁盘上的数据。

### `/dev/null` —— 黑洞

往里面写什么，什么就消失。从里面读，得到 EOF。

```zsh
command > /dev/null 2>&1
```

只管命令的副作用，不要输出。写脚本的时候天天用。

### `/dev/zero` —— 无限零

读它，得到无穷无尽的 `\0` 字节（不是数字零，是空字节）。

```zsh
# 创建一个 1MB 的空文件
dd if=/dev/zero of=blank.bin bs=1024 count=1024
```

创建虚拟磁盘镜像、预分配文件时用到。日常脚本很少碰它，但它说明了 fd 机制的弹性——「文件」可以是内核提供的无限供应。

### `/dev/fd` —— 当前进程的 fd 视图

前面看过了。Linux 上它是 `/proc/self/fd` 的符号链接，输出清晰。macOS 上单独实现，类型码因会话而异。效果一样——都是当前进程的 fd 表快照。

---

## 动手试试

```zsh
mkdir -p ~/cmd_practice/fd-demo
cd ~/cmd_practice/fd-demo

# 1. 看看你的 /dev/fd
ls -l /dev/fd/

# 2. 让 stdout 去文件
exec 3> test.txt
echo "hello fd 3" >&3
exec 3>&-
cat test.txt

# 3. 验证 2>&1 顺序
ls /nonexistent > ok.txt 2>&1
cat ok.txt                # 错误信息在文件里

ls /nonexistent 2>&1 > fail.txt
cat fail.txt              # 文件是空的

# 4. 自定义 fd 的临时保存和恢复
exec 4>&1                 # 备份 stdout
exec > temp.txt
echo "这条在文件里"
exec 1>&4                 # 恢复
exec 4>&-                 # 关掉备份
echo "这条在屏幕上"
cat temp.txt
```

---

## 小结

核心就一张表：**文件描述符表**。重定向、管道、网络 socket，绕一圈回来操作的其实都是这张表。

- fd 0/1/2 是进程启动时默认打开的三个通道
- `N>&M` 和 `N<&M` 是在复制 fd 表的指向
- 重定向的顺序影响结果，因为每一步都在改表
- `exec N>file` 让你在 shell 里直接操作 fd 表
- `/dev/null`、`/dev/zero`、`/dev/fd` 是伪设备，通过 fd 机制暴露的内核接口

再看到 `2>&1`，不用背了——知道这是「把 fd 2 的指向改成和 fd 1 一样」就够了。

---

> [← 查看系列目录]({% link _pages/series-command-line.md %})
