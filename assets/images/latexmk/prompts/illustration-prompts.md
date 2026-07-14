# Illustration Prompts — latexmk Article

## Overview
- Topic: latexmk LaTeX compilation tool
- Style: editorial (tech explainer)
- Output: standalone SVG, no external dependencies

## Illustration 1 — Compilation Pipeline Comparison

**File:** `01-comparison-compilation-pipeline.svg`
**Type:** comparison
**Purpose:** Show contrast between old multi-step pipeline and modern direct compilation

**Prompt summary:**
- Left side (red-tinted background): Traditional compilation path showing .tex → .dvi → .ps → .pdf via tex/dvips/ps2pdf and alternate .tex → .dvi → .pdf via dvipdfmx. Highlights "模式 2" and "模式 3" labels. Warning about intermediary file pollution (.aux, .toc, .bbl).
- Right side (green-tinted background): Modern direct compilation with three parallel columns — pdflatex, lualatex, xelatex — each going directly from .tex to .pdf in one step.
- Center: "VS" badge divider.
- Footer: "传统：3~4 步 + 手动编排" vs "现代：1 步，latexmk 自动完成".

## Illustration 2 — latexmk Decision Loop

**File:** `02-flowchart-latexmk-loop.svg`
**Type:** flowchart
**Purpose:** Visualize latexmk's intelligent compilation loop

**Prompt summary:**
- Top-down flowchart with these nodes:
  1. Start ellipse → "开始编译"
  2. Process rectangle → "运行 LaTeX 引擎"
  3. Process rectangle → "扫描 .log 文件"
  4. Decision diamond → "有 'Rerun' 提示？"
     - NO branch → "编译完成" (ellipse, green)
     - YES branch → check "缺引用？缺索引？" box
       - Need Biber/BibTeX → "运行 Biber / BibTeX / Makeindex"
       - No auxiliary needed → dashed line directly to loop-back
  5. Loop-back arrow returning to "运行 LaTeX 引擎"
  6. Side note: "已达最大编译次数（默认 5~10）" as guard rail
  7. Side panel listing trigger strings from .log: "Rerun to get cross-references", etc.
  8. Footer: -pvc mode tip

## Illustration 3 — .latexmkrc Config Overview

**File:** `03-infographic-config-overview.svg`
**Type:** infographic
**Purpose:** Summarize key configuration variables

**Prompt summary:**
- Hub-and-spoke layout
- Center: "~/.latexmkrc" in dark card with "Perl 配置文件" subtitle
- Surrounding categories (clockwise from top-left):
  1. "编译引擎" (blue): $pdf_mode = 1/4/5 with engine names
  2. "编译参数" (purple): -synctex=1, -interaction=nonstopmode, -file-line-error
  3. "输出目录" (green): $out_dir, $aux_dir → 'build'
  4. "PDF 预览器" (yellow): $pdf_previewer example
  5. "清理规则" (red): @generated_exts, $clean_ext
- Bottom zone: "latexmk -pvc -pdf main.tex" as daily use command
- Footer: latexmk -C (clean) and latexmk -g (force re-run)
