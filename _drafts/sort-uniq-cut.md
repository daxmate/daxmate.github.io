---
layout: single
title:  "从零开始学命令行：sort、uniq、cut — 文本处理三件套"
date:   2026-07-01 14:00:00 +0800
categories:
  - Command Line
tags:
  - 命令行
  - sort
  - uniq
  - cut
  - 文本处理
---

`grep` 帮你找到想要的内容。找到之后，通常还需要进一步处理——排序、去重、提取特定的列。这时候 `sort`、`uniq`、`cut` 就该出场了。

这三个命令都很轻量，不占脑容量，但日常处理数据的频率远超你想象。

---

## 准备工作

```zsh
mkdir -p ~/cmd_practice/sort-demo
cd ~/cmd_practice/sort-demo

cat > scores.txt << 'EOF'
张三 数学 92
李四 数学 85
王五 数学 76
张三 英语 88
李四 英语 91
王五 英语 79
张三 语文 95
李四 语文 82
王五 语文 88
EOF

cat > visits.txt << 'EOF'
/home
/about
/home
/contact
/home
/about
/products
/home
EOF
```

---

## `sort`：排序

万事排好序，看着就舒服。

### 基本排序

```zsh
sort visits.txt
```

```
/about
/about
/contact
/home
/home
/home
/home
/products
```

默认按字母顺序排。注意它是按**整行**排序的，不是按某个字段。

### 数字排序：`-n`

默认排序是字典序（alphabetical），"10" 会排在 "2" 前面（因为 "1" < "2"）。如果内容真的是数字，用 `-n` 按数值排：

```zsh
echo -e "2\n10\n1\n20" | sort
```

```
1
10
2
20
```

不对，因为字典序。

```zsh
echo -e "2\n10\n1\n20" | sort -n
```

```
1
2
10
20
```

对了。

### 倒序：`-r`

```zsh
sort -r visits.txt
```

```
/products
/home
/home
/home
/home
/contact
/about
/about
```

### 按某一列排序：`-k`

`scores.txt` 有三列：名字、科目、分数。按分数从高到低排：

```zsh
sort -k3 -nr scores.txt
```

```
张三 语文 95
张三 数学 92
李四 英语 91
张三 英语 88
王五 语文 88
李四 数学 85
李四 语文 82
王五 英语 79
王五 数学 76
```

`-k3` 是按第三列排序，`-n` 是数字排序，`-r` 是倒序（从高到低）。

### 去重排序：`-u`

```zsh
sort -u visits.txt
```

```
/about
/contact
/home
/products
```

`-u` 在排序的同时去重。

---

## `uniq`：去重

`uniq` 本身只删除**连续**重复的行。这是很多人踩过的坑——以为 `uniq` 能直接去重，但其实它只删相邻的。

```zsh
cat visits.txt | uniq   # 效果很有限
```

```
/home
/about
/home
/contact
/home
/about
/products
/home
```

所以 `uniq` 的标配是**先 `sort` 再 `uniq`**：

```zsh
sort visits.txt | uniq
```

```
/about
/contact
/home
/products
```

去重成功。

### 统计重复次数：`-c`

```zsh
sort visits.txt | uniq -c
```

```
   2 /about
   1 /contact
   4 /home
   1 /products
```

`/home` 出现了 4 次，比别的都多。这个组合在做数据统计的时候天天用。

### 只显示重复的：`-d`

```zsh
sort visits.txt | uniq -d
```

```
/about
/home
```

只显示出现了一次以上的。

### 只显示不重复的：`-u`

```zsh
sort visits.txt | uniq -u
```

```
/contact
/products
```

只出现了一次的。

---

## `cut`：切出指定列

数据是按固定分隔符排好的（逗号、空格、制表符），很多时候只需要其中的某一列。`cut` 就是干这个的。

### 按字符位置切：`-c`

```zsh
echo "abcdefg" | cut -c1-3
```

```
abc
```

切出第 1 到第 3 个字符。

### 按分隔符切：`-d` 和 `-f`

`scores.txt` 里每行是三列，空格分隔。只取科目和分数：

```zsh
cut -d' ' -f2,3 scores.txt
```

```
数学 92
数学 85
数学 76
英语 88
英语 91
英语 79
语文 95
语文 82
语文 88
```

`-d' '` 指定以空格分隔，`-f2,3` 取第 2 和第 3 列。

只取名字：

```zsh
cut -d' ' -f1 scores.txt | sort | uniq
```

```
张三
李四
王五
```

从成绩表里提取出所有不重复的学生名字。这就是串联的威力。

### 处理 CSV 文件

CSV 用逗号分隔：

```zsh
echo "name,age,city" > people.csv
echo "张三,25,北京" >> people.csv
echo "李四,30,上海" >> people.csv
echo "王五,28,广州" >> people.csv

cut -d',' -f1,3 people.csv
```

```
name,city
张三,北京
李四,上海
王五,广州
```

---

## 串联使用：三个一起上

假设你要从一个访问日志里找出访问量最高的 3 个页面：

```zsh
sort visits.txt | uniq -c | sort -rn | head -n 3
```

```
   4 /home
   2 /about
   1 /products
```

拆开看每一步在干嘛：

1. `sort visits.txt` — 排序（`uniq` 需要排好序的数据）
2. `uniq -c` — 统计每行出现次数
3. `sort -rn` — 按统计数字倒序排列
4. `head -n 3` — 取前三名

四个命令串成一条线，一步到位。这就是命令行的优美之处——每个命令只做一件事，但把它们串起来能做很复杂的工作。

---

## 动手试试

```zsh
cd ~/cmd_practice/sort-demo

# sort
sort scores.txt
sort -k3 -nr scores.txt

# uniq（先 sort）
sort visits.txt | uniq
sort visits.txt | uniq -c
sort visits.txt | uniq -d

# cut
cut -d' ' -f1 scores.txt
cut -d' ' -f1 scores.txt | sort | uniq

# 串联
sort visits.txt | uniq -c | sort -rn | head -n 3
```

---

## 小结

| 命令 | 核心选项 | 常用场景 |
|------|----------|----------|
| `sort` | `-n` 数字排序，`-r` 倒序，`-k` 指定列，`-u` 去重排序 | 日志排序、数据整理 |
| `uniq` | `-c` 统计次数，`-d` 重复项，`-u` 唯一项 | 去重和统计，先 `sort` 再用 |
| `cut` | `-d` 分隔符，`-f` 选列，`-c` 选字符 | 提取 CSV 列、切字段 |

记住两个常用组合：
- `sort | uniq -c | sort -rn` — 排序 + 统计 + 倒序（频率统计标配）
- `cut -d'分隔符' -f列号` — 提取指定列

下一篇聊 `sed`——一个能批量替换文本的流编辑器。改配置文件、批量重命名内容，用它能省下大量手工编辑的时间。

---

> [← 查看系列目录]({% link _pages/series-command-line.md %})
