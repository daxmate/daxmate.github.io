---
layout: single
title:  "文件变了就自动跑命令——entr"
date:   2026-06-30 15:00:00 +0800
categories:
  - Tooling
tags:
  - CLI
  - entr
  - 效率工具
  - macOS
---

改完代码要手动跑测试，改了配置要重启服务，改了 CSS 要刷新浏览器——每次改一点东西就要多敲一条命令，很打断节奏。

**entr** 就是来解决这个问题的。它做的事一句话：**监视文件变化，变了就帮你跑命令。**

## 安装

```zsh
brew install entr
```

就一个二进制文件，没有依赖。

## 基本用法

把要监视的文件列表传给它，它会在文件变动时执行指定的命令：

```zsh
echo main.c | entr make
```

`main.c` 一保存，`entr` 自动跑 `make`。

监视多个文件：

```zsh
find . -name "*.py" | entr pytest
```

目录下任何一个 `.py` 文件变了，自动跑测试。

## 重启型任务：`-r`

很多场景下，文件变了不只是跑一次命令，而是需要**把旧进程杀掉重新启动**——比如改 `_config.yml` 要重启 Jekyll，改代码要重启 dev server。

这时候用 `-r`：

```zsh
echo _config.yml | entr -r bundle exec jekyll serve --livereload
```

保存 `_config.yml`，entr 杀掉旧的 `jekyll serve`，重新起一个新的。不用手动 Ctrl+C 再重敲命令了。

`-r` 的意思是：**先 SIGTERM 杀掉子进程，再重新执行命令。**

## 覆盖全项目：`-d`

监视目录下的所有文件，包括新增的：

```zsh
find src | entr -d make
```

`-d` 让 entr 也监视目录里新增的文件。不加 `-d`，新建的文件不会被追踪。

## 实际场景

### 改博客主题，自动重启

Jekyll 的 `--livereload` 能自动刷新文章和 CSS，但 `_config.yml` 改了必须重启整个进程。配合 entr：

```zsh
ls _config.yml | entr -r bundle exec jekyll serve --livereload
```

改皮肤、改配置，保存即生效。

### 写代码，改完自动跑测试

```zsh
find . -name "*.py" -not -path "./venv/*" | entr python -m pytest -q
```

排除虚拟环境目录，专注项目代码。

### 改 LaTeX，自动编译

```zsh
echo thesis.tex | entr latexmk -pdf thesis.tex
```

每次保存，自动编译 PDF。

### 编译 C 项目

```zsh
find . -name "*.c" -o -name "*.h" | entr make
```

## 和同类工具的对比

| 工具 | 特点 |
|------|------|
| `fswatch` | macOS 原生支持好，但需要写 shell 逻辑来重启进程 |
| `nodemon` | Node.js 生态，默认只适合 Node 项目 |
| `watchexec` | Rust 写的，功能类似 entr |
| `entr` | 最轻量（几十 KB），Unix 哲学——只做一件事 |

entr 的优势是极简。没有配置文件，没有复杂选项，从 stdin 读文件列表，用起来就是管道那种感觉。

## 几个注意点

1. **不是递归的**——`find` 出来的文件列表交给 entr，entr 自己不扫描目录
2. **stdin 是唯一输入源**——没有配置文件，靠管道和 find 配合
3. **`-r` 杀进程是 SIGTERM**——优雅退出，不是 force kill

## 小结

```
文件列表 → entr → 命令
```

就这个模式。需要自己手动重复跑的命令，交给 entr 就行。

`entr` 和 `fswatch` 经常被放在一起比较。下次可以聊聊怎么用 `fswatch` 做 entr 做不了的事。

---

*写完这篇才发现，这篇本身也算「输出倒逼输入」——为了写清楚 entr，把 man 手册又翻了一遍。*
