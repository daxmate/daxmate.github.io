---
layout: single
title:  "从零开始学命令行：进程管理"
date:   2026-07-01 14:00:00 +0800
categories:
  - Command Line
tags:
  - 命令行
  - 进程
  - ps
  - top
  - kill
  - 后台运行
---

电脑上同时跑着几百个程序——有些是你打开的（浏览器、编辑器、终端），有些是系统在后台默默干的。每一个正在跑的程序都是一个**进程**。

进程管理就是：看看谁在跑、谁在偷资源、卡死了怎么关掉、怎么让程序在后台跑。

---

## 看看谁在跑：`ps`

不加任何选项的 `ps` 很骨感，只显示当前终端里的进程：

```zsh
ps
```

```
  PID TTY           TIME CMD
12345 ttys001    0:00.21 -zsh
```

`PID` 是进程 ID——每个进程的唯一编号。

想看更多，用 `ps aux`（这是 BSD/macOS 的经典组合）：

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
| RSS | 实际物理内存 |
| STAT | 状态（R=运行中，S=休眠，Z=僵尸） |
| START | 启动时间 |
| TIME | 累计 CPU 时间 |
| COMMAND | 命令 |

### 筛选

太多了，配合 `grep` 只找关心的：

```zsh
ps aux | grep "python"
```

但这样会把 `grep python` 自己也匹配出来。加一个排除的小技巧：

```zsh
ps aux | grep "[p]ython"
```

`[p]` 是一个正则字符类，只匹配 `p`。这行的字面是 `[p]ython`，但正则匹配的是 `python`。`grep` 自己命令的参数是 `[p]ython`，这个字面不会匹配 `python`，所以 grep 自己不会出现在结果里。

或者更直接的办法：

```zsh
ps aux | grep python | grep -v grep
```

---

## 实时监控：`top`

`ps` 拍的是一瞬间的快照。想实时看系统负载，用 `top`：

```zsh
top
```

屏幕会持续刷新，显示当前占用 CPU 和内存最多的进程。按 `q` 退出。

`top` 默认按 PID 排序。想知道谁是 CPU 大户，进了 `top` 之后按 `o` 然后输入 `cpu`，回车——按 CPU 使用率排。

macOS 上的 `top` 还可以按内存排序：进 `top` 后按 `o` 输入 `mem`。

---

## 关掉进程：`kill`

某个程序卡死了，关不掉，用 `kill` 强制终止。

先找到它的 PID：

```zsh
ps aux | grep "程序名"
```

然后：

```zsh
kill PID
```

如果普通 `kill` 不管用，上更强的：

```zsh
kill -9 PID
```

`-9` 发送的是 SIGKILL 信号——操作系统直接砍掉进程，不给它收拾的机会。相当于「你被解雇了，现在立刻离开」。`kill` 不带数字的话发送的是 SIGTERM，相当于「你被解雇了，收拾收拾走人吧」。大多数时候 SIGTERM 就够了，不响应再用 `-9`。

### 按名字关：`pkill`

如果记得名字但不想查 PID：

```zsh
pkill 程序名
```

比如：

```zsh
pkill -f "python server.py"
```

`-f` 匹配完整的命令行，不只是进程名。`pkill python` 只匹配进程名是 `python` 的，`pkill -f "python server.py"` 才会匹配到 `python server.py` 这个完整命令。

---

## 后台运行

### 让程序在后台跑：`&`

```zsh
python server.py &
```

```
[1] 12346
```

`[1]` 是作业编号，`12346` 是 PID。程序在后台跑，终端还能继续用。

但它的输出还是会打到屏幕上，混在你的输入里。如果输出很多，可以重定向掉：

```zsh
python server.py > /dev/null 2>&1 &
```

### 前台程序切到后台

如果你已经在前台跑了一个程序，想把它切到后台：

1. 按 `Ctrl + Z` 暂停程序
2. 敲 `bg` 让它在后台继续跑

### 后台程序切回前台

```zsh
fg
```

把最近一个后台程序拉回前台。

---

## 查看后台作业：`jobs`

```zsh
jobs
```

```
[1]  + running    python server.py
[2]  - suspended  vim notes.txt
```

`+` 表示最近操作的作业，`-` 是上一个。

拉指定作业到前台：

```zsh
fg %2   # 把作业编号为 2 的程序切回前台
```

---

## 不受终端关闭影响：`nohup`

`&` 绑定的后台进程会在终端关闭时一起被杀掉。如果想让程序**在终端关了之后继续跑**，用 `nohup`：

```zsh
nohup python server.py &
```

`nohup` 是 **no hang up** 的缩写——即使终端关掉（hang up），程序也不停。输出默认写到 `nohup.out`。

---

## 杀死后台作业

```zsh
kill %1        # 按作业编号
kill 12346     # 按 PID
```

---

## 动手试试

```zsh
# 看看自己在跑什么
ps
ps aux | head -n 5

# 实时监控
top
# 按 q 退出

# 开一个后台进程
sleep 100 &
jobs

# 再开一个
sleep 200 &
jobs

# 看看它的 PID
ps aux | grep sleep

# 关掉它
kill %1
jobs

# 清理
kill %2 2>/dev/null
```

---

## 小结

| 命令 | 作用 |
|------|------|
| `ps` | 看当前终端进程 |
| `ps aux` | 看系统所有进程（BSD 格式） |
| `top` | 实时监控系统资源 |
| `kill PID` | 终止进程（先 SIGTERM，不管用再 `-9`） |
| `pkill 名字` | 按名字终止进程 |
| `command &` | 后台运行 |
| `Ctrl + Z` → `bg` | 前台暂停→切后台 |
| `fg` | 切回前台 |
| `jobs` | 查看后台作业 |
| `nohup command &` | 后台运行且不受终端关闭影响 |

日常最常用的就三个场景：
- 看谁在吃资源：`top` 或 `ps aux | head`
- 程序卡死了：`ps aux | grep 名字` → `kill PID`
- 让东西在后台跑：`command &`

下一篇是命令行系列的最后一站——`curl`、`ping`、`ssh` 这几个网络工具，让你不离开终端就能上网打交道。

---

> [← 查看系列目录]({% link _pages/series-command-line.md %})
