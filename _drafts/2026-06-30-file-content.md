---
layout: single
title:  "从零开始学命令行(6)--查看文件内容"
date:   2026-06-30 12:00:00 +0800
categories:
  - Command Line
tags:
  - 命令行
  - 文件操作
  - cat
  - head
  - tail
  - less
  - wc
---

前几篇我们学了文件操作的基本功——创建、复制、移动、删除。现在的问题是：文件里有内容了，怎么看？

上一篇我们其实已经用过 `cat` 和 `less` 了，但只是最基本用法。今天把它们展开，再加上 `head`、`tail` 和 `wc`——几个不用动脑、但每天都会用到的查看工具。

---

## `cat` 的更多用法

`cat` 就是把文件内容打印到屏幕上。简单，但有几个变体很实用。

### 显示行号

```zsh
cat -n notes.txt
```

```
     1	This is my readme file
     2	This is a second line.
```

调试配置文件的时候特别好用——想跟别人说「看第 27 行」，直接报行号就行。

### 合并文件

`cat` 的名字来自 **concatenate**（串联），把多个文件串起来输出就是它的本职工作：

```zsh
cat file1.txt file2.txt
```

配合重定向可以合并文件：

```zsh
cat notes.txt diary.txt > combined.txt
```
```zsh
cat notes.txt diary.txt >> combined.txt
```

`>` 是覆盖写入，`>>` 是追加到末尾。上一篇讲过这个区别。

### 在终端里快速创建小文件

有时候想快速写个几句话的文件，不想打开编辑器：

```zsh
cat > quick-note.txt
```

然后直接敲内容，敲完了按 `Ctrl + D` 结束输入。屏幕上看起来是这样的：

```zsh
cat > quick-note.txt
这是一段话。
按 Ctrl+D 结束。
```

按 `Ctrl + D` 之后，内容就写进去了。查看一下：

```zsh
cat quick-note.txt
```

```
这是一段话。
按 Ctrl+D 结束。
```

比 `echo` 一行行 `>>` 方便多了。这个技巧在写脚本的时候经常用。

---

## 只看开头几行：`head`

有时候文件很长，你只想知道它长什么样——看看头几行就够了。

```zsh
head long-file.txt
```

默认显示前 10 行。想多看或少看，加 `-n`：

```zsh
head -n 5 long-file.txt    # 只看前 5 行
head -n 20 long-file.txt   # 只看前 20 行
```

这个命令最适合快速确认文件的格式。拿到一个 CSV 文件，`head -n 3` 看一下头三行就明白数据结构了。

---

## 只看末尾几行：`tail`

`tail` 跟 `head` 正好相反——只看文件末尾：

```zsh
tail long-file.txt
```

也是默认 10 行。`-n` 控制行数：

```zsh
tail -n 5 long-file.txt
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

查一下项目里有多少个代码文件：

```zsh
ls *.py | wc -l
```

一秒知道有多少个 Python 文件。

---

## 把命令串起来：管道 `|`

前面用过几次 `>` 重定向——把命令的输出写到文件里。管道 `|` 类似，但它不是写到文件，而是**传给下一个命令**。

```zsh
cat long-file.txt | head -n 5
```

把 `cat` 输出的内容「喂」给 `head`，`head` 只取前 5 行。

```zsh
cat long-file.txt | wc -l
```

把文件内容传给 `wc -l`，直接数行数。

现在看起来有点多余——`wc -l long-file.txt` 就行了，不用多此一举加 `cat`。但后面学到更多命令之后，管道的威力才会真正显现出来。这里先知道有这个东西，后面会大量用到。

> 初学阶段记住一条：`|` 把左边命令的输出变成右边命令的输入。

---

## 动手试试

找个文件多的目录试试，或者直接新建一个：

```zsh
cd ~/cmd_practice 2>/dev/null || mkdir ~/cmd_practice && cd ~/cmd_practice

# 创建几个有内容的文件
echo "Line 1: hello" > test1.txt
echo "Line 2: world" >> test1.txt
echo "Line 3: hello again" >> test1.txt
echo "Line 4: foo" >> test1.txt
echo "Line 5: bar" >> test1.txt
echo "Line 6: baz" >> test1.txt
echo "Line 7: qux" >> test1.txt
echo "Line 8: quux" >> test1.txt
echo "Line 9: corge" >> test1.txt
echo "Line 10: grault" >> test1.txt
echo "Line 11: garply" >> test1.txt
echo "Line 12: waldo" >> test1.txt

# 看全部
cat test1.txt

# 带行号
cat -n test1.txt

# 只看开头
head test1.txt
head -n 3 test1.txt

# 只看末尾
tail test1.txt
tail -n 3 test1.txt

# 数一下
wc test1.txt
wc -l test1.txt

# 用 | 串起来
cat test1.txt | head -n 5
cat test1.txt | tail -n 3 | wc -l
```

---

## 小结

| 命令 | 作用 | 常用搭档 |
|------|------|----------|
| `cat` | 看文件内容 | `-n` 显示行号；`cat a b > c` 合并文件 |
| `cat > file` | 快速创建文件 | `Ctrl + D` 结束输入 |
| `head` | 看文件开头 | `-n 行数` |
| `tail` | 看文件末尾 | `-f` 实时跟踪日志 |
| `wc` | 数行数/字数/字节数 | `-l` 只看行数 |
| `\|` 管道 | 把输出传给下一个命令 | 跟 `head`、`tail`、`wc` 组合 |

后面还会学到更强大的文件搜索工具——`grep`，它能在文件内容里搜索关键词，还能配合正则表达式做精确匹配。不过那涉及到正则表达式，是个值得单独开一篇来讲的话题。

下一篇我们聊文件权限——为什么有些文件能看不能改，`chmod` 到底在干什么。

---

> [← 上一篇：文件操作：创建、读取、编辑、删除]({% post_url 2026-06-28-file-crud %})
>
> 下一篇：文件权限——谁可以读、写、执行
