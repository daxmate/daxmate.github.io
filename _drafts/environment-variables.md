---
layout: single
title:  "从零开始学命令行：环境变量"
date:   2026-07-09 09:00:00 +0800
categories:
  - Command Line
tags:
  - 命令行
  - 环境变量
  - PATH
  - export
  - zshrc
---

装了 Node.js，但终端里敲 `node` 说找不到。手动改了路径，关掉终端再打开又不行了。

这些问题的根，都在环境变量。

---

## 什么是环境变量

环境变量就是在操作系统中存储的一些键值对，用来告诉系统和程序「当前的运行环境是什么样的」。最简单的理解：**它们是给程序看的全局配置**。

```zsh
echo $HOME
```

```
/Users/dax
```

`$HOME` 是一个环境变量，存的是当前用户的主目录路径。前面加 `$` 表示取它的值。

---

## 查看环境变量

### 看某一个

```zsh
echo $HOME
echo $USER
echo $SHELL
```

### 看全部

```zsh
env
```

会输出一长串，每个程序启动时都会继承这些变量。

```zsh
env | grep HOME
```

只看跟 `HOME` 有关的。

---

## 最重要的环境变量：`PATH`

`PATH` 决定了**当你敲一个命令时，Shell 去哪找对应的程序**。

```zsh
echo $PATH
```

```
/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
```

这是一串用 `:` 分隔的目录路径。当你敲 `git` 的时候，Shell 按顺序在这串目录里找 `git` 这个可执行文件。第一个找到的就被执行。

**`which` 查的就是这个**：

```zsh
which git
```

```
/usr/bin/git
```

意思是：在 `PATH` 的目录列表里，最先找到 `git` 的地方是 `/usr/bin`。

### 为什么装了软件但命令找不到

一个常见的情况：你用 Homebrew 装了某个工具，但 `which` 找不到它。

原因是：Homebrew 安装在 `/opt/homebrew/bin/`（Apple Silicon）或 `/usr/local/bin/`（Intel），但这个路径没在 `PATH` 里。

换句话说，Shell 根本不知道要去那里找。

### 怎么修

把目录加到 `PATH` 里：

```zsh
export PATH="/opt/homebrew/bin:$PATH"
```

`$PATH` 放在最后，意思是：先在 `/opt/homebrew/bin` 里找，找不到再去原来的目录找。

`export` 的意思是把这个变量**传给子进程**——不加 `export` 的话，只有当前 shell 能看到，你启动的其他程序看不到。

---

## 设置环境变量

### 临时设置（只在当前终端有效）

```zsh
export MY_VAR="hello"
echo $MY_VAR
```

```
hello
```

关掉终端重开，`MY_VAR` 就没了。

### 永久设置：写到配置文件

想让变量每次打开终端都生效，写到 `~/.zshrc`（zsh 的启动配置文件）：

```zsh
echo 'export MY_VAR="hello"' >> ~/.zshrc
source ~/.zshrc   # 让刚才的修改立即生效，不用重开终端
```

`source` 的意思是「在当前 Shell 里执行这个文件里的所有命令」。相当于把文件内容逐行敲一遍。

---

## 常用环境变量

| 变量 | 含义 | 例子 |
|------|------|------|
| `HOME` | 当前用户主目录 | `/Users/dax` |
| `USER` | 当前用户名 | `dax` |
| `SHELL` | 当前 Shell 路径 | `/bin/zsh` |
| `PATH` | 命令搜索路径 | `/usr/bin:...` |
| `PWD` | 当前工作目录 | `/Users/dax/codes` |
| `LANG` | 语言和编码 | `zh_CN.UTF-8` |
| `EDITOR` | 默认文本编辑器 | `vim` 或 `nano` |

`EDITOR` 是个好例子——`git commit` 不指定编辑器的时候，它会弹哪个编辑器？就看你 `EDITOR` 设的是什么。

---

## 查看和修改 PATH 的完整示例

```zsh
# 1. 看看现在有哪些
echo $PATH

# 2. 看看某个目录在不在里面
echo $PATH | grep "homebrew"

# 3. 如果没有，加到 ~/.zshrc
echo 'export PATH="/opt/homebrew/bin:$PATH"' >> ~/.zshrc

# 4. 生效
source ~/.zshrc

# 5. 验证
echo $PATH
```

---

## `.zshrc` vs `.zprofile`：该写哪个

zsh 有几个启动文件，作用不同：

| 文件 | 什么时候执行 | 适合放什么 |
|------|------------|-----------|
| `~/.zshrc` | 每次打开交互式 Shell（终端窗口） | alias、PATH、prompt 设置 |
| `~/.zprofile` | 登录时执行一次 | 全局环境变量、SSH key 加载 |
| `~/.zshenv` | 任何 zsh 启动都执行 | 极少数情况用 |

日常最常用的是 `~/.zshrc`。除非你明确知道某个变量需要「登录时跑一次就好」，否则放 `~/.zshrc` 不会错。

---

## 动手试试

```zsh
# 查看
echo $HOME
echo $USER
echo $SHELL
echo $PATH

# 临时设置一个变量
export MY_TEST="hello world"
echo $MY_TEST

# 看看 env 列表
env | head -n 10

# 让子进程也能看到
zsh -c 'echo $MY_TEST'   # 新开一个 zsh，看能不能读到 MY_TEST
```

---

## 小结

| 操作 | 命令 |
|------|------|
| 查看变量值 | `echo $变量名` |
| 查看全部 | `env` |
| 临时设置 | `export 变量=值` |
| 永久设置 | 写入 `~/.zshrc` 然后 `source ~/.zshrc` |
| PATH 追加 | `export PATH="/新路径:$PATH"` |

**核心认知**：`PATH` 是 Shell 找命令的地图。装了软件找不到命令，十有八九是 `PATH` 里没它。

下一篇终于要开始写脚本了——不是那种复杂的，就是把你常用的几条命令写成一个文件，一个名字就能跑。

---

> [← 查看系列目录]({% link _pages/series-command-line.md %})
