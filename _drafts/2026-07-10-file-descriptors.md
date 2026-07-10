---
layout: single
title:  "从零开始学命令行：文件描述符——不只是 0、1、2"
date:   2026-07-10 10:30:00 +0800
categories:
  - Command Line
tags:
  - 文件描述符
  - 重定向
---

上一篇管道和重定向里，反复提到「stdin（0）」「stdout（1）」「stderr（2）」这些编号。当时说「后面细讲」，现在兑现。

这篇把文件描述符这件事从根上讲透。理解了它，再看重定向、管道、甚至网络 socket，都会有种豁然开朗的感觉——因为它们本质上是同一个东西：**向一个整数读写数据**。

---

## 一个整数背后

随便打开一个文件：

```c
int fd = open("notes.txt", O_RDONLY);
char buf[1024];
read(fd, buf, sizeof(buf));
close(fd);
```

`open` 返回一个整数——**文件描述符**（file descriptor，简称 fd）。后面所有的读写操作，都靠这个整数来指代「那个文件」。

进程创建时，内核维护一张**文件描述符表**（per-process file descriptor table）。这张表的核心是：

```text
文件描述符表（在 kernel 里）
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

表里每一项记录的是「这个 fd 指向内核中哪个打开的文件描述」。`read(fd)` 就是告诉内核：「从 fd 指向的那个地方读」。

这就是文件描述符的本质——**一个数组索引**，每个整数索引到内核表中的一个打开文件记录。所有系统调用都通过这个索引来工作。

---

## 为什么是 0、1、2

进程启动时，shell 替它打开三样东西：

| 编号 | 名称 | 打开目的 | 对应 C 常量 |
|------|------|----------|------------|
| 0 | stdin | 用来读输入 | `STDIN_FILENO` |
| 1 | stdout | 用来写输出 | `STDOUT_FILENO` |
| 2 | stderr | 用来写错误消息 | `STDERR_FILENO` |

为什么从 0 开始编号？因为 `open()` 返回的是**当前可用的最小整数**。进程启动时表是空的，第一个打开的（stdin）拿到 0，然后 1，然后 2。

这个约定来自早期的 Unix，后来写进了 POSIX 标准。所有 Unix 系的操作系统都遵守，包括 Linux、macOS、各种 BSD。如果一个程序不遵守——自己把 fd 0 关了去打开别的文件——麻烦的是它自己。

但 shell 层面的「重定向」替换的正是这张表里的条目。不是改程序，是改 table。

---

## 在命令行里看它们

macOS 和 Linux 都可以：

```console
$ ls -l /dev/fd/
total 0
lrwx------  1 dax  staff  64 Jul 10 10:00 0 -> /dev/ttys000
lrwx------  1 dax  staff  64 Jul 10 10:00 1 -> /dev/ttys000
lrwx------  1 dax  staff  64 Jul 10 10:00 2 -> /dev/ttys000
```

当前 shell 进程的 fd 0、1、2 都指向同一个终端设备——因为是交互式 shell，输入来自键盘，输出和错误都打到屏幕。

试试重定向之后再看：

```zsh
$ ls -l /dev/fd/ > /tmp/fd-snapshot.txt
$ cat /tmp/fd-snapshot.txt
total 0
lrwx------  1 dax  staff  64 Jul 10 10:00 0 -> /dev/ttys000
-rw-------  1 dax  staff   0 Jul 10 10:01 /tmp/fd-snapshot.txt
lrwx------  1 dax  staff  64 Jul 10 10:00 2 -> /dev/ttys000
```

这次 fd 1 指向了一个普通文件（重定向的目标文件），不是终端。而 fd 0 和 2 仍然指向终端。

实际显示因系统和 shell 而异——macOS 上 `/dev/fd` 是伪设备，Linux 上则是 `/proc/self/fd` 的符号链接。但核心思想一样：打开这个目录看到的是当前进程自己的 fd 表快照。

Linux 上 `/proc/self/fd/` 效果一样。

---

## 重定向就是修改这张表

写了这么久的 `> file`，背后发生了什么？

内核中大致是这样操作的（以 `command > out.txt` 为例）：

1. shell 先 fork 出子进程（子进程继承了 shell 的 fd 表）
2. 在子进程中，`open("out.txt", O_WRONLY | O_CREAT | O_TRUNC)`
3. 得到一个新 fd（比如 3）
4. `dup2(3, 1)` —— 把 fd 1 的表项指向 `out.txt` 的打开记录
5. `close(3)` —— 关掉临时 fd
6. `exec(command)` —— 替换成目标程序，但 fd 表不动

子进程的 fd 1 现在指向的是 `out.txt`。程序自己完全不知道——它只管往 fd 1 `write`。写到了文件里，不是屏幕上。

---

## 一个细节：`> file 2>&1` 和 `2>&1 > file` 顺序为什么不同

这件事在上一篇只写了结果，没解释原因。现在可以讲清楚了。

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
结果：fd 1 → file，fd 2 → 屏幕 ✗（不是我们想要的）
```

画出图更清楚：

```text
正确写法：
         fd 1 ──→ file ←── fd 2

错误写法：
   步骤 1: fd 2 ──→ fd 1 ──→ 屏幕
   步骤 2: fd 2 ──→ 屏幕（不变）
           fd 1 ──→ file（走远了）
```

在 shell 中，重定向是按照**从左到右的顺序依次修改 fd 表**。每次 `N>&M` 都是取**当前** M 指向的目标，不是最终的目标。

想确认的话，可以自己验证：

```zsh
$ echo "hello" 2>&1 > file
$ cat file
hello
```

这个例子里 stderr 本来就没有输出，所以看不出区别。那就让 stderr 有输出：

```zsh
$ ls /nonexistent 2>&1 > file
```

这条命令执行时——`2>&1` 先让 stderr 指向屏幕（当前 stdout 的位置），然后 `> file` 把 stdout 重定向到文件——所以屏幕上会看到错误信息，而文件里是空的（因为 `ls` 没输出正常内容）。

换过来才对：

```zsh
$ ls /nonexistent > file 2>&1
$ cat file
ls: /nonexistent: No such file or directory
```

都进了文件。

---

## 不只是 0、1、2：用 `exec` 操作自定义描述符

shell 允许我们直接操作自己的 fd 表，不需要运行任何外部命令。用的是 `exec`。

### 打开文件用于写入

```zsh
exec 3> log.txt
```

现在 fd 3 指向了 `log.txt`。用 `echo` 往 fd 3 写：

```zsh
echo "first line" >&3
echo "second line" >&3
```

### 从文件读取

```zsh
exec 3< config.txt
read -u 3 line
echo $line
```

`-u 3` 告诉 `read` 从 fd 3 读，不是默认的 fd 0。

### 同时读写

```zsh
exec 3<> data.txt
```

`<>` 打开文件同时用于读写。不常用，但有时候写网络工具会遇到。

### 关闭

```zsh
exec 3>&-
```

`>&-` 是「关闭这个 fd」的意思。关闭读端用 `<&-`。`exec 3>&-` 就是把 fd 3 从 fd 表里移除。

### 实际场景：临时保存和恢复

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

这段脚本的输出：第一句去文件，第二句回屏幕。用到 `exec 5>&1` 这个技巧——先把当前 fd 1 的指向「备份」到 fd 5，等改完了再恢复。

---

## 特殊文件：`/dev/null`、`/dev/zero`、`/dev/fd`

这三个是 Unix 里最有代表性的伪设备。

### `/dev/null` —— 黑洞

往里面写什么，什么就消失。从里面读，得到 EOF（文件结束）。

```zsh
command > /dev/null 2>&1
```

这样跑一个命令，它产生的所有输出都不会出现在屏幕上，也不会存到任何地方。跑完就完了。

用得最多的地方：

- 只要命令的副作用（比如返回值），不要输出
- 脚本中安静地执行一个操作
- 忽略某个命令的错误消息

### `/dev/zero` —— 无限零

读它，会得到无穷无尽的 `\0` 字节（不是数字零，是空字节）。

```zsh
# 创建一个 1MB 的空文件
dd if=/dev/zero of=blank.bin bs=1024 count=1024
```

用在哪？创建虚拟磁盘镜像、预分配交换文件、安全擦除前的覆盖。日常写脚本很少直接碰它，但它展示了文件描述符机制的另一面——「文件」不一定是磁盘上的数据，也可以是内核提供的无限供应。

### `/dev/fd` —— 当前进程的 fd 视图

前面演示过了。它本质上是一个特殊的文件系统入口，内核会根据你打开 `/dev/fd/N` 时当前进程的 fd 表来决定指向哪里。

在 macOS 上它是独立的伪设备，在 Linux 上它是指向 `/proc/self/fd` 的符号链接。效果一样。

有一个实用技巧：如果你想从 fd 的内容「复制」一个副本，可以：

```zsh
exec 3< /dev/fd/0   # 让 fd 3 读当前 stdin 的内容
```

在脚本里偶尔会用——把当前 stdin 保存起来，等用完了别的输入再恢复。

---

## 一个完整的例子

写一段脚本，展示怎么用自定义 fd 做日志记录和恢复：

```zsh
#!/usr/bin/env zsh

# 把 fd 3 分配给日志文件
exec 3> script.log

# 正常的 stdout 和 stderr 不变
echo "屏幕上能看到这条"

# 同时往日志里写一份
echo "[$(date)] 用户登录" >&3

# 模拟一个错误（写到 stderr）
echo "警告：磁盘空间不足" >&2

# 把错误也记到日志
echo "[$(date)] 警告：磁盘空间不足" >&3

# 关闭日志 fd
exec 3>&-

echo "脚本执行完毕"
```

运行后，屏幕的输出和日志文件里的内容互不干扰。需要注意：`>&3` 只能写入已经打开的 fd，如果 fd 3 没打开直接写会报错。

---

## 理解文件描述符的好处

- 看 `2>&1` 不再死记硬背——知道这是「把 fd 2 的指向改成和 fd 1 一样」
- 能解释为什么 `>` 和 `<` 顺序很重要——本质上是在操作 fd 表，每一步都有状态
- 遇到 `exec 3> file` 不会慌——知道 shell 能操作自己的 fd 表
- 理解 `/dev/fd`、`/dev/null` 这些不是普通文件——它们是内核暴露的接口，通过 fd 机制访问

Unix 里「一切皆文件」这句话有点被说滥了。但文件描述符机制确实是从底层支撑了这个理念——不管背后是普通文件、终端、网络 socket、管道还是伪设备，在程序看来都是**一个整数 + 三个系统调用（read、write、close）**。

---

> [← 查看系列目录]({% link _pages/series-command-line.md %})
