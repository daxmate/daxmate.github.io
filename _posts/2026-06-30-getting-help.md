---
layout: single
title:  "从零开始学命令行：获取帮助与认识命令"
date:   2026-06-30 12:53:00 +0800
categories:
  - Command Line
tags:
  - 命令行
  - man
  - help
  - type
  - which
  - whence
  - where
  - run-help
---

到这个阶段，我们已经学了十几个命令了。新问题来了——**这个命令怎么用来着？** 记不住选项怎么办？发现一个不认识的新命令，怎么知道它是干什么的？

这篇就来解决这些问题。学会了怎么查，后面学新东西就不用每次问别人了。

---

## 最权威的文档：`man`

几乎每个正经的命令都自带一本说明书——**手册**。在终端里查手册，用 `man` 命令：

```zsh
man ls
```

屏幕上会显示 `ls` 的完整手册，从名字、用法、所有选项到注意事项，一应俱全。

翻页方式和 `less` 一样：
- **空格**或 **`f`** 翻下一页
- **`b`** 翻上一页
- **`/关键词`** 搜索
- **`q`** 退出

因为 `man` 背后用的就是 `less` 来显示内容，所以操作习惯完全一致。

### man 手册的构成

一份 `man` 手册通常分这么几个部分：

| 章节 | 内容 |
|------|------|
| NAME | 命令名字和一句话简介 |
| SYNOPSIS | 用法格式 |
| DESCRIPTION | 详细说明 |
| OPTIONS | 所有选项逐一解释 |
| EXAMPLES | 使用示例 |
| SEE ALSO | 相关命令 |

拿到一个不熟悉的命令，先看 NAME 知道它干嘛的，再看 SYNOPSIS 知道怎么写，然后去 OPTIONS 找需要的选项。

### 手册分类

`man` 的手册是按编号分卷的。最常用的是第 1 卷——**用户命令**。但还有其他卷：

```zsh
man 1 ls       # 第1卷：用户命令（默认）
man 2 open     # 第2卷：系统调用
man 3 printf   # 第3卷：C 语言库函数
man 5 crontab  # 第5卷：配置文件格式
man 7 pipe     # 第7卷：杂项
```

日常基本只看第 1 卷。有些命令同时出现在多卷里，比如 `crontab` 既是命令（第 1 卷）也有配置文件格式说明（第 5 卷），这时候需要指定卷号：

```zsh
man 5 crontab   # 查看 crontab 的配置文件格式
```

---

## 快速参考：`--help`

`man` 虽好，但内容太多。有时候只是想快速看一眼选项列表——这时候用 `--help` 或 `-h`：

```zsh
ls --help
```

输出简洁得多，就一屏或两屏，列出了最常用的选项和说明。

大部分命令都支持 `--help`，但不是所有。有些命令用 `-h` 而不是 `--help`。可以两个都试试：

```zsh
command --help   # 如果不行
command -h       # 试试这个
```

习惯是：**记不清具体选项时先 `--help`，需要深入了解时再看 `man`**。

---

## zsh 自带帮助：`run-help`

`man` 和 `--help` 都是通用机制，zsh 还有一个自己的帮助系统——`run-help`。

先把它加载进来（这一行可以加到 `~/.zshrc` 里）：

```zsh
autoload run-help
```

然后就能用了：

```zsh
run-help cd
```

`run-help` 跟 `man` 的区别在于它**知道命令是谁家的**。如果是 zsh 内置命令，它会打开 zsh 自己的文档；如果是外部命令，它会直接调 `man`。不用自己判断该看哪种手册。

还可以绑一个快捷键——按 `Alt+H` 直接看当前敲的命令的帮助：

```zsh
bindkey '^[h' run-help
```

之后在终端里敲 `cd` 然后按 `Alt+H`，就能看到 `cd` 的帮助，不用先回车再打 `man`。把上面两行加到 `~/.zshrc` 里就能永久生效。

---

## 不知道命令名字，只知道功能：`apropos`

这是最难的情况——**知道想做什么，但不知道用什么命令**。

理论上有个命令叫 `apropos`（等于 `man -k`），会搜索所有 man 手册的简介，把包含关键词的命令列出来：

```zsh
apropos "compress"
```

但在实际使用中，这个命令在 macOS 上常常搜不出结果——它依赖的索引数据库不一定完整。可以试试先重建索引：

```zsh
sudo /usr/libexec/makewhatis
```

跑完之后再搜，可能会好一些，但也不要抱太高期望。

说实话，初学阶段最靠谱的「搜命令」方式是**直接上网搜**——"macOS command line how to find files" 比 `apropos` 管用得多。`apropos` 知道有这个东西就行，等以后在 Linux 服务器上也许会用到。

---

## 这个命令到底是什么：`type`

Shell 里的命令不止一种来源——有些是 Shell 自带的（**内置命令**），有些是系统里的独立程序（**外部命令**），有些是别名。

`type` 告诉我们一个命令到底是哪种：

```zsh
type cd
```

```
cd is a shell builtin
```

```zsh
type ls
```

```
ls is /bin/ls
```

```zsh
type cat
```

```
cat is /bin/cat
```

如果之前设置过别名，也会显示：

```zsh
type ls
```

```
ls is aliased to `ls --color=auto'
```

这个命令的实用场景是：**当某个命令行为古怪时，先看看它到底是啥**。比如 `rm` 被 alias 成了警告，敲 `type rm` 一看就知道是怎么回事了。

---

## 这个程序在哪：`which`

`which` 回答的问题是：**这个外部命令对应的程序文件，在文件系统的哪个位置？**

```zsh
which python
```

```
/usr/bin/python
```

```zsh
which git
```

```
/usr/bin/git
```

跟 `type` 不同，`which` 只关心命令对应的程序文件在哪里，不展开别名：

```zsh
which cd
```

```
cd: shell built-in command
```

所以 `which` 最常见的用法是：确认某个程序有没有装、装在哪里：

```zsh
which node  && echo "Node.js 已安装" || echo "未安装"
```

---

## zsh 的增强版：`whence` 和 `where`

如果你是 zsh 用户，`type` 和 `which` 各有一个加强版值得了解。

### `whence`：加强版 `type`

`whence` 是 zsh 原生的命令查询工具，比 `type` 更灵活：

```zsh
whence ls
# → /bin/ls

whence -v ls
# → ls is a shell function from ~/.zshrc

whence -c ls
# → 显示实际上会执行的那个命令

whence -w ls
# → command（只告诉你类型，不显示路径）
```

`whence -v` 最实用——它会告诉你**这个命令到底是谁定义的**。如果你装了某个工具但行为不对，用 `whence -v` 看看路径是不是你预期的那一个。

### `where`：一个名字，多个位置

`which` 只告诉你第一个匹配的路径。但有时候一个命令**装了多个版本**：

```zsh
where python
# → /usr/local/bin/python      # Homebrew 装的
# → /usr/bin/python            # 系统自带的
```

哪一个会被执行？靠前的那一个。`where` 把全部路径列出来，一眼就知道优先级。

装了多个 Python、多个 Node 版本、不知道到底在跑其中哪一个的时候——`where` 能帮你理清楚。

---

## 绕过别名：`command`

前面讲过可以用 alias 把 `rm` 改成警告。那如果真的需要执行原本的 `rm` 怎么办？

除了用 `/bin/rm` 全路径之外，还有一个更通用的办法——**`command`**：

```zsh
command rm file.txt
```

`command` 会绕过别名和函数，直接执行命令原本的程序。对内置命令也一样有效：

```zsh
command cd /tmp
```

它比全路径好在不用记程序装在哪里，Shell 自己会去找。

---

## 动手试试

打开终端，试一遍：

```zsh
# 查手册
man ls
man cp

# 快速参考
ls --help
cp --help

# zsh 自带帮助
autoload run-help
run-help cd

# 看命令类型
type cd
type ls
type which

# 找程序路径
which python
which git

# zsh 增强版
whence -v python
where python

# 创建别名再绕过
alias greet='echo "hello"'
type greet
greet
command greet   # 会报错，因为 greet 没有对应的原始命令
unalias greet   # 删除别名
```

把 `man` 和 `--help` 养成习惯——碰到不熟的命令先 `--help`，还看不懂再 `man`。

---

## 小结

| 命令 | 作用 | 适用场景 |
|------|------|----------|
| `man 命令` | 查看完整手册 | 需要详细了解一个命令 |
| `命令 --help` | 查看快速参考 | 选项忘了，看一眼 |
| `run-help 命令` | zsh 自带帮助 | 自动区分内置命令和外部命令，建议绑快捷键 |
| `apropos 关键词` | 按功能搜索命令 | 不常用，macOS 上经常搜不到，不如上网搜 |
| `tldr 命令` | 简化版手册 | 以后再聊，比 `man` 好读得多 |
| `type 命令` | 看命令的类型 | 命令行为异常时排查原因 |
| `which 命令` | 找程序文件路径 | 确认程序是否安装 |
| `whence -v 命令` | zsh 版 `type`，更详细 | 追查命令来源 |
| `where 命令` | 列出命令所有位置 | 装了多个版本时辨清优先级 |
| `command 命令` | 绕过别名执行 | alias 了某个命令但偶尔想用原版 |

> [← 查看系列目录]({% link _pages/series-command-line.md %})
