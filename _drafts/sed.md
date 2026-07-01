---
layout: single
title:  "从零开始学命令行：sed — 流编辑器入门"
date:   2026-07-07 09:00:00 +0800
categories:
  - Command Line
tags:
  - 命令行
  - sed
  - 文本处理
  - 替换
---

有些事用编辑器做太慢——比如把 50 个文件里的某个词全部改成另一个词。打开、查找替换、保存、关掉，重复 50 次？脑子没坏掉就已经算意志力很强了。

`sed` 就是干这个的。它的全称是 **Stream Editor**，流编辑器。它不像 VS Code 那样打开一个文件让你编辑，而是**对文本流进行处理**——一行一行地读、处理、输出。原始文件不会动（除非你特意让它写回去）。

`sed` 非常强大，能做的事情远不止查找替换。但这篇只讲日常最常用的场景，剩下的等真正遇到需求时自然会查。

---

## 准备工作

```zsh
mkdir -p ~/cmd_practice/sed-demo
cd ~/cmd_practice/sed-demo

cat > config.yaml << 'EOF'
server:
  host: localhost
  port: 8080
  debug: true

database:
  host: localhost
  port: 5432
  name: myapp_dev
EOF

cat > fruits.txt << 'EOF'
apple
banana
Apple
grape
pineapple
orange
EOF
```

---

## 替换：`s/旧/新/`

`sed` 最核心的命令是 `s`（substitute，替换）。格式：

```zsh
sed 's/旧内容/新内容/' 文件名
```

把 `localhost` 改成 `127.0.0.1`：

```zsh
sed 's/localhost/127.0.0.1/' config.yaml
```

```
server:
  host: 127.0.0.1
  port: 8080
  debug: true

database:
  host: localhost
  port: 5432
  name: myapp_dev
```

咦——只改了第一处 `localhost`，第二处没动。

**默认情况下，`sed` 的 `s` 命令只替换每行的第一个匹配**。

---

## 替换所有匹配：`g` 标志

```zsh
sed 's/localhost/127.0.0.1/g' config.yaml
```

```
server:
  host: 127.0.0.1
  port: 8080
  debug: true

database:
  host: 127.0.0.1
  port: 5432
  name: myapp_dev
```

加一个 `g`（global），这一行里所有匹配的地方都会被替换。

---

## 替换指定行

`sed` 可以只对特定行做替换。在最前面加上行号：

```zsh
sed '2s/localhost/127.0.0.1/' config.yaml
```

只替换第 2 行。

行范围：

```zsh
sed '2,4s/localhost/127.0.0.1/' config.yaml   # 第 2 到第 4 行
```

匹配模式的行：

```zsh
sed '/database/,/name:/s/localhost/remote-db/' config.yaml
```

从 "database" 那一行开始，到 "name:" 那一行结束，中间所有行做替换。

---

## 删除行：`d`

```zsh
sed '3d' config.yaml   # 删除第 3 行
sed '/debug/d' config.yaml   # 删除包含 debug 的行
```

---

## 直接修改文件：`-i`

前面的操作都只是输出到屏幕，原文件没变。如果要**原地修改文件**，用 `-i`：

```zsh
sed -i '' 's/localhost/127.0.0.1/g' config.yaml
```

**macOS 上 `-i` 后面必须带一个空字符串 `''`**，这是 macOS 版 `sed` 和 Linux 版 `sed` 最大的区别。Linux 上直接 `sed -i` 就行。

如果不带 `''`，macOS 会报错。带上 `''` 的意思是不创建备份。如果想备份，可以给一个后缀：

```zsh
sed -i '.bak' 's/localhost/127.0.0.1/g' config.yaml
```

这时候会生成一个 `config.yaml.bak` 备份文件。

两个平台的差别就这一个地方要记住。其余用法都是一样的。

---

## 一次做多个替换：`-e`

```zsh
sed -e 's/localhost/127.0.0.1/g' -e 's/debug: true/debug: false/' config.yaml
```

每个 `-e` 后面跟一个 `sed` 命令，按顺序执行。

---

## 打印特定行：`-n` + `p`

只看第 3 行：

```zsh
sed -n '3p' config.yaml
```

只看 2 到 4 行：

```zsh
sed -n '2,4p' config.yaml
```

`-n` 的意思是「默认不输出任何东西」，然后 `p` 显式指定哪些行要输出。删掉 `-n` 的话，`sed` 会先输出所有行，再额外输出匹配的行——结果匹配的行会重复出现，这通常不是想要的效果。

---

## 管道里的 sed

`sed` 最常用的场景是配合管道，一边流一边处理：

```zsh
cat config.yaml | sed 's/localhost/127.0.0.1/g' | grep -E "host|port"
```

```
  host: 127.0.0.1
  port: 8080
  host: 127.0.0.1
  port: 5432
```

替换 → 过滤，一条线串下来。

---

## 实战：批量修改文件

假设项目里所有 `.yaml` 文件都把 `localhost` 改成 `127.0.0.1`：

```zsh
find . -name "*.yaml" -exec sed -i '' 's/localhost/127.0.0.1/g' {} \;
```

`find` 找到所有 `.yaml` 文件，`sed -i` 原地替换。一条命令，全改了。

---

## 动手试试

```zsh
cd ~/cmd_practice/sed-demo

# 先复制一份，用备份模式试（安全）
cp config.yaml config.yaml.orig

# 替换（只输出，不改文件）
sed 's/localhost/127.0.0.1/' config.yaml

# 全局替换
sed 's/localhost/127.0.0.1/g' config.yaml

# 原地修改（带备份）
sed -i '.bak' 's/debug: true/debug: false/' config.yaml
cat config.yaml
cat config.yaml.bak

# 删除行
sed '/debug/d' config.yaml

# 打印指定行
sed -n '1,3p' config.yaml

# 管道配合
cat config.yaml | sed 's/127.0.0.1/localhost/g' | grep host
```

---

## 小结

| 操作 | 命令 | 说明 |
|------|------|------|
| 替换 | `s/旧/新/` | 每行第一个匹配 |
| 全局替换 | `s/旧/新/g` | 每行全部匹配 |
| 指定行 | `行号s/旧/新/` | 只对特定行替换 |
| 删除行 | `行号d` 或 `/模式/d` | 删除匹配的行 |
| 原地修改 | `-i ''` | macOS 必须带空字符串 |
| 备份修改 | `-i '.bak'` | 生成 .bak 备份 |
| 多个操作 | `-e` | 串联多个 sed 命令 |
| 打印行 | `-n '行号p'` | 只输出指定行 |

`sed` 的 `s/旧/新/g` 是刻在肌肉里的命令。改配置、改代码、批量处理文本——这一个命令能代替无数次手工编辑。

`sed` 的 `s/旧/新/g` 覆盖日常 90% 的需求。如果模式很复杂（涉及正则分组、前后断言），直接上 Python 写个脚本会更顺手。但对命令行日常操作来说，到这里就够了。

下一篇聊聊管道的深入用法——不只是 `cat | grep` 这么简单，`xargs` 能把管道输出变成命令参数。

---

> [← 查看系列目录]({% link _pages/series-command-line.md %})
