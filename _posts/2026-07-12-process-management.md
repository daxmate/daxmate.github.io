---
layout: single
title:  "从零开始学命令行：进程管理——ps、top、kill 与信号"
date:   2026-07-12 17:23:00 +0800
categories:
  - Command Line
tags:
  - 命令行
  - 进程
  - ps
  - top
  - kill
  - 信号
---

电脑上同时跑着几百个程序——有些是你打开的（浏览器、编辑器、终端），有些是系统在后台默默干的。每一个正在跑的程序都是一个**进程**。

进程管理就是：看看谁在跑、谁在偷资源、卡死了怎么关掉。

---

## 看看谁在跑：`ps`

`ps` 是 **process status** 的缩写，最早在 Version 4 Unix（1973 年）的开发者手册里出现，那时候还没有标准的 Shell 语言入口（Shell 需要等一年后的 V7 才出现）。最初的实现直接读取 `/dev/mem` 来获取内核进程表的信息。几十年下来，`ps` 命令的选项在各个 Unix 变体上花样百出——Linux 的 GNU 版 `ps` 甚至支持了上百个选项——但核心设计没变。

不加任何选项的 `ps` 很骨感，只显示当前终端里的进程：

```zsh
ps
```

```
  PID TTY           TIME CMD
12345 ttys001    0:00.21 -zsh
```

`PID` 是进程 ID——每个进程的唯一编号。`TTY` 是关联的终端，`TIME` 是累计消耗的 CPU 时间。

想看更多，用 `ps aux`（这是 BSD/macOS 的经典组合，Linux 也兼容）：

```zsh
ps aux
```

输出一大串，每列的含义：

| 列 | 含义 |
|-----|------|
| USER | 谁在跑 |
| PID | 进程 ID |
| %CPU | CPU 使用率 |
| %MEM | 内存使用率 |
| VSZ | 虚拟内存大小 |
| RSS | 实际物理内存 RSS |
| STAT | 状态（R=运行中，S=休眠，Z=僵尸） |
| START | 启动时间 |
| TIME | 累计 CPU 时间 |
| COMMAND | 命令 |

### 进程的几种状态

`STAT` 那列代表了进程当前在干什么：

- **R（Running）**——正在运行或可运行（在运行队列里等着 CPU）
- **S（Sleeping）**——在等什么东西（等 I/O、等网络、等定时器），这是最常见的状态
- **D（ uninterruptible sleep）**——深度睡眠，通常在等磁盘 I/O，这个状态下的进程连 `kill -9` 都杀不死
- **Z（Zombie）**——僵尸进程——已经死了但还没被清理，通常只存在一瞬间
- **T（Stopped）**——被暂停了（按了 `Ctrl+Z` 就会变成这个状态）

### 进程关系

进程不是孤立的。每个进程都有父进程（parent process），`PPID` 列可以看它的爸爸是谁。整个进程树从 `PID 1`（`launchd` on macOS，`init` 或 `systemd` on Linux）开始发芽。

用 `pstree`（macOS 上可以用 `brew install pstree` 装）可以直观地看进程树：

```zsh
pstree
```

### 筛选输出

`ps aux` 输出太长了，配合 `grep` 只找关心的：

```zsh
ps aux | grep "python"
```

但这样会把 `grep python` 自己也匹配出来。加一个排除的小技巧：

```zsh
ps aux | grep "[p]ython"
```

`[p]` 是一个正则字符类，只匹配 `p`。`grep` 自己命令行里的 `[p]ython` 不会匹配正则 `python`，所以 `grep` 自己被排除在外了。

---

## 实时监控：`top`

`ps` 拍的是一瞬间的快照。想实时看系统负载，用 `top`：

```zsh
top
```

屏幕会持续刷新。`top` 是 William LeFebvre 在 1984 年写的，当年他管理着一台被几十个用户共享的 BSD 系统，总有人跑着占 CPU 的程序不放手。于是 `top` 诞生了——这个命令的设计初衷到今天也没变：看一眼就知道谁在吃资源。

按 `q` 退出 `top`。

实用操作：
- 进 `top` 后按 `o`，输入 `cpu`，回车——按 CPU 使用率排序
- 按 `o` 输入 `mem`——按内存使用排序
- 按 `?` 显示帮助

macOS 上的 `top` 和 Linux 上的 `top` 有一点差异，但基本操作逻辑一样。如果觉得 `top` 不够直观，可以试试 `htop`（`brew install htop`），带颜色、可以鼠标操作。

---

## 信号——进程间的小纸条

Unix 里的**信号（signal）**是一种简单的进程间通信方式——告诉进程「出事了」或者「该干嘛了」。信号机制从 1970 年代早期的 Bell Labs 就存在了，最初只有 9 个信号，现在已经扩展到三十多个。

常用的信号（以 macOS 编号为例，不同 Unix 系统稍有差异）：

| 信号名 | 编号 | 默认行为 | 用途场景 |
|--------|------|----------|----------|
| SIGHUP | 1 | 终止 | 终端关闭时发给前台进程组 |
| SIGINT | 2 | 终止 | 按 `Ctrl+C` 发送 |
| SIGQUIT | 3 | 终止+core dump | 按 `Ctrl+\` 发送，会生成崩溃转储 |
| SIGKILL | 9 | 终止 | 强制杀死，进程不能拦截 |
| SIGTERM | 15 | 终止 | `kill` 默认发送，给进程收拾时间 |
| SIGSTOP | 17 | 暂停 | 暂停进程，不能拦截 |
| SIGCONT | 19 | 继续 | 让暂停的进程继续跑 |

> 不同系统上部分信号的编号有差异（比如 Linux 上 SIGSTOP 是 19，SIGCONT 是 18），但 1、2、3、9、15 在主流 Unix 系统上通用。

信号能被进程**忽略或捕获**（除了 9 和 19）。这也就是 `kill -9` 作为「终极大招」的原因——SIGKILL 不能被任何进程忽略。

---

## 关掉进程：`kill`

`kill` 的名字有点误导——它实际干的是**发送信号**，不只是杀死。不带选项的 `kill` 默认发送 SIGTERM（15 号），给进程一个优雅退出的机会。

```zsh
kill PID
```

先找到它的 PID：

```zsh
ps aux | grep "程序名" | grep -v grep
```

然后：

```zsh
kill 12345   # 发送 SIGTERM
```

如果进程不响应，再用 SIGKILL：

```zsh
kill -9 PID      # 或者 kill -KILL PID
kill -SIGKILL PID
```

记 GDB 调试案例的时候，不！要！用 `kill -9`。它会直接干掉进程，不给调试器处理断点的机会。先用 `kill`（SIGTERM），不行再上 `-9`。

### 按名字关：`pkill`

如果不记得 PID 但记得名字：

```zsh
pkill 程序名
pkill -f "python server.py"
```

`-f` 匹配完整的命令行，不只是进程名。`pkill python` 只匹配进程名是 `python` 的，`pkill -f "python server.py"` 才会匹配到 `python server.py` 这个完整命令。

用到 `pkill` 的时候格外小心——名字太短可能杀掉意外的东西。比如 `pkill java` 只杀 Java 进程，但 `pkill j` 就可能误伤很多名字带 j 的进程。

### 按名字全杀：`killall`

macOS 上还有一个 `killall`，按**进程名**杀掉所有同名进程。名字不匹配就不杀，不会误伤：

```zsh
killall -9 Finder
killall -TERM Safari
```

`killall` 在 macOS 上一个实用场景是重启 Finder：

```zsh
killall Finder
```

Finder 收到 SIGTERM 后自动重启，适合改了 Finder 偏好设置需要刷新的时候。

> Linux 也有 `killall`，行为类似但来自不同的包（`psmisc`/`procps-ng`）。macOS 和 Linux 的 `killall` 都可以按名字批量杀进程，注意不是系统里的 `kill` 加 `all` 选项——是另一个命令。

我们已经在一篇单独的文章里详细聊过前后台任务管理（`&`、`bg`、`fg`、`jobs`、`nohup`、`disown`），这里简单提一下和进程管理相关的点：

后台任务的本质就是**进程**。`jobs` 列出的是 Shell 任务表中的作业，用 `ps` 一样能看到这些进程的 PID。杀死后台进程也一样用 `kill`：

```zsh
sleep 100 &
kill 上一步显示的 PID
```

---

## 动手试试

```zsh
# 看看自己在跑什么
ps
ps aux | head -n 5

# 看进程树（如果装了 pstree）
pstree 2>/dev/null || echo "没装 pstree"

# 实时监控
top
# 按 q 退出

# 启动一个进程
sleep 100 &
# 用 ps 找到它
ps aux | grep sleep

# 发送 SIGTERM
kill 上一步显示的 PID
# 再查一下确认没了
ps aux | grep sleep

# 杀不听话的时候用 -9 看看效果
sleep 200 &
kill -9 上一步显示的 PID
```

---

## 小结

| 命令/操作 | 作用 |
|-----------|------|
| `ps` | 看当前终端进程 |
| `ps aux` | 看所有进程（BSD 格式） |
| `top` | 实时监控资源 |
| `kill PID` | 发 SIGTERM 让进程退出 |
| `kill -9 PID` | 发 SIGKILL 强制杀死 |
| `pkill 名字` | 按名字关（注意误杀！） |
| `killall 名字` | 按进程名全杀（精确匹配） |
| `kill -信号 PID` | 发任意信号 |

几个要点记住：

- **先 SIGTERM，不行再 `-9`** > `-9` 是终极大招
- 进程状态看 STAT 列：R 在跑，S 在等，Z 出事了
- 进程的信号 9 和 19 不能被拦截——这就是为什么 `kill -9` 永远有效

---

> [← 查看系列目录]({% link _pages/series-command-line.md %})
