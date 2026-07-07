---
layout: single
title:  "从零开始学命令行：Shell 是怎么找到命令的"
date:   2026-07-07 10:00:00 +0800
categories:
  - Command Line
tags:
  - 命令行
  - shell
  - PATH
  - builtin
  - hash
---

敲了那么多命令，有没有想过一个问题：**当我们敲 `ls` 的时候，Shell 怎么知道 `ls` 在哪？**

它不是一个文件吗——怎么不需要告诉 Shell 它在哪个目录，敲名字就能跑？

这篇就来看看这个机制。搞懂了它，以后装完软件遇到 command not found，就知道问题出在哪了。

---

## Shell 找命令的三个步骤

当我们敲了 `ls` 按回车，Shell 做了三件事，一秒钟都不到：

1. **先看是不是自己就会的**（内置命令）
2. **再看有没有缓存**（hash 表）
3. **最后去目录里找**（PATH）

下面一步步拆开看。

---

## 第一步：内置命令

有些命令 Shell 自己就会——不需要去硬盘上找任何文件。比如 `cd`、`echo`、`type`、`export`、`alias`。

这些叫「内置命令」（builtin）。它们直接编译在 Shell 的代码里，敲下去瞬发。

验证一下：

```zsh
type cd
```

```
cd is a shell builtin
```

再看看 `ls`：

```zsh
type ls
```

```
ls is an alias for ls -G
```

哎等等——这里还有一层，Shell 其实先看了 alias。完整顺序是这样的：

1. 先检查是不是 alias（别名）
2. 再检查是不是 builtin（内置命令）
3. 都不是，才去外部找

不过 alias 不是这篇的重点，后面再单独聊。

---

## 第二步：hash 缓存

如果命令不是内置的，Shell 不会每次都去硬盘上搜一遍——太慢了。它会记住上次找到的位置。

```zsh
ls                      # 先敲一次，让 Shell 记住位置
hash                    # 查看缓存
```

```
hash
hits    command
  1     /bin/ls
```

`/bin/ls` 就是 `ls` 这个命令在硬盘上的实际位置。Shell 记下了这个地址，下次敲 `ls` 就直接去 `/bin/ls`，跳过搜索步骤。

缓存什么时候会出问题？装了新版本的程序或者移除了某个目录之后，缓存可能还是旧的。这时候清一下缓存就行：

```zsh
rehash    # zsh
hash -r   # bash
```

---

## 第三步：PATH 搜索

到了这一步，Shell 确认这不是内置命令，缓存里也没有（或者没命中），那就得去硬盘上找了。

问题来了：**去哪找？**

总不能把整个硬盘翻一遍——太慢了。Shell 只看 `PATH` 环境变量里列出来的目录。

```zsh
echo $PATH
```

```
/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
```

这是 macOS 上的默认 PATH。冒号分隔了五个目录。Shell 会按顺序在这五个目录里挨个找：

1. 先去 `/usr/local/bin` 看看有没有叫 `ls` 的程序
2. 没有，去 `/usr/bin` 看看
3. 还没有，去 `/bin` 看看
4. 找到了！就它了

敲 `which ls` 验证一下：

```zsh
which ls
```

```
/bin/ls
```

`which` 这个命令就是用来告诉我们：Shell 在 PATH 的哪个目录里找到了某个命令。

如果所有目录都找遍了还是没找到，Shell 就报错：

```zsh
some-random-command
```

```
zsh: command not found: some-random-command
```

---

## 为什么 PATH 的顺序很重要

因为 Shell **按顺序查找，找到就停**。所以先出现的目录优先级更高。

假设 `/usr/local/bin` 和 `/usr/bin` 里都有一个叫 `python3` 的程序：

```zsh
echo $PATH
```

```
/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
```

如果两个目录里都有，跑的是 `/usr/local/bin/python3`，因为它在 PATH 里排前面。

这就是为什么用 Homebrew 安装软件后，它经常提醒我们：

```
Warning: /opt/homebrew/bin is not in PATH
```

Homebrew 把自己装的程序放在 `/opt/homebrew/bin`，如果这个目录不在 PATH 里，或者排在其他同名程序后面，敲命令时跑的就可能不是我们想要的版本。

---

## 用 `which` 和 `type` 排查问题

当命令行为异常的时候，先用 `which` 确认它到底从哪来的：

```zsh
which python3
```

```
/usr/local/bin/python3
```

再用 `type` 看看是不是被 alias 或者 builtin 挡住了：

```zsh
type ls
```

```
ls is an alias for ls -G
```

而 `where` 会列出所有找到的位置，能看到版本冲突：

```zsh
where python3
```

```
/usr/local/bin/python3
/usr/bin/python3
```

---

## 动手试试

```zsh
# 看看 PATH 里有哪些目录
echo $PATH

# 挑几个常用命令，看看它们在哪
which ls
which cat
which grep
which python3     # 如果有的话

# 看缓存
hash

# 看 type
type cd
type ls
type which

# 看看一个不存在的命令会怎样
type nonexistent-command
```

---

## 小结

Shell 找命令就三步：

| 步骤 | 查什么 | 说明 |
|------|--------|------|
| alias | 命令别名 | 优先级最高 |
| builtin | 内置命令 | Shell 自己就会，不用找文件 |
| hash | 缓存的上次位置 | 加速用的，装新软件后要清 |
| PATH 搜索 | 遍历目录 | 按 PATH 顺序逐个目录找 |

**装完软件报 command not found → 99% 是 PATH 的问题。**

要么程序没装进 PATH 里的目录，要么 PATH 里没有包含那个目录。

下一篇来聊环境变量——PATH 只是其中之一，还有 HOME、EDITOR 这些老朋友等着认识。

---

> [← 查看系列目录]({% link _pages/series-command-line.md %})
