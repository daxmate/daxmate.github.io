---
layout: single
title:  "从零开始学命令行：grep — 基础搜索"
date:   2026-07-01 14:00:00 +0800
categories:
  - Command Line
tags:
  - 命令行
  - grep
  - 正则表达式
  - 文本搜索
---

前面用 `cat`、`head`、`tail` 查看文件内容，用 `find` 搜文件名。但如果要在文件**内容**里搜关键词——比如在某一个日志文件里找到所有包含 "error" 的行——就需要 `grep` 了。

`grep` 的名字来自 `ed` 编辑器里的一个命令：**g**lobally search a **r**egular **e**xpression and **p**rint。翻译过来就是「全局搜索正则表达式并打印」。听上去复杂，用起来很简单。

---

## 准备工作

```zsh
mkdir -p ~/cmd_practice/grep-demo
cd ~/cmd_practice/grep-demo

cat > fruits.txt << 'EOF'
apple
banana
Apple pie
grape
pineapple
orange
PEAR
watermelon
EOF
```

---

## 最简单的搜索

```zsh
grep "apple" fruits.txt
```

```
apple
pineapple
```

`grep` 找到所有包含 "apple" 的行并打印出来。注意它匹配的是**子串**——`pineapple` 也包含 "apple"，所以也被找出来了。

### 大小写

默认是区分大小写的。`Apple` 没被匹配到（首字母大写），`PEAR` 也不会被匹配。如果要忽略大小写：

```zsh
grep -i "apple" fruits.txt
```

```
apple
Apple pie
pineapple
```

`-i` 是 ignore case。

---

## 常用选项

### 显示行号：`-n`

```zsh
grep -n "apple" fruits.txt
```

```
1:apple
5:pineapple
```

### 统计数量：`-c`

```zsh
grep -c "apple" fruits.txt
```

```
2
```

不显示匹配的内容，只告诉你有几行。

### 反过来——不匹配的：`-v`

```zsh
grep -v "apple" fruits.txt
```

```
banana
Apple pie
grape
orange
PEAR
watermelon
```

`-v` 是 invert（反转），显示**不包含**关键词的行。这在过滤数据的时候特别好用——比如看日志的时候排除掉所有 `INFO` 行。

### 精确匹配整个词：`-w`

```zsh
grep -w "apple" fruits.txt
```

```
apple
```

`-w` 匹配完整的单词，`pineapple` 不会被匹配（因为 "apple" 在 `pineapple` 里不是独立单词）。

### 只显示文件名：`-l`

搜索多个文件时，只列出**包含匹配**的文件名，不显示具体内容：

```zsh
grep -l "apple" *.txt
```

---

## 正则表达式基础

`grep` 真正的威力在于正则表达式。正则表达式是一种描述文本模式的语法——看起来像天书，但其实几个基础规则挺容易掌握的。

### 行首 `^` 和行尾 `$`

```zsh
grep "^apple" fruits.txt   # 以 apple 开头的行
```

```
apple
```

```zsh
grep "e$" fruits.txt       # 以 e 结尾的行
```

```
apple
Apple pie
grape
pineapple
orange
```

### 任意字符 `.`

`.` 匹配**任意一个**字符：

```zsh
grep "a.e" fruits.txt
```

```
grape
```

`a.e` 匹配 a + 任意一个字符 + e。`grape` 里的 `ape` 符合，`apple` 里的 `app` 不符合（中间有两个字符）。

### 字符集合 `[...]`

`[abc]` 匹配 a、b、c 中的任意一个字符：

```zsh
grep "[gp]rape" fruits.txt
```

```
grape
# pineapple 里的 "eapple" 不符合，因为 "rape" 前面不是 g 或 p
```

```zsh
grep "[aeiou]pple" fruits.txt   # 元音 + pple
```

```
apple
# pineapple 里的 "eapple" 不符合
```

`[a-z]` 表示 a 到 z 之间的任意一个字符，`[0-9]` 表示任意一个数字。

### 重复 `*`

`*` 表示前面的字符可以出现**零次或多次**：

```zsh
grep "ap*le" fruits.txt
```

```
apple        # p 出现 2 次
pineapple    # p 出现 1 次
```

`ap*le`：a + 零个或多个 p + le。

### 转义特殊字符

如果真的要搜 `.` 这个字面字符（而不是任意字符），前面加 `\`：

```zsh
grep "hello\.txt" somefile
```

`.`, `*`, `[`, `^`, `$` 都是正则里的特殊字符，匹配它们本身的时候要加 `\`。

---

## 在多个文件和目录中搜

搜当前目录下所有 `.txt` 文件：

```zsh
grep "apple" *.txt
```

递归搜整个目录（包括子目录）：

```zsh
grep -r "apple" ~/cmd_practice/
```

`-r` 是递归（recursive）。搜索代码库的时候这个选项是标配。

排除某些目录：

```zsh
grep -r "keyword" . --exclude-dir=node_modules
```

---

## 配合管道

`grep` 最常见的使用方式是配合管道 `|`，过滤其他命令的输出：

```zsh
ls -la | grep ".txt"        # 只显示 .txt 文件
ps aux | grep "python"      # 只看 python 进程
history | grep "git"        # 在历史里搜 git 命令
```

这些在后面会大量用到，先混个脸熟。

---

## 动手试试

```zsh
cd ~/cmd_practice/grep-demo

# 基础搜索
grep "apple" fruits.txt
grep -i "apple" fruits.txt
grep -n "apple" fruits.txt
grep -c "apple" fruits.txt
grep -v "apple" fruits.txt
grep -w "apple" fruits.txt

# 正则表达式
grep "^apple" fruits.txt
grep "e$" fruits.txt
grep "a.e" fruits.txt
grep "[gp]rape" fruits.txt
grep "ap*le" fruits.txt

# 递归搜索
grep -r "hello" ~/cmd_practice/
```

---

## 小结

| 选项 | 作用 |
|------|------|
| `grep "关键词" 文件` | 搜索匹配的行 |
| `-i` | 忽略大小写 |
| `-n` | 显示行号 |
| `-c` | 统计匹配行数 |
| `-v` | 反转匹配 |
| `-w` | 匹配完整单词 |
| `-l` | 只显示文件名 |
| `-r` | 递归搜索目录 |

正则基础：

| 符号 | 含义 |
|------|------|
| `^` | 行首 |
| `$` | 行尾 |
| `.` | 任意一个字符 |
| `[abc]` | a、b、c 中的任意一个 |
| `[a-z]` | a 到 z 的任意一个 |
| `*` | 前面的字符出现零次或多次 |
| `\` | 转义特殊字符 |

这几条看起来不多，但日常 80% 的搜索需求都能覆盖了。下一篇讲 `grep -E`（扩展正则），引入 `+`、`?`、`|`、`()` 这些更强大的模式——搜起东西来更灵活。

---

> [← 查看系列目录]({% link _pages/series-command-line.md %})
