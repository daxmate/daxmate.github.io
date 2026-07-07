---
layout: single
title:  "从零开始学命令行：通配符——Shell 的模式匹配"
date:   2026-07-07 13:00:00 +0800
categories:
  - Command Line
tags:
  - 命令行
  - 通配符
  - glob
  - 文件匹配
---

用过 `ls *.txt` 或者 `rm -rf *` 吧？那我们已经在用通配符了。可能没注意过它，但它一直在默默工作。

通配符，也叫 globbing（来自 global 这个词的缩写），是 Shell 在**执行命令之前**做的一件事情——把匹配模式展开成实际的文件名列表。

---

## 通配符什么时候发生的

先搞懂一个关键的时间点：通配符展开是 Shell 干的，不是命令本身干的。

```zsh
echo *.txt
```

Shell 先看到 `*.txt`，在当前目录下找到所有以 `.txt` 结尾的文件，把 `*.txt` 替换成这些文件名，然后才把替换后的结果传给 `echo`。

所以如果目录里有 `a.txt`、`b.txt`、`c.txt`，Shell 看到的实际命令是：

```zsh
echo a.txt b.txt c.txt
```

```zsh
ls *.txt
```

同理——Shell 先展开，再传给 `ls`。`ls` 收到的是一堆文件名，它根本不知道 `*` 的存在。

---

## 三个基础通配符

### `*` — 匹配任意字符（包括零个）

```zsh
# 所有 .txt 文件
ls *.txt

# 所有以 "file" 开头的文件
ls file*

# 所有包含 "data" 的文件
ls *data*
```

`*` 是最常用的通配符，但注意它**不匹配以 `.` 开头的隐藏文件**：

```zsh
ls *
```

`.zshrc`、`.git` 这些不会被列出来。想匹配隐藏文件得显式写 `.*`：

```zsh
ls .*      # 所有隐藏文件
ls .*      # 注意：也包括 . 和 ..
```

`.` 和 `..` 是特殊目录，`ls .*` 会顺便把它们也列出来。用 `ls` 的时候无所谓，但用 `rm -rf .*` 的时候就是个灾难——它会删掉当前目录的父目录。所以对 `.*` 做危险操作的时候要格外小心。

### `?` — 匹配单个字符

```zsh
# 匹配 file1.txt、file2.txt……但不匹配 file10.txt
ls file?.txt

# 匹配三个字符的文件名
ls ???
```

`?` 严格匹配一个字符，不多不少。

### `[...]` — 匹配字符集合

```zsh
# 匹配 a.txt 或 b.txt
ls [ab].txt

# 匹配 0-9 范围内的数字
ls file[0-9].txt

# 匹配字母（大小写都行）
ls [a-zA-Z].txt

# 取反——匹配不是 a 或 b 的
ls [^ab].txt
```

方括号里支持范围（`a-z`、`0-9`）和取反（`^` 或 `!`）。

---

## 组合使用

```zsh
# 所有 .jpg 和 .png 文件
ls *.{jpg,png}

# 以 a 到 e 开头、后面跟两个字符、后缀 .txt
ls [a-e]??.txt
```

大括号 `{...}` 严格来说不是 glob 语法，是 Shell 的另一种展开机制——大括号展开（brace expansion），它在 glob 展开之前发生。比如 `{jpg,png}` 会展开成 `jpg png`，然后再跟 `*.` 拼成 `*.jpg *.png`。不过它跟通配符经常一起用，放在一起说更顺。

---

## 如果没匹配到

通配符什么都没匹配到的时候，不同 Shell 的处理方式不一样：

```zsh
echo *.nonexistent
```

- **bash**：把 `*.nonexistent` 当成普通字符串输出——如果当前目录没有 `.nonexistent` 结尾的文件，会看到 `*.nonexistent` 本身
- **zsh**：直接报错——`zsh: no matches found: *.nonexistent`

这其实是 zsh 的一个保护机制——宁可报错，也不让我们在没匹配到的时候误操作。

---

## 扩展通配符

除了基础的几个，zsh 还提供了一组扩展模式匹配。先列一下，后面用到的时候再展开：

| 语法 | 作用 | 示例 |
|------|------|------|
| `**/*.txt` | 递归匹配所有子目录 | 当前目录及所有子目录中的 .txt 文件 |
| `^pattern` | 排除匹配 | `ls ^*.txt` 显示非 .txt 文件 |
| `pattern~pattern` | 排除特定模式 | `ls *.txt~readme*` 显示除了 readme 开头的 txt |

关于 `**` 多说一句：在 bash 里它默认不开启，需要 `shopt -s globstar`。在 zsh 里默认就能用。

---

## 实战场景

### 场景 1：批量删除日志

```zsh
# 删除所有 .log 文件（不包括子目录）
rm *.log

# 删除所有子目录下的 .log 文件
rm **/*.log

# 删除之前先确认一下
ls **/*.log
```

### 场景 2：只保留某些文件

```zsh
# 把除了 .md 以外的文件移到另一个目录
mv ^*.md ~/temp/
```

### 场景 3：处理一组特定文件

```zsh
# 用 vim 打开所有第 3 章的文件
vim chapter3-*.md
```

### 场景 4：日期或编号范围

```zsh
# 备份配置文件的特定版本
cp .zshrc .zshrc.backup-202[6-7]*
```

---

## 动手试试

```zsh
mkdir -p ~/cmd_practice/glob && cd ~/cmd_practice/glob

# 准备一些测试文件
touch a.txt b.txt c.txt file1.txt file10.txt file2.txt
touch readme.md notes.txt data.csv
touch .hidden.txt
mkdir subdir && touch subdir/deep.txt

# * 通配符
echo *.txt
echo f*
echo *.*

# ? 通配符
echo file?.txt
echo ???.txt

# [] 通配符
echo [a-c].txt
echo file[0-9].txt

# 大括号
echo *.{txt,md}

# 递归匹配
echo **/*.txt

# 看看没匹配到的行为
echo *.nothing

# 清理
cd ..
trash glob
```

---

## 小结

| 通配符 | 作用 | 示例 |
|--------|------|------|
| `*` | 匹配任意字符（含零个） | `*.txt` |
| `?` | 匹配单个字符 | `file?.txt` |
| `[abc]` | 匹配集合中的字符 | `[a-z]*` |
| `[^abc]` | 匹配不在集合中的字符 | `[^0-9]*` |
| `{a,b}` | 大括号展开（不是 glob，但经常一起用） | `*.{jpg,png}` |
| `**` | 递归匹配（zsh 默认，bash 需开启） | `**/*.log` |

通配符是 Shell 在日常交互中使用频率最高的机制之一。我们没有刻意学它，但它一直在——搞懂它，就不会再被那些 `*` 和 `?` 的意外行为困扰了。

---

> [← 查看系列目录]({% link _pages/series-command-line.md %})
