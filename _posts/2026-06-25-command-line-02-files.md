---
title: "命令行系列 #2：文件操作——别再一个个点了"
date: 2025-01-02
categories:
  - Blog
tags:
  - 命令行
  - bash
---

> 上一篇我们说了为什么命令行值得学。这篇来点实在的——文件操作。这些是你每天都会用到的命令，学会了效率至少翻倍。

---

## ls —— 你的文件清单

`ls` 是 list 的缩写，就是「列出文件」。

```bash
ls                    # 简单列出
ls -l                 # 详细信息（权限、大小、日期）
ls -a                 # 包括隐藏文件（以.开头的）
ls -la                # 组合使用，最常见
```

试一下 `ls -la`，你会看到类似这样的输出：

```
-rw-r--r--   1 dax  staff   1024 Jun 25 10:00 .bashrc
drwxr-xr-x   3 dax  staff     96 Jun 25 09:00 Documents
```

第一列的 `d` 表示文件夹（directory），`-` 表示文件。后面 `rwx` 是权限，后面再讲，现在先不用管。

---

## mv —— 移动和重命名

`mv` 可以干两件事：

**移动文件：**
```bash
mv myfile.txt Documents/      # 把文件移到 Documents 文件夹
```

**重命名：**
```bash
mv oldname.txt newname.txt    # 改名
```

**批量移动：**
```bash
mv file1.txt file2.txt file3.txt Documents/    # 全部移到 Documents
# 注意：最后一个参数是目标文件夹，前面的全都是要移动的文件
```

### ⚠️ 防翻车技巧

`mv` 默认会直接覆盖同名文件，没有任何提示。一个不小心就把重要文件覆盖了。

**解决办法：养成好习惯，加 `-i` 参数。**

```bash
mv -i important.txt Documents/   # 如果目标已有同名文件，会先问你是否要覆盖
```

你也可以设一个别名（alias），让 `mv` 默认就带 `-i`：

```bash
alias mv='mv -i'
```

把上面这行加到 `~/.bashrc` 或 `~/.zshrc` 里，以后就不用每次都手动加 `-i` 了。

---

## cp —— 复制文件

```bash
cp oldfile newfile              # 复制并改名
cp file.txt Documents/          # 复制到文件夹
cp -R folder/ backup/           # 复制整个文件夹（-R 是递归的意思）
cp -v file.txt destination/     # -v 显示复制过程，方便看有没有出错
```

`cp` 默认不能复制文件夹，必须加 `-R`（recursive）。这是很多人第一次用 `cp` 时被坑的地方。

---

## rm —— 删除（小心！）

```bash
rm file.txt                     # 删除文件
rm -r folder/                   # 删除文件夹
rm -rf folder/                  # 强制删除，不提示
```

**`rm -rf` 是最危险的一个命令。** `r` 是递归删除文件夹，`f` 是 force——它会直接把东西删掉，不进回收站，找不回来。

> 我曾见过有人跑 `rm -rf /` 把自己整个系统删了的（虽然在现代系统上这不容易发生了，但后果一样严重）。

**安全习惯：**

1. 永远在 `rm` 之前先 `ls` 确认你要删什么
2. 写 `rm -rf` 之前，深呼吸，再检查一遍路径

还有一个好办法：用 `trash` 代替 `rm`（macOS 上 `brew install trash`），它会移到废纸篓而不是直接删除，可以反悔。

---

## find —— 找文件

文件一多就找不到在哪里了？`find` 是你的搜索利器。

```bash
find . -name "*.md"             # 在当前目录下找所有 .md 文件
find / -name "config.yml"       # 在整个系统里找
find . -type d -name "draft"    # 找名为 draft 的文件夹
find . -mtime -7                # 找最近 7 天修改过的文件
```

`find` 的强大之处是可以结合其他命令用。比如找到所有 `.log` 文件并删除：

```bash
find . -name "*.log" -exec rm {} \;
```

这里的 `{}` 是占位符，代表每一个找到的文件。`\;` 表示命令结束。

不过刚开始不用急着用 `-exec`，先用 `find` 找到文件，确认没错，再手动处理就行。

---

## 实战场景

### 场景一：整理下载文件夹

你的下载文件夹乱成一锅粥？一行命令就能分类整理：

```bash
cd ~/Downloads
mkdir -p Images Documents Archives
mv *.jpg *.png *.gif Images/ 2>/dev/null
mv *.pdf *.docx *.txt Documents/ 2>/dev/null
mv *.zip *.tar.gz Archives/ 2>/dev/null
```

（`2>/dev/null` 的意思是「错误信息不要显示出来」，因为如果某种文件不存在，`mv` 会报错，而我们不想看到那些错误。）

### 场景二：批量改名

把一堆 `report-2024-*.pdf` 改成 `report-2025-*.pdf`：

```bash
for file in report-2024-*.pdf; do
    mv "$file" "${file/2024/2025}"
done
```

看不懂没关系。这是循环+变量替换。你先知道有这种操作就行，后面会详细讲。

---

## 小结

这一篇你学会了五个核心命令：

| 命令 | 作用 | 最常用搭配 |
|------|------|-----------|
| `ls` | 列出文件 | `ls -la` |
| `mv` | 移动/改名 | `mv -i` |
| `cp` | 复制 | `cp -R`（复制文件夹） |
| `rm` | 删除 | `rm -rf`（小心！） |
| `find` | 搜索文件 | `find . -name` |

**最重要的不是记住全部参数，而是知道「有这个东西」，用的时候能想起来去查。** 没有人能把所有参数背下来，`man` 命令（或者 Google）就是你最好的老师。

---

*下一篇：文本处理——grep、sed、awk，让数据乖乖听话*
