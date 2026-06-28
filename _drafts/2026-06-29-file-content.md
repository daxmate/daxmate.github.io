---
layout: single
title:  "从零开始学命令行(5)--查看和搜索文件内容"
date:   2026-06-29 12:00:00 +0800
categories:
  - Command Line
tags:
  - 命令行
  - 文件操作
  - cat
  - head
  - tail
  - grep
  - less
  - wc
---

前几篇我们学了文件操作的基本功——创建、复制、移动、删除。上一篇特别花了篇幅讲删除的事，核心就一个意思：**别用 `rm`，用 `trash`**。

今天的事就是看文件里面写的是什么——以及怎么在一堆内容里快速找到想要的那一行。

---

## `cat` 的更多用法

上一篇已经用过 `cat` 了，就是直接把文件内容打印到屏幕上。但它不止能看一个文件：

```zsh
cat file1.txt file2.txt
```

会把两个文件的内容连在一起输出。`cat` 的名字就是这么来的——**concatenate**，串联。

日常用得更多的其实是配合重定向来合并文件：

```zsh
cat notes.txt diary.txt > combined.txt
```

把两个文件的内容合并成一个新文件。比打开编辑器复制粘贴快得多。

还有一个实用技巧——给 `cat` 加上行号：

```zsh
cat -n notes.txt
```

```
     1	This is my readme file
     2	This is a second line.
```

调试配置文件的时候特别好用，想看哪一行直接看行号就行。

---

## 只看开头几行：`head`

有时候文件太长，你只想知道它长什么样——看看头几行就够了。

```zsh
head notes.txt
```

默认显示前 10 行。想多看或少看，加 `-n`：

```zsh
head -n 5 notes.txt    # 只看前 5 行
head -n 20 notes.txt   # 只看前 20 行
```

这个命令最适合快速确认文件的格式。比如拿到一个 CSV 文件，`head -n 3` 看一下头三行就明白数据结构了。

---

## 只看末尾几行：`tail`

`tail` 跟 `head` 正好相反——只看文件末尾：

```zsh
tail notes.txt
```

也是默认 10 行。`-n` 控制行数：

```zsh
tail -n 5 notes.txt
```

但 `tail` 最闪光的地方是 `-f` 选项——**follow**，实时跟踪：

```zsh
tail -f /var/log/system.log
```

文件末尾如果有新内容写入，它会实时显示出来。调试程序、看日志的时候，这个命令能救命——开着这个窗口，那边程序一跑，这边立刻能看到输出。

按 `Ctrl + C` 退出。

---

## 数行数字数字：`wc`

`wc` 是 **Word Count** 的缩写。不加选项的时候，它会告诉你三件事：

```zsh
wc notes.txt
```

```
  2   6  39 notes.txt
```

三列依次是：**行数**、**单词数**、**字节数**。

只要行数的话：

```zsh
wc -l notes.txt
```

```
2 notes.txt
```

想知道代码仓库里有多少行代码？一行搞定：

```zsh
find . -name "*.py" | xargs wc -l
```

---

## 文本搜索之王：`grep`

终于到了 `grep`。

如果你只学一个命令行文本处理工具，学 `grep`。它是**全局正则搜索**（Global Regular Expression Print）的缩写，名字听起来很唬人，但做的事很简单——**在一堆文字里找匹配的行**。

最基本用法：

```zsh
grep "关键词" 文件名
```

比如在日志里搜 "error"：

```zsh
grep "error" system.log
```

```
2026-06-28 10:00:01 [ERROR] connection timeout
2026-06-28 10:00:05 [ERROR] failed to connect
```

所有包含 "error" 的行都被列出来了。

行号也想要的话，加 `-n`：

```zsh
grep -n "error" system.log
```

```
12: 2026-06-28 10:00:01 [ERROR] connection timeout
27: 2026-06-28 10:00:05 [ERROR] failed to connect
```

---

### 不区分大小写：`-i`

默认 `grep` 是区分大小写的。`"Error"` 和 `"error"` 是两回事。加 `-i` 就不分了：

```zsh
grep -i "error" system.log
```

会搜到 `ERROR`、`Error`、`error` 所有变体。

---

### 反向匹配：`-v`

有时候你想看的不是匹配的行，而是**不匹配**的行。比如过滤掉注释行：

```zsh
grep -v "^#" config.ini
```

`^#` 表示以 `#` 开头的行，`-v` 就是排除它们。这样配置文件里的注释就全被过滤掉了，只看有效配置。

---

### 递归搜索整个目录：`-r`

如果你不知道关键词在哪个文件里，让 `grep` 自己翻：

```zsh
grep -r "TODO" ~/projects/
```

递归搜索 `~/projects/` 下所有文件，找出哪些文件里含有 TODO。这个在接手别人代码的时候特别有用——看看还有哪些没干完的活。

加上 `-l` 就只看文件名，不显示匹配的内容本身：

```zsh
grep -rl "TODO" ~/projects/
```

只看哪些文件有 TODO，不看具体内容。

---

### 只匹配整个单词：`-w`

搜 `"cat"` 的时候，`grep` 会把 `catalog`、`category`、`concatenate` 里的 `cat` 也匹配出来。加 `-w`（word）就能只匹配完整单词：

```zsh
grep -w "cat" notes.txt
```

---

### 统计匹配行数：`-c`

不想看内容，只想知道有多少行匹配：

```zsh
grep -c "error" system.log
```

```
15
```

有 15 行包含 "error"。

---

### 实用组合

这几个选项可以组合起来用，效果翻倍：

```zsh
grep -rin "password" ~/config/
```

不区分大小写、显示行号、递归搜索——三秒内翻遍整个目录找到所有跟密码相关的配置。

---

## `grep` 配合管道

`grep` 真正的威力在于跟管道 `|` 配合。管道可以把一个命令的输出「喂」给另一个命令：

```zsh
ps aux | grep "python"
```

列出所有进程，只留下包含 "python" 的。想找某个程序有没有在运行，这个是最快的办法。

再配合 `head`：

```zsh
dmesg | grep "error" | head -n 5
```

从系统日志里搜出所有错误，只看前 5 条。

`wc -l` 也是一样：

```zsh
history | grep "git" | wc -l
```

看看自己今天敲了多少次 git 命令。数字大说明在频繁提交，数字小说明该努力了。

---

## 动手试试

找一个文件多的目录，或者直接用之前的练习目录（没有就新建一个）：

```zsh
cd ~/cmd_practice 2>/dev/null || mkdir ~/cmd_practice && cd ~/cmd_practice

# 创建几个有内容的文件
echo "Line 1: hello" > test1.txt
echo "Line 2: world" >> test1.txt
echo "Line 3: hello again" >> test1.txt
echo "APPLE" > fruit.txt
echo "banana" >> fruit.txt
echo "CHERRY" >> fruit.txt
echo "apple pie" > recipe.txt

# 看文件
cat test1.txt
head -n 2 test1.txt
tail -n 1 test1.txt

# 搜关键词
grep "hello" test1.txt
grep -i "apple" fruit.txt recipe.txt
grep -c "apple" fruit.txt

# 管道练习
cat test1.txt | grep "hello" | head -n 1
cat fruit.txt recipe.txt | grep -i "apple" | wc -l
```

试试不同的 `grep` 选项组合，感受一下怎么快速从一堆文本里捞出自己想要的内容。

---

## 小结

| 命令 | 作用 | 常用搭档 |
|------|------|----------|
| `cat` | 看文件内容 | `-n` 显示行号，`cat a b > c` 合并文件 |
| `head` | 看文件开头 | `-n 行数` |
| `tail` | 看文件末尾 | `-f` 实时跟踪日志 |
| `wc` | 数行数/字数/字节数 | `-l` 只看行数 |
| `grep` | 搜索文本 | `-i`（忽略大小写）、`-r`（递归）、`-n`（行号）、`-v`（反向）、`-w`（整词）、`-c`（计数） |
| `\|` 管道 | 把上一条命令的输出传给下一条 | 跟 `grep`、`wc`、`head`、`tail` 组合使用 |

`grep` 的选项看着多，其实记三个就够了：`-i`、`-r`、`-n`。剩下的遇到了再查。

下一篇我们聊文件权限——为什么有些文件能看不能改，`chmod` 到底在干什么。

---

> [← 上一篇：文件操作：创建、读取、编辑、删除]({% post_url 2026-06-28-file-crud %})  
> 下一篇：文件权限——谁可以读、写、执行
