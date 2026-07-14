---
layout: single
title:  "LaTeX 编译就用 latexmk——一个智能构建工具"
date:   2026-07-14 18:45:00 +0800
categories:
  - Tools
tags:
  - LaTeX
  - 工具
---

> **工具笔记** 这个系列偏技术笔记类，记录一些工具使用的深入话题。不追求教程式的完整，重点是把理解写透。

## latexmk 是什么

写 LaTeX 的人多少都有过这种经历：编译一遍，交叉引用还是问号；再编译一遍，参考文献出来了但目录页码不对；再编一遍……终于对了。下次改了一行字，又得手动重复这套流程。

`latexmk` 就是来解决这个问题的。它不是编译器（不是 `pdflatex`、`xelatex` 那种角色），而是一个 **自动化构建工具**。它帮你决定：需要编几次、按什么顺序编、要不要先跑 `bibtex`。

名字里的 "mk" 跟 `make` 同源——它做的就是 LaTeX 世界的 make。

## 落伍的编译方式

现在用 LaTeX 的人很多没经历过传统编译流程。但在 `latexmk` 普及之前，LaTeX 的编译其实是一片混乱。

这一切要从 Knuth 设计 TeX 的时代说起。

### 从 .tex 到 .dvi

TeX 最初的设计输出格式是 DVI（DeVice Independent），一种与设备无关的二进制格式。我们把 `.tex` 文件丢给 `tex` 命令，得到一个 `.dvi` 文件。DVI 不是给人直接看的——它描述了页面上的字符和位置，需要在显示器或打印机上通过驱动程序解析才能呈现。

用命令来说就是：

```shell
tex main.tex    # → main.dvi
```

### 从 DVI 到 PS 再到 PDF

后来 PostScript 成为打印行业的标准，大家开始把 DVI 转成 PS（PostScript）：

```shell
dvips main.dvi -o main.ps    # → main.ps
```

再后来 PDF 成了文档交换的事实标准，于是又多了一步：

```shell
ps2pdf main.ps                # → main.pdf
```

两条管道拼起来就是完整的流程：

```shell
tex main.tex && dvips main.dvi -o main.ps && ps2pdf main.ps
```

这就是 `$pdf_mode = 2` 那条路径的由来——一条如今看来相当绕路的编译管道。

### 直接从 DVI 到 PDF

有人觉得从 DVI 到 PS 再到 PDF 太多弯路了，于是出现了 `dvipdf`（或者更完善的 `dvipdfmx`），直接从 DVI 转 PDF：

```shell
tex main.tex            # → main.dvi
dvipdfmx main.dvi       # → main.pdf
```

这就是 `$pdf_mode = 3`。比 PS 中转少了一步，但还是依赖 DVI 这个中间产物。

![传统编译路径 vs 现代直接编译](/assets/images/latexmk/01-comparison-compilation-pipeline.svg)

### 中间文件的泛滥

不管走哪条路径，每次编译都会产生一大批中间文件：

- `.aux`——交叉引用信息
- `.log`——编译日志
- `.toc`——目录
- `.lof` / `.lot`——图表目录
- `.bbl` / `.blg`——参考文献相关
- `.ind` / `.ilg`——索引相关
- `.out`——超链接
- `.synctex.gz`——正向反向搜索数据

这些文件是编译过程中的副产品，用来记录上一步的信息、供下一步读取。传统上它们跟 `.tex` 源文件混在一起，把项目目录搞得乌烟瘴气。

### 这一套的问题

传统编译方式的痛点很明显：

**管道的脆弱性。** 每一步的输出是下一步的输入，任何一个环节报错，后面的步骤都不会执行。编到一半发现 BibTeX 报错，修正后要重新从头开始跑整条链。

**固定流程的盲目。** 很多人写脚本固定跑 `latex → bibtex → latex → latex` 四遍。但有时候只需要两遍就够，有时候需要六遍才稳定——固定的次数要么浪费要么不够。

`latexmk` 改变了这一切——不再需要手动管这些。

## 和手动编译的区别

手动编译通常是这样的：

```shell
xelatex main.tex
bibtex main
xelatex main.tex
xelatex main.tex
```

这个顺序不是固定的——有时候只需要编两次，有时候要多编一次，取决于文档里用了多少引用、索引、术语表。

`latexmk` 只需要：

```shell
latexmk -pdf main.tex
```

然后它自己判断接下来要做什么。

## 怎么判断的

每次编译完成后，`latexmk` 会读取生成的 `.log` 文件，在里面找一些特定的提示字符串：

- `Rerun to get cross-references`——标签引用变了，重跑
- `Rerun to get citations correct`——文献引用变了，重跑
- `Label(s) may have changed. Rerun`——标签可能变了
- `Package biblatex Warning: Please rerun LaTeX`——biblatex 要求的

如果日志里出现了这些，它就自动再跑一次。如果提示缺引用信息，它会中间插一次 `biber` 或 `bibtex`，跑完再回来继续编。

一直重复到日志里没有 "Rerun" 提示了，才算完。

跟写死 4 遍的脚本比，它的好处是：一篇纯文本文章只编 1 次就停了，加了引用才多编几次——不浪费时间，也不漏步骤。

考虑到有些极端情况会形成循环依赖（比如 `cleveref` 对标签前后引用），`latexmk` 设了一个编译次数上限（默认 5~10 次），超过就停。

![latexmk 智能编译决策流程](/assets/images/latexmk/02-flowchart-latexmk-loop.svg)

## 持续预览模式

加上 `-pvc` 参数可以进入持续预览模式：

```shell
latexmk -pvc -pdf main.tex
```

之后每次保存源文件，`latexmk` 会自动重新编译。但关键在于：它不是固定跑 N 遍，而是根据改动内容智能判断。只改了正文文字（没增减引用或标签），就只编 1 次；新增了 `\label` 或 `\cite`，就多编几次。

## `.latexmkrc` 配置

`latexmk` 的行为通过一个配置文件控制，放在三个位置（优先级从低到高）：

1. 系统级——TeX 发行版提供的默认配置
2. 用户级——`~/.latexmkrc`，放个人的通用偏好
3. 项目级——项目根目录下的 `./latexmkrc`，放项目特定设置

### 选择编译引擎

通过 `$pdf_mode` 变量：

| 模式 | 含义 |
|------|------|
| 0 | 只生成 DVI，不生成 PDF |
| 1 | `pdflatex`——兼容性最广，但 Unicode 和特殊字体受限 |
| 2 | 先 LaTeX 生成 PS，再用 `ps2pdf` 转 PDF |
| 3 | 先 LaTeX 生成 DVI，再用 `dvipdf` 转 PDF |
| 4 | `lualatex`——全面支持 Unicode 和 OpenType 字体 |
| 5 | `xelatex`——同样支持 Unicode 和系统字体，中文模板常用 |

模式 2 和 3 是两条传统路径，现在很少作为首选。**模式 2** 适用于文档里有 `pstricks` 画的图（需要 PostScript 处理），**模式 3** 适用于用了 `eepic` 或 `tpic` 这类依赖 DVI 的宏包。日常使用推荐 1、4、5。

![.latexmkrc 配置概览](/assets/images/latexmk/03-infographic-config-overview.svg)

### 一个完整的配置示例

```perl
# 1. 选择编译引擎（这里选 XeLaTeX）
$pdf_mode = 5;

# 2. 为 XeLaTeX 添加常用参数
$xelatex = 'xelatex -interaction=nonstopmode -file-line-error -synctex=1 %O %S';

# 3. （可选）设置输出目录
$out_dir = 'build';
$aux_dir = 'build';

# 4. （可选）设置 PDF 预览器（macOS 用 Skim）
$pdf_previewer = 'open -a Skim';

# 5. （可选）清理规则
@generated_exts = (@generated_exts, 'synctex.gz');
```

几个常用参数的含义：
- `-synctex=1`——PDF 和源文件之间来回跳转，编辑必备
- `-interaction=nonstopmode`——遇到错误不中断，尽量继续
- `-file-line-error`——错误信息显示为"文件名:行号:内容"
- `--shell-escape`——允许调用外部程序（`minted` 代码高亮、`gnuplot` 绘图等）

### 注意

`-shell-escape` 是有安全风险的——它允许 LaTeX 执行系统命令。只在确实需要调外部程序时才开启，不要当默认参数一直挂着。

## 唯一的盲区

`latexmk` 判断的依据是 `.log` 里的标准 "Rerun" 提示字符串。如果某个宏包作者没有按规范输出这些提示，`latexmk` 就可能感知不到需要重跑。这时可以用 `-g` 参数强制它忽略已有的日志记录，从头完整跑一遍：

```shell
latexmk -g -pdf main.tex
```

配套的还有 `-C`，用来清理所有生成的辅助文件（`.aux`、`.log`、`.out` 等）。

## 小结

一句话：`latexmk -pdf main.tex` 就够了，剩下的交给工具。
