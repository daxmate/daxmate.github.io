---
layout: single
title:  "从零开始学命令行：grep — 扩展正则"
date:   2026-07-05 09:00:00 +0800
categories:
  - Command Line
tags:
  - 命令行
  - grep
  - 正则表达式
  - ERE
---

上篇讲了 `grep` 的基础用法和基本的正则符号。这篇把正则表达式再往前推一步——扩展正则表达式（Extended Regular Expression，简称 ERE）。

上一篇的几个符号（`^`, `$`, `.`, `[...]`, `*`）是基础正则（BRE）的核心。这篇引入的 `+`、`?`、`|`、`()`、`{}` 是扩展正则的武器。

在命令行里，有两种方式启用扩展正则：

1. `grep -E`（推荐）
2. `egrep`（`grep -E` 的别名，老一代人用得多）

---

## 准备工作

```zsh
mkdir -p ~/cmd_practice/grep-ere
cd ~/cmd_practice/grep-ere

cat > log.txt << 'EOF'
2024-01-01 10:00:00 INFO Server started
2024-01-01 10:00:01 DEBUG Loading config
2024-01-01 10:00:05 INFO Listening on port 8080
2024-01-01 10:01:00 WARN Disk usage at 80%
2024-01-01 10:02:00 ERROR Connection refused: 192.168.1.100
2024-01-01 10:03:00 ERROR Timeout: 10.0.0.1
2024-01-01 10:04:00 INFO Request completed in 150ms
2024-01-01 10:05:00 FATAL Out of memory
EOF

cat > phones.txt << 'EOF'
13812345678
13987654321
021-12345678
400-800-1234
abcdefghijk
12345
EOF
```

---

## 一个或多个：`+`

上篇讲了 `*` 表示零次或多次。`+` 表示**一次或多次**——至少出现一次：

```zsh
grep -E "ap+le" test.txt
```

`apple` 匹配（两个 p），`aple` 匹配（一个 p），但 `ale` 不匹配（p 必须出现至少一次）。

实际使用中最常见的场景是匹配时间戳——日志里那些精确到毫秒的数字：

```zsh
grep -E "[0-9]+ms" log.txt
```

```
2024-01-01 10:04:00 INFO Request completed in 150ms
```

`[0-9]+` 匹配一个或多个数字，后面紧跟 `ms`。

---

## 零个或一个：`?`

`?` 表示前面的字符出现**零次或一次**——可有可无：

```zsh
grep -E "colou?r" somefile
```

既能匹配 `color`（美式），也能匹配 `colour`（英式）。`u?` 表示 u 可有可无。

一个经典的实用场景：匹配 `https?`，同时覆盖 `http` 和 `https`：

```zsh
grep -E "https?://" urls.txt
```

---

## 二选一：`|`

`|` 就是"或"：

```zsh
grep -E "ERROR|FATAL" log.txt
```

```
2024-01-01 10:02:00 ERROR Connection refused: 192.168.1.100
2024-01-01 10:03:00 ERROR Timeout: 10.0.0.1
2024-01-01 10:05:00 FATAL Out of memory
```

同时匹配包含 `ERROR` 或 `FATAL` 的行。看日志的时候太实用了——把要紧的信息一把抓出来。

`|` 可以串联多个：

```zsh
grep -E "ERROR|FATAL|WARN" log.txt
```

---

## 分组：`()`

`()` 把多个字符当成一个整体，然后对这个整体用 `+`、`?`、`|`：

```zsh
grep -E "(ERROR|FATAL)" log.txt
```

这跟上面 `ERROR|FATAL` 的效果一样。但配合其他东西的时候就不一样了。比如匹配重复的模式：

```zsh
echo "hahaha" | grep -E "(ha)+"
```

`(ha)+` 匹配 `ha` 重复一次或多次——`ha`、`haha`、`hahaha` 都行。

---

## 精确重复次数：`{}`

`{n}` 精确重复 n 次，`{n,m}` 重复 n 到 m 次：

| 写法 | 含义 |
|------|------|
| `{3}` | 正好 3 次 |
| `{3,}` | 至少 3 次 |
| `{3,5}` | 3 到 5 次 |
| `{,5}` | 最多 5 次 |

匹配手机号（11 位数字）：

```zsh
grep -E "^1[0-9]{10}$" phones.txt
```

```
13812345678
13987654321
```

`1` 开头，后面跟 10 个数字，正好 11 位。

匹配 IP 地址（简单版）：

```zsh
grep -E "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" log.txt
```

```
2024-01-01 10:02:00 ERROR Connection refused: 192.168.1.100
2024-01-01 10:03:00 ERROR Timeout: 10.0.0.1
```

`\.` 是转义的点，匹配字面的 `.`。如果不加 `\`，`.` 会匹配任意字符。

---

## 实战：解析日志

把几个工具组合起来，威力就出来了。

找出某一天的 ERRORS 和 FATALS，并显示行号：

```zsh
grep -E -n "2024-01-01.*(ERROR|FATAL)" log.txt
```

```
6:2024-01-01 10:02:00 ERROR Connection refused: 192.168.1.100
7:2024-01-01 10:03:00 ERROR Timeout: 10.0.0.1
8:2024-01-01 10:05:00 FATAL Out of memory
```

`2024-01-01.*` 匹配日期之后任意内容，然后 `(ERROR|FATAL)`。

配合 `-v` 排除不关心的行：

```zsh
grep -E -v "DEBUG|INFO" log.txt
```

```
2024-01-01 10:01:00 WARN Disk usage at 80%
2024-01-01 10:02:00 ERROR Connection refused: 192.168.1.100
2024-01-01 10:03:00 ERROR Timeout: 10.0.0.1
2024-01-01 10:05:00 FATAL Out of memory
```

只看值得关注的内容。

---

## BRE vs ERE：什么时候用哪个

简单总结：

| 场景 | 用哪个 |
|------|--------|
| 搜一个固定的词 | `grep "word"`，不用正则 |
| 简单的模式（用 `^`, `$`, `.`, `*`, `[]`） | `grep`（基础正则） |
| 需要 `+`, `?`, `|`, `()`, `{}` | `grep -E`（扩展正则） |

如果不确定，直接用 `grep -E` 不会有坏处。

另外，上一篇说过，基础正则里的 `*`, `.` 等都有自己的含义。如果真的要搜这些符号本身，前面加 `\` 转义。但在扩展正则里，`{}` 和 `()` 不需要转义——这就是 BRE 和 ERE 最直观的差别。

---

## 动手试试

```zsh
cd ~/cmd_practice/grep-ere

# + 号：一个或多个
grep -E "[0-9]+ms" log.txt

# | 号：或
grep -E "ERROR|FATAL" log.txt

# 组合：日期 + (ERROR 或 FATAL)
grep -E "2024-01-01.*(ERROR|FATAL)" log.txt

# 排除不关心的
grep -E -v "DEBUG|INFO" log.txt

# IP 地址匹配
grep -E "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" log.txt

# 手机号
grep -E "^1[0-9]{10}$" phones.txt

# 多看一个：匹配重复
echo "hahaha hehe ha" | grep -E -o "(ha)+"
# -o: 只输出匹配的部分
```

---

## 小结

| 符号 | 含义 | 例子 |
|------|------|------|
| `+` | 一次或多次 | `[0-9]+` 匹配一个或多个数字 |
| `?` | 零次或一次 | `https?` 匹配 http 或 https |
| `\|` | 或 | `ERROR\|FATAL` 匹配两者之一 |
| `()` | 分组 | `(ha)+` 匹配 ha 的重复 |
| `{3}` | 正好 3 次 | `[0-9]{3}` 正好三位数字 |
| `{3,}` | 至少 3 次 | `[a-z]{3,}` 至少三个字母 |
| `{3,5}` | 3 到 5 次 | `[0-9]{3,5}` 三到五位数字 |

`grep -E` 是日常文本处理的利器。其实大部分情况用这些就够了，不需要学更复杂的正则。把 `|`（或）和 `()`（分组）搞熟，处理日志和配置文件的时候效率会明显提升。

下一篇我们继续文本处理的路子——`sort`、`uniq`、`cut` 三个轻量命令，专治各种数据整理的日常烦恼。

---

> [← 查看系列目录]({% link _pages/series-command-line.md %})
