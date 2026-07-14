# Illustration Outline — LaTeX latexmk Article

## Settings
- type: comparison, flowchart, infographic
- style: editorial
- count: 3
- output: `assets/images/latexmk/`

## Illustrations

### 1. 01-comparison-compilation-pipeline.svg
**Type:** comparison
**Position:** After "直接从 DVI 到 PDF" subsection
**Purpose:** Show the contrast between old multi-step compilation pipelines and modern direct compilation
**Content:**
- Left column: "传统编译路径"
  - Path A (模式 2): .tex → .dvi → .ps → .pdf (tex → dvips → ps2pdf)
  - Path B (模式 3): .tex → .dvi → .pdf (tex → dvipdfmx)
  - Extra box: 中间文件泛滥 (.aux, .toc, .bbl...)
- Right column: "现代直接编译"
  - Three direct paths: pdflatex → .pdf, lualatex → .pdf, xelatex → .pdf
  - Single step, clean
- Visual: arrows showing pipeline length difference, "vs" divider in center

### 2. 02-flowchart-latexmk-loop.svg
**Type:** flowchart
**Position:** After "怎么判断的" section
**Purpose:** Visualize latexmk's intelligent decision loop
**Content:**
- Start node: 开始编译
- Process: 运行 LaTeX 引擎
- Scan .log node (with magnifying glass icon text)
- Decision diamond: 日志有 "Rerun" 提示?
- Yes → check: 需要 bibtex/biber? → run auxiliary tool → loop back to engine
- No → 完成
- Edge case: 已达最大编译次数? → 强制停止
- Labels with actual trigger strings from .log

### 3. 03-infographic-config-overview.svg
**Type:** infographic
**Position:** In "选择编译引擎" subsection
**Purpose:** Summarize .latexmkrc configuration variables
**Content:**
- Central "latexmkrc" box
- Surrounding nodes:
  - `$pdf_mode` (0-5) with each engine labeled
  - `$xelatex` etc. with parameters (-synctex=1, -interaction=nonstopmode)
  - `$out_dir` / `$aux_dir` (build directory)
  - `$pdf_previewer` (PDF viewer)
  - `@generated_exts` (cleanup rules)
- Color coding by category: engine (process), params (input), output (storage), tools (external)
