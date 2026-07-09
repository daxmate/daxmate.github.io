---
layout: single
title:  "从零开始学命令行：字符编码——为什么中文会乱码"
date:   2026-07-09 13:00:00 +0800
categories:
  - Command Line
tags:
  - 命令行
  - 编码
  - UTF-8
  - locale
  - 乱码
---

打开一个文本文件，看到一堆 `å\x9B\xBE\xE7\x89\x87` 或者 `���`——乱码了。

原因很简单：**文件存的时候用一种编码，打开的时候用另一种编码**。编码不对，文字就显示不对。

---

## 什么是编码

电脑不认识文字，只认识数字。**编码就是文字和数字之间的映射表**。

比如 `A` 在 ASCII 编码里对应 `65`：
- 存：把 `A` 转成 `65`
- 读：把 `65` 转成 `A`

中文的编码比较复杂——汉字太多了，一个字节（0-255）根本不够用。于是有了各种方案：

| 编码 | 特点 | 常用场景 |
|------|------|----------|
| **ASCII** | 7 位，只用前 128 个字符 | 英文文本 |
| **GBK / GB2312** | 中文编码，每个汉字 2 字节 | 老旧系统、Windows 中文版 |
| **UTF-8** | 可变长度（1-4 字节），兼容 ASCII | **现代通用标准** |
| **UTF-16** | 大部分字符 2 字节 | Windows 内部、Java |
| **ISO-8859-1** | 单字节，支持西欧字符 | 老旧欧洲文本 |

---

## 为什么会有 UTF-8

以前各国各搞各的编码——日本用 Shift-JIS，简体中文用 GBK，繁体中文用 Big5。编码不一样、文件交换时就是灾难。

**UTF-8 把这个问题一次性解决了**。它能编码全世界的所有字符，而且兼容 ASCII——也就是说纯英文的 UTF-8 文件跟 ASCII 文件完全一样。

现在绝大多数系统和工具都默认用 UTF-8。

---

## 查看当前系统编码

```zsh
locale
```

```
LANG="zh_CN.UTF-8"
LC_ALL=
...
```

关键看 `LANG`——中间的 `UTF-8` 表示终端默认用 UTF-8。

查看文件的实际编码：

```zsh
file -I 文件名
```

```
文件名: text/plain; charset=utf-8
```

---

## 常见乱码场景

### 场景 1：文件是 GBK，终端是 UTF-8

```zsh
cat old_gbk_file.txt
```

显示一堆乱码。

处理方法：

```zsh
iconv -f GBK -t UTF-8 old_gbk_file.txt > new_utf8_file.txt
```

`iconv` 是编码转换工具，`-f` 是原编码，`-t` 是目标编码。

### 场景 2：文件是 UTF-8，cat 正常，less 乱码

```zsh
less 文件.txt
```

`less` 默认可能没设对编码。按 `:set encoding=utf-8`，或者在环境变量里设：

```zsh
export LESSCHARSET=utf-8
```

### 场景 3：文件名乱码

```zsh
ls
```

看到 `?????.txt` 或者 `\xE5\x9B\xBE.txt`。

用 `convmv` 转码文件名（先装 `brew install convmv`）：

```zsh
convmv -f GBK -t UTF-8 --notest *
```

---

## 文件头（BOM）

有些编辑器会在 UTF-8 文件开头加三个特殊字节（`EF BB BF`），叫 **BOM（Byte Order Mark）**。

这通常是无害的，但在某些情况下会引起问题——比如 Shell 脚本加了 BOM 后第一行的 `#!/bin/zsh` 就识别不了。

```zsh
# 检查有没有 BOM
hexdump -C script.sh | head -1

# 去掉 BOM
sed -i '1s/^\xEF\xBB\xBF//' script.sh
```

---

## 动手试试

```zsh
# 查看当前编码
locale | grep LANG

# 查看文件的编码信息
file -I ~/.zshrc

# 创建一个 GBK 文件，转成 UTF-8
echo "你好" > /tmp/test.txt
file -I /tmp/test.txt    # 通常是 utf-8

# 模拟编码问题
cat /tmp/test.txt | iconv -f UTF-8 -T GBK 2>/dev/null || echo "转错编码就会乱码"
```

---

## 小结

- **UTF-8** 是现在的通用标准，绝大多数情况用它就对了
- 看到乱码先用 `file -I` 查看文件实际编码
- `iconv -f 原编码 -t UTF-8` 把文件转成 UTF-8
- 自己新建的文件都存 UTF-8，能省掉 90% 的编码问题

---

> [← 查看系列目录]({% link _pages/series-command-line.md %})
