---
layout: single
title:  "entr 和 fswatch：两个文件监控工具，两种思路"
date:   2026-07-11 11:07:00 +0800
categories:
  - Tooling
tags:
  - CLI
  - entr
  - fswatch
  - 效率工具
  - macOS
---

写代码时改完一个文件，手动切回终端跑测试；写文档时保存，切回去看编译结果；改个配置，手动重启服务。

这些小动作频率不高，但每次打断思路。于是有了文件监控工具——文件变了，自动跑活。

市场上的文件监控工具有不少：Python 的 watchdog、Node.js 的 nodemon、Rust 写的 watchexec。但今天要聊的两款比较特别，**一个极简到不到 40KB，一个以数据流的方式工作。** 它们不是竞品，是两种不同的设计哲学。

---

## entr — 我把全套帮你做了

### 安装

```zsh
brew install entr
```

就一个二进制文件，不到 40KB。

### 基本模式

```zsh
echo main.c | entr make
```

核心工作流：**stdin 传文件列表 → entr 监视 → 变了就跑命令。**

把 `echo main.c` 换成文件列表就行：

```zsh
find . -name "*.py" | entr pytest
```

所有 `.py` 文件一有变化，自动跑测试。

entr 的设计思路很清晰——它只关心三件事：**看哪些文件、跑什么命令、跑一次还是重启**。

### 常用选项

**`-r`** — 重启模式。进程首次执行后持续监听，文件变了就 kill 旧进程、起新进程。适用于 dev server、Jekyll 这类长期服务：

```zsh
echo _config.yml | entr -r bundle exec jekyll serve --livereload
```

保存 `_config.yml`，entr 杀掉旧的 Jekyll 进程，开新的。不用手动 Ctrl+C 再重敲命令了。

**`-d`** — 也监控新增文件。默认 entr 只监视初始传入的文件列表，不留意后来新创建的文件。加了 `-d`，目录结构变了它会退出（因为监听到了目录变动），`find` 可以重新跑一遍列出新文件，配合 while 循环就能持续追踪：

```zsh
while sleep 1; do find src | entr -d make; done
```

**`-c`** — 每次跑命令前清屏。跑测试时特别实用，一眼就看最新结果：

```zsh
find . -name "*.py" | entr -c pytest
```

**`-p`** — 等第一次变化再执行。默认 entr 启动后先跑一次命令，不想这样可以用 `-p` 推迟。

### 更多用法

编译 LaTeX：

```zsh
echo thesis.tex | entr latexmk -pdf
```

监视 git 跟踪的文件：

```zsh
git ls-files '*.py' | entr pytest
```

用 `/_` 占位符引用被监控文件（仅限单个文件的情况）：

```zsh
echo schema.sql | entr psql -f /_
```

更多时候，entr 是跟 `find` 或 `rg` 搭配的：

```zsh
rg -l --type py | entr -c python -m pytest
```

### entr 的特点

- **极轻**——~38KB 二进制，零依赖
- **stdin 是唯一的输入源**——没有配置文件，不扫描目录
- **只管"监视→执行"这一个闭环**——不输出事件细节，不提供回调

---

## fswatch — 我给你原始事件流

### 安装

```zsh
brew install fswatch
```

### 基本模式

entr 不需要指定要监视哪个目录——文件列表全从 stdin 来。fswatch 相反，它以**目录或文件**为参数：

```zsh
fswatch .
```

一启动就会疯狂输出——把当前目录下每个被修改的文件路径打出来。这其实是 fswatch 的模式：它不帮你做"跑什么命令"，它只告诉你"什么东西发生了什么事"，你要自己处理。

### 为什么说它像传感器

fswatch 默认输出一行一个路径，可以 pipe 到 `while read`：

```zsh
fswatch . | while read f; do echo "Changed: $f"; done
```

但只拿个路径太浪费了。用 `-x` 可以拿到事件类型和时间戳：

```zsh
fswatch -x .
```

输出会变成类似这样（根据不同后端和上下文，标志位有差异）：

```
/path/to/file.go  IsFile Updated AttributeModified
```

至少能看出是文件还是目录、是更新还是新增。这才是传感器的样子——**把原始数据丢给你，你自己决定怎么用**。

### 常用选项

**`-r`** — 递归子目录（默认 false，必须显式加）

**`-l`** — 延迟（拉长事件合并间隔，避免一次保存触发太多事件）：

```zsh
fswatch -r -l 2 .
```

每 2 秒合并一次事件，默认是 1 秒。

**`-x`** — 输出事件标志（更新、创建、删除、属性变更等）

**`-t`** — 输出时间戳

**`-1`** — 收到一次事件后退出。适合一次性任务。

**`--exclude=REGEX`** / **`--include=REGEX`** — 正则过滤路径

**`-m fsevents_monitor`** — 指定监控后端。macOS 默认为 FSEvents（内核级，高效），Linux 会用 inotify，还可以选 kqueue 或 poll 做备用。

**`--event=TYPE`** — 只监听特定事件类型。

### 实际怎么用

因为 fswatch 不帮我们跑命令，要自己拼 shell。最简单的模式：

```zsh
fswatch -o . | xargs -n1 make
```

这里 `-o` 不是输出文件路径，而是把每批事件计数为一次输出（避免一次保存触发多次构建）。`xargs -n1` 保证每次变化都跑一次。

带延迟合并的版本：

```zsh
fswatch -l 2 -o src/ | xargs -n1 make
```

延迟 2 秒，适合在编辑器中连续写代码时避免频繁触发。

重启服务稍微麻烦点，要自己写循环：

```zsh
fswatch -o . | while read n; do
  kill -TERM $(cat server.pid) 2>/dev/null
  ./start-server.sh &
  echo $! > server.pid
done
```

比 entr 麻烦——entr 一个 `-r` 搞定的事，fswatch 要自己处理进程生命周期。但好处是事件类型、路径、时间戳都拿得到，想怎么处理都行。

---

## 放在一起看

| | entr | fswatch |
|---|---|---|
| 设计 | 给个命令，自动跑 | 输出事件流，你自己处理 |
| 输入 | stdin 文件列表 | 目录/文件路径 |
| 输出 | 无（直接执行命令） | 事件文本流 |
| 递归 | 靠 `find`/`rg` 配合 | 自带 `-r` |
| 重启进程 | 内置 `-r` | 自己写循环 |
| 事件过滤 | 无（全量触发） | `--exclude`、`--event`、`-x` |
| 跨平台 | BSD/macOS kqueue + Linux inotify | FSEvents/kqueue/inotify/poll |
| 体积 | ~38KB | ~106KB |
| 学习曲线 | 平 | 稍陡（但更灵活） |

表格只能反映功能差异，真正的区别在**设计思路**。

entr 像是工厂里的一台自动化机器——原料进去，成品出来，内部是个黑盒。不需要关心它怎么检测文件变化，它已经做好了"变化→执行"的全流程。

fswatch 像是一个数据采集仪——它不停地读数，吐给你原始数据。拿这些数干什么都行：算平均值、画图、报警。它只保证数据的准确性，不猜测你的用途。

这两种思路没有高下之分，看你需要什么：

- 日常工作流中，80% 的场景是**"文件变了就跑个命令(或重启个服务器)"**——entr 最合适，一行搞定
- 剩下的 20%——需要按文件类型区别处理、需要记录日志、需要在特定条件下才触发——fswatch 更有发挥空间

---

## 怎么选

简单回答：

**不想多操心 → entr。** 一行命令，开箱即用。跑测试、编译、重启 dev server，日常开发最常用的模式都覆盖了。

**需要精细控制 → fswatch。** 要过滤事件类型、要处理不同目录的不同反应、要记录日志——fswatch 提供事件流，逻辑自己写。

当然也可以两个都用。我日常是这样的：

- entr 跑测试和编译（一天用几十次）
- fswatch 用在自定义脚本里——比如只修改 `config/` 下的 YAML 文件时才触发重启，用 `--exclude` 和 `--event` 过滤好事件，再塞给重启逻辑

不过别过度设计了。几年前我用 fswatch 搭过一个"自动触发"系统，后来发现其实一行 entr 就能搞定。**能从简单的开始，就不要从复杂的开始**——也算 CLI 工作的一个通用经验。

---

*两个工具，两种思路。一个替你做了全套决策，一个给你全部数据让你自己做决策。没有谁替代谁，看你处在什么场景。*
