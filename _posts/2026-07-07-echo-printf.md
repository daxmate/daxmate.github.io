---
layout: single
title:  "从零开始学命令行：echo 与 printf——输出的学问"
date:   2026-07-07 12:00:00 +0800
categories:
  - Command Line
tags:
  - 命令行
  - echo
  - printf
  - 输出
  - 转义
---

从系列的第一篇开始，我们就一直在用 `echo`。它可能是整个命令行里最常用的命令之一，简单到不需要动脑。

但也正因为太简单了，有些细节容易被忽略。这篇就聊聊 `echo` 的坑和更靠谱的替代 `printf`。

---

## echo 的基本用法

`echo` 的默认行为：输出内容，然后自动换行。

```zsh
echo hello
echo world
```

```
hello
world
```

每次执行 `echo`，末尾都会带一个换行符——你按回车后光标跳到下一行那个效果。

有时候不希望它换行，比如想在原地追加内容：

```zsh
echo -n "正在加载"
echo " ... 完成"
```

```
正在加载 ... 完成
```

`-n` 就是"no newline"——告诉 `echo` 输出完不要追加换行符。这是所有 shell 实现中最一致的选项之一，基本都支持。

但 `echo` 的其他方面就没这么统一了……

---

## echo 的问题

`echo` 在不同 shell 和不同系统上行为不一样。这是个历史遗留问题。

在 zsh 里试试：

```zsh
echo "hello\nworld"
```

```
hello
world
```

zsh 默认会解释转义字符，`\n` 被当成了换行。这叫 **escape interpretation**。

但在 bash 下：

```bash
echo "hello\nworld"
```

```
hello\nworld
```

bash 默认**不解释**转义——`\n` 就被当成两个字面字符。

想让它解释得加 `-e`：

```bash
echo -e "hello\nworld"
```

```
hello
world
```

但问题来了：`-e` 不是 POSIX 标准的一部分。有些系统上 `echo -e` 输出的是 `-e hello\nworld`。

不仅是 `\n`，还有：

```zsh
echo "列1\t列2\t列3"
```

在 zsh 里是制表符分隔，在别的 shell 里可能就变成了字面 `\t`。

**所以 `echo` 的跨 shell 行为是不可靠的。** 这个问题很老了——Unix 老兵们吵了几十年也没统一。

---

## printf：统一的替代方案

`printf` 来自 C 语言的同名函数，行为在所有 shell 上都一致：

```zsh
printf "hello\nworld\n"
```

```
hello
world
```

`\n` 始终是换行，不需要 `-e`，不会因为 shell 不同就改变。

注意一个问题：`printf` **不会自动加换行**。跟 C 语言的 `printf` 一样，我们得自己写 `\n`：

```zsh
printf "hello"      # 没有换行
printf "hello\n"    # 有换行
```

所以 `echo` 的：

```zsh
echo "hello"
```

等价于：

```zsh
printf "hello\n"
```

---

## printf 的格式化能力

这才是 `printf` 真正强大的地方。它的第一个参数是**格式字符串**，后面可以跟多个参数，格式字符串里的 `%s`、`%d`、`%f` 会被替换：

```zsh
printf "用户名: %s\nID: %d\n" "dax" 1001
```

```
用户名: dax
ID: 1001
```

`%s` 是字符串，`%d` 是整数，`%f` 是浮点数。

**对齐输出：**

```zsh
printf "%-10s %s\n" "命令" "说明"
printf "%-10s %s\n" "ls" "列出文件"
printf "%-10s %s\n" "cat" "查看文件"
printf "%-10s %s\n" "grep" "搜索文本"
```

```
命令        说明
ls          列出文件
cat         查看文件
grep        搜索文本
```

`%-10s` 表示"左对齐，占 10 个字符宽度"。表格对齐不用靠猜空格了。

**输出到文件：**

```zsh
printf "error at line %d\n" 42 > log.txt
```

和重定向配合，跟其他命令一样。

---

## 顺便一提：zsh 的 `print`

如果你用的是 zsh，还有一个 `print` 命令。它是 zsh 的内置命令，bash 里没有：

```zsh
print "hello"
print -n "不换行"     # -n 表示不要结尾的换行
print -P "%F{red}红色文字%f"  # -P 可以解释提示符转义
```

`print` 是 zsh 独有的，所以不存在跨 shell 的行为差异问题。它的选项也比 `echo` 更可控（比如 `-n` 在所有版本里都一致生效）。不过这篇聚焦的是通用的输出工具，zsh 特有的就留到以后专门的 zsh 篇再聊。

---

## 日常怎么用

实用原则：

- **纯输出简单内容** → 用 `echo`，方便，zsh 下很稳定
- **需要转义字符或格式控制** → 用 `printf`
- **写脚本要考虑移植性** → 用 `printf`
- `echo` 不适合显示 `-n` 之类以 `-` 开头的内容

边界情况：

```zsh
echo "-n"
```

zsh 下什么都不会输出——`echo` 会把 `-n` 当成选项。

```zsh
printf "%s\n" "-n"
```

```
-n
```

`printf` 没有这个问题。

---

## 动手试试

```zsh
# echo 的 zsh 行为
echo "hello\nworld"
echo "a\tb\tc"

# printf 的基础
printf "hello\nworld\n"
printf "a\tb\tc\n"

# 格式化
printf "| %-15s | %5d |\n" "苹果" 5
printf "| %-15s | %5d |\n" "香蕉" 12
printf "| %-15s | %5d |\n" "菠萝" 3
```

---

## 小结

| 场景 | 用什么 |
|------|--------|
| 随便看看输出 | `echo` |
| 需要换行/制表符 | `printf` |
| 写脚本（跨 shell） | `printf` |
| 对齐表格 | `printf` 格式化 |
| 输出以 `-` 开头的内容 | `printf` |

`echo` 是日常顺手用的便利工具，`printf` 是"正式场合"的可靠替代。

---

> [← 查看系列目录]({% link _pages/series-command-line.md %})
