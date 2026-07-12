---
layout: single
title:  "从零开始学命令行：alias 和 Shell 函数——给命令起外号"
date:   2026-07-12 14:00:00 +0800
categories:
  - Command Line
tags:
  - 命令行
  - alias
  - 函数
  - 效率
  - zshrc
---

有些命令组合我们天天敲——比如 `ls -la`、`git status`、`ssh myserver`。每次都敲全太烦了。能不能缩短？

当然能。两个方法：**alias** 和 **Shell 函数**。

---

## alias：最简单的缩写

```zsh
alias ll='ls -la'
alias gs='git status'
alias ..='cd ..'
```

设置之后，敲 `ll` 就等于敲 `ls -la`，敲 `..` 就等于 `cd ..`。

查看当前所有 alias：

```zsh
alias
```

临时跳过 alias（用原生命令）：

```zsh
\ls           # 跳过 ls 的 alias
command ls    # 同上，显式调用原生命令
```

### 删除 alias：unalias

加了 alias 之后想取消？用 `unalias`：

```zsh
unalias ll
```

查看所有 alias 时发现某个不想要了，直接 `unalias <名字>` 就行：

```zsh
unalias ll gs     # 一次删多个
unalias -a         # 删除所有 alias（小心用）
```

### 单引号还是双引号？

定义 alias 时引号的选择有讲究：

```zsh
alias foo="echo $HOME"   # 双引号——定义时 $HOME 就展开成路径了
alias bar='echo $HOME'    # 单引号——每次敲 bar 时才展开 $HOME
```

绝大多数情况下用**单引号**更安全，因为我们不希望定义的时候变量就被求值。拿不准的时候就写单引号。

### 参数呢？

alias 有一个局限：**不能接受参数**。如果我们写：

```zsh
alias mkcd='mkdir $1 && cd $1'
```

然后敲 `mkcd mydir`，Shell 会把命令展开成这样：

```text
mkdir $1 && cd $1 mydir
```

交互式 Shell 里 `$1` 是空的，实际执行的是：

```text
mkdir && cd mydir
```

`mkdir` 没给参数会报使用错误，而 `mydir` 被追加到了末尾而不是填入 `$1`——整个命令逻辑全乱了。

**alias 只是纯文本替换**，我们跟在 alias 后面的参数会追加在替换结果的末尾，不会填充到 `$1`、`$2` 这些位置参数里。想要传参，得用函数。

这就引出了 Shell 函数。

---

## Shell 函数：带参数的自定义命令

```zsh
mkcd() {
  mkdir -p "$1" && cd "$1"
}
```

用的时候：

```zsh
mkcd mydir
# 创建 mydir 目录并进去
```

函数可以接受多个参数，可以做判断，可以写循环——它就是一段轻量级的 Shell 脚本。

> 也可能见到有人用 `function 名字 { ... }` 写函数定义，那是旧式 Bourne shell 的语法，zsh 和 bash 也支持。`名字() { ... }` 的写法最通用，POSIX 标准也认它。

### 实用函数示例

```zsh
# 解压——不记参数，一个命令搞定
extract() {
  if [ -f "$1" ]; then
    case "$1" in
      *.tar.gz)  tar -xzf "$1" ;;
      *.zip)     unzip "$1" ;;
      *.rar)     unrar x "$1" ;;
      *)         echo "不知道这个格式" ;;
    esac
  fi
}

# 快速打开文件并进入目录
cdvim() {
  cd "$(dirname "$1")" && vim "$(basename "$1")"
}
```

### 删除函数：unset -f

和 alias 一样，函数也可以删：

```zsh
unset -f mkcd
```

```zsh
unset -f mkcd extract    # 一次删多个
```

`-f` 表示操作的是函数（function），不加 `-f` 的话 `unset` 默认操作的是变量。

---

## 存在哪里：`~/.zshrc`

alias 和函数都要写到配置文件里才能每次打开终端自动生效。

```zsh
# ~/.zshrc

# alias
alias ll='ls -la'
alias gs='git status'
alias ..='cd ..'

# 函数
mkcd() {
  mkdir -p "$1" && cd "$1"
}

extract() {
  # ...
}
```

改完之后 `source ~/.zshrc` 让它立刻生效。

### 一个提示：不要贪多

刚学 alias 的时候特别容易沉迷——什么都想搞个缩写。但设多了反而记不住，最后常用的还是那五六个。

比较好的做法是：**发现一个命令敲了三遍以上，再考虑给它加 alias**。自然生长出来的 alias 列表才真正好用。

---

## 别名和函数选哪个

| | alias | 函数 |
|--|------|------|
| 适用场景 | 简单的缩写 | 需要参数的逻辑 |
| 接受参数 | ❌ | ✅ |
| 复杂逻辑 | ❌ | ✅ |
| 定义语法 | `alias x='命令'` | `x() { 命令; }` |
| 性能 | 略快 | 略慢（几乎感觉不到） |

经验法则：**两个词以内用 alias，两个词以上或有参数用函数。**

---

## 动手试试

```zsh
# 临时设置 alias（退出终端就没了）
alias ll='ls -la'
ll

# 临时定义函数
mkcd() { mkdir -p "$1" && cd "$1"; }
mkcd ~/cmd_practice/test-alias
pwd
cd ..
rmdir ~/cmd_practice/test-alias

# 查看当前 alias
alias

# 看看哪些 alias 已经设好了
alias | grep git

# 删掉刚才设的 alias 和函数
unalias ll
unset -f mkcd
```

---

## 小结

- **alias** = 简单的字符串替换，适合固定的缩写
- **Shell 函数** = 带参数和逻辑的自定义命令
- 都写在 `~/.zshrc` 里才持久生效
- 不要一口气设太多，让 alias 列表自然生长

我们后面讲 Shell 脚本的时候会深入函数的更多用法。这里先掌握最基础的——能给自己造几个顺手的命令，日常效率就会高一个档次。

---

> [← 查看系列目录]({% link _pages/series-command-line.md %})
