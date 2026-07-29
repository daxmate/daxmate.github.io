---
title: "用 Ghostscript 压缩 PDF 文件"
---

> **工具笔记** 不是教程，是日常使用中积累下来的一些技巧和备忘。

## 场景

收到一个 PDF，几十 MB 甚至上百 MB，里面其实没什么复杂内容，就是扫描件或者带了几张高清图。发邮件太大，传文件太慢——需要压缩。

## Ghostscript 是什么

Ghostscript 是一个 PostScript 和 PDF 的解释器。它最出名的用途是作为打印系统的后端，但它也自带了一个 `pdfwrite` 输出设备——可以理解为"把 PDF 重新写一遍"，重写的过程中可以做压缩、降采样、字体子集化等优化。

macOS 上通过 Homebrew 安装：

```shell
brew install ghostscript
```

## 一句话命令

```shell
gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.4 \
   -dPDFSETTINGS=/ebook \
   -dNOPAUSE -dQUIET -dBATCH \
   -sOutputFile=compressed.pdf input.pdf
```

分解一下每个参数：

| 参数 | 作用 |
|------|------|
| `-sDEVICE=pdfwrite` | 输出设备设为"重新生成 PDF" |
| `-dCompatibilityLevel=1.4` | PDF 版本兼容性（1.4 够用，支持压缩） |
| `-dPDFSETTINGS=/ebook` | **核心参数**——压缩质量预设（见下文） |
| `-dNOPAUSE` | 每页处理完不暂停 |
| `-dBATCH` | 处理完所有文件后自动退出 |
| `-dQUIET` | 减少输出信息 |
| `-sOutputFile=compressed.pdf` | 输出文件名 |

## 四个质量预设

`-dPDFSETTINGS` 是最关键的参数，控制压缩到什么程度：

| 预设 | 质量 | 用途 |
|------|------|------|
| `/screen` | 最低（72 dpi） | 屏幕预览用，体积最小 |
| `/ebook` | 中等（150 dpi） | 日常推荐，平衡体积和质量 |
| `/printer` | 较高（300 dpi） | 打印用，质量不错 |
| `/prepress` | 最高（300 dpi） | 印前出版，保留最多细节 |

### 实际效果参考

同一份 26 MB 的扫描 PDF：

- `/screen` → 约 2 MB（文字还能看，插图糊了）
- `/ebook` → 约 5 MB（最佳平衡）
- `/printer` → 约 12 MB（几乎看不出区别）
- `/prepress` → 约 18 MB（基本是原样）

日常发文件用 `/ebook` 就够了。

## 更多控制

如果预设不够用，还可以手动调：

```shell
gs -sDEVICE=pdfwrite -dCompatibilityLevel=1.4 \
   -dPDFSETTINGS=/ebook \
   -dColorImageResolution=120 \
   -dGrayImageResolution=120 \
   -dMonoImageResolution=300 \
   -dDownsampleColorImages=true \
   -dDownsampleGrayImages=true \
   -dEmbedAllFonts=true \
   -dSubsetFonts=true \
   -dNOPAUSE -dQUIET -dBATCH \
   -sOutputFile=compressed.pdf input.pdf
```

常用微调选项：

| 选项 | 意义 |
|------|------|
| `-dColorImageResolution=120` | 彩色图降采样到 120 dpi |
| `-dGrayImageResolution=120` | 灰度图降采样到 120 dpi |
| `-dMonoImageResolution=300` | 黑白图（扫描文字）保留 300 dpi |
| `-dDownsampleColorImages=true` | 启用彩色图降采样 |
| `-dEmbedAllFonts=true` | 嵌入所有字体 |
| `-dSubsetFonts=true` | 只嵌入用到的字符，减小体积 |

## 批量处理

压缩目录下所有 PDF：

```shell
mkdir -p compressed
for f in *.pdf; do
  gs -sDEVICE=pdfwrite -dPDFSETTINGS=/ebook \
     -dNOPAUSE -dQUIET -dBATCH \
     -sOutputFile="compressed/${f%.pdf}-compressed.pdf" "$f"
done
```

## 对比压缩率

压缩完看一眼效果：

```shell
# 压缩前
ls -lh original.pdf
# 压缩后
ls -lh compressed.pdf
# PDF 信息
pdfinfo compressed.pdf | grep -E "Pages|Page size"
```

## 注意

- Ghostscript 压缩是重新编码 PDF，**不是无损的**。图片会被降采样、重新压缩，质量会有损失。关键文档先保留原件。
- 对文本型 PDF（纯文字的论文、报告）效果最好，压缩率很高。
- 对已经高度压缩的 PDF（比如已经是 72 dpi 的扫描件），再压也压不了多少。

## 小结

PDF 压缩这件事，用得最多的就是这一条命令。把常用的预设记在脑子里，需要的时候敲出来就行。
