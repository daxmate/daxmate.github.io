---
layout: single
title:  "从零开始学命令行：管道、重定向与 xargs"
date:   2026-07-08 09:00:00 +0800
categories:
  - Command Line
tags:
  - 命令行
  - 管道
  - 重定向
  - xargs
  - stdin
  - stdout
---

前面零零散散用过很多次 `>`、`>>`、`|` 了。这篇把它们系统地讲一遍——不只是「怎么用」，更重要的「为什么这样设计」。

理解这几个机制，你对命令行的认知会从「记住一个个命令」升级到「知道怎么把命令组合起来」。这是命令行哲学的精华。

---

## 三个标准流

Unix 的设计里，每个程序默认都有三个「通道」：

| 编号 | 名字 | 全称 | 干什么的 |
|------|------|------|----------|
| 0 | stdin | 标准输入 | 程序从哪读数据（默认：键盘） |
| 1 | stdout | 标准输出 | 程序把正常结果输出到哪（默认：屏幕） |
| 2 | stderr | 标准错误 | 程序把错误信息输出到哪（默认：屏幕） |

用图形理解：

```
键盘 ──→ stdin ──→ [程序] ──→ stdout ──→ 屏幕
                        └──→ stderr ──→ 屏幕
```

重定向和管道，本质就是在**改变这些通道的走向**。

---

## 输出重定向：`>` 和 `>>`

### `>` 覆盖写入

```zsh
echo "hello" > output.txt
```

`echo` 本来输出到屏幕，`>` 让它输出到 `output.txt`。文件以前的内容会被清空。

```zsh
cat output.txt
```

```
hello
```

### `>>` 追加写入

```zsh
echo "world" >> output.txt
cat output.txt
```

```
hello
world
```

`>` 覆盖，`>>` 追加。这个区别前面用过很多次了。

### 只重定向 stderr

有时候需要把错误信息存下来分析，但正常输出继续看。用 `2>`：

```zsh
ls nonexistent.txt 2> error.log
```

屏幕什么都不显示（因为没有正常输出），错误信息写进了 `error.log`。

```zsh
cat error.log
```

```
ls: nonexistent.txt: No such file or directory
```

`2>` 里的 `2` 是 stderr 的编号。

### stdout 和 stderr 都写到同一个文件

```zsh
command > output.txt 2>&1
```

`2>&1` 的意思是：让 stderr（2）的输出流向跟 stdout（1）一样——都到 `output.txt`。

这种写法在写脚本的时候很常见——把所有输出都存到日志文件里，方便排查问题。

---

## `/dev/null`：黑洞

想把输出扔掉（不显示也不保存），扔到 `/dev/null`：

```zsh
command > /dev/null 2>&1
```

不管输出什么，都丢进黑洞。安静跑，什么都不显示。

---

## 输入重定向：`<`

```zsh
cat < input.txt
```

把文件内容「喂」给 `cat` 作为 stdin。跟 `cat input.txt` 效果一样——`cat` 本来就接受文件名参数。

`<` 更多用在那些**不接受文件名参数、只接受 stdin** 的命令上：

```zsh
grep "error" < log.txt
```

跟 `grep "error" log.txt` 的区别肉眼看不出来。但背后机制不一样：前者是把文件内容通过 stdin 传给 `grep`，后者是把文件名作为参数传给 `grep` 让它自己读。这两种读取方式在效率上没有显著差异，但遇到「某个命令只接受 stdin 不接受文件名参数」的时候，`<` 就是解决方案。

---

## `<<`：Here Document

把一段多行文本直接嵌在命令里：

```zsh
cat << 'EOF' > config.txt
server {
    host: localhost
    port: 8080
}
EOF
```

从 `<< 'EOF'` 到 `EOF` 之间的所有内容作为 stdin。这个在前面准备测试数据的时候用过好几次了，写脚本的时候也经常出现。

---

## 管道：`|`

`|` 把一个命令的 stdout 直接连到另一个命令的 stdin：

```
[程序A] stdout ──→ stdin [程序B]
```

```zsh
cat access.log | grep "ERROR" | sort | uniq -c | sort -rn
```

五个命令串成一条线：

1. `cat` 读取日志文件
2. `grep` 筛出 ERROR 行
3. `sort` 排序
4. `uniq -c` 统计次数
5. `sort -rn` 按次数倒排

每一步只做一件事，串起来能干大事。这就是 Unix 哲学。

---

## 管道不给力的时候：`xargs`

管道有个限制：**`|` 只能传 stdout 到 stdin**。如果下一个命令不接受 stdin，只接受**命令行参数**，管道就传不过去了。

举个例子：

```zsh
echo "hello.txt" | cat
```

能行——`cat` 同时接受 stdin 和文件名参数。

但：

```zsh
echo "hello.txt" | rm
```

不行——`rm` **不接受 stdin**。它不理会管道传过来的内容，只能接受命令行参数：`rm hello.txt`。

`xargs` 解决的正是这个问题——**把 stdin 的内容转成命令行参数**：

```zsh
echo "hello.txt" | xargs rm
```

`xargs` 拿到 stdin 里的 `hello.txt`，把它变成 `rm hello.txt` 然后执行。等价于手动敲 `rm hello.txt`。

---

### xargs 的经典场景

**删除所有 `.tmp` 文件：**

```zsh
find . -name "*.tmp" | xargs trash
```

`find` 找到所有 `.tmp` 文件，`xargs` 把它们拼成 `trash file1.tmp file2.tmp ...` 然后执行。

**批量重命名（配合 sed）：**

```zsh
ls *.txt | sed 's/\.txt$//' | xargs -I {} mv {}.txt {}.md
```

这里用到了 `-I {}`——`{}` 是占位符，`xargs` 会把每一行 stdin 代入 `{}` 的位置再执行命令。

例如 stdin 里有 `readme`，`xargs` 执行的命令就是 `mv readme.txt readme.md`。

---

### `xargs` 和空格的坑

`xargs` 默认用空格和换行来分割输入。如果文件名里带空格，会出问题。解决方法是用 `\0`（null 字符）做分隔符：

```zsh
find . -name "*.txt" -print0 | xargs -0 trash
```

`find` 的 `-print0` 用 `\0` 分隔，`xargs` 的 `-0` 用 `\0` 读。文件名里的空格就不会被误会了。

如果只是删文件，更省心的写法是用 `find -exec`，不需要 `xargs`：

```zsh
find . -name "*.tmp" -exec trash {} \;
```

---

## 把 stderr 也送进管道

管道默认只传递 stdout。如果想同时处理 stderr，用 `|&`（zsh 和 bash 都支持）：

```zsh
command 2>&1 | grep "error"   # 传统写法
command |& grep "error"       # zsh 简化写法
```

---

## 动手试试

```zsh
cd ~/cmd_practice

# 重定向
echo "line 1" > test.txt
echo "line 2" >> test.txt
cat test.txt

# stderr 重定向
ls nofile.txt 2> error.log
cat error.log

# 管道串联
cat /etc/passwd | grep "root" | cut -d: -f1

# xargs
echo "test.txt" | xargs cat

# 安全删除：用 find -exec 或 xargs + trash
touch temp1.tmp temp2.tmp temp3.tmp
find . -name "*.tmp" -print0 | xargs -0 trash

# 清理
trash error.log test.txt 2>/dev/null
```

---

## 小结

| 符号 | 作用 | 说明 |
|------|------|------|
| `>` | 覆盖写入文件 | stdout → 文件 |
| `>>` | 追加写入文件 | stdout → 文件末尾 |
| `2>` | 只重定向 stderr | 错误输出 → 文件 |
| `2>&1` | stderr 合并到 stdout | 一起输出 |
| `/dev/null` | 扔掉输出 | 黑洞 |
| `<` | 输入重定向 | 文件 → stdin |
| `<<` | Here Document | 内嵌多行文本 |
| `\|` | 管道 | stdout → 下一个命令的 stdin |
| `xargs` | stdin 转参数 | 解决管道只能传 stdin 的限制 |
| `\|&` | 管道（含 stderr） | zsh/bash 简写 |

记住一个核心规则：**管道传的是 stdout，`xargs` 把它转成参数**。搞明白这个区别，其他都是细节。

下一篇聊环境变量——`PATH` 到底是干什么的，为什么装了软件有时候命令行认不出来。

---

> [← 查看系列目录]({% link _pages/series-command-line.md %})
