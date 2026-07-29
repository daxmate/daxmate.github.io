---
layout: single
title:  "Zsh 补全：\_arguments 使用总结"
date:   2026-07-17 17:44:00 +0800
categories:
  - Command Line
tags:
  - 命令行
  - zsh
  - 补全
---

> 基于 xxd / ls / npm / pnpm 补全编写经验

---

## 一、补全文件结构

```zsh
#compdef 命令名

_xxx() {
  local -a options
  options=(
    # 选项定义...
    '*:位置参数:_files'
  )
  _arguments -s $options && return

  case "$state" in
    something) _something ;;
  esac
}
```

---

## 二、选项定义格式

### 基本格式

```zsh
'互斥列表-选项名[描述]:消息:动作'
```

### 布尔选项

```zsh
'-v[显示版本信息]'
'(-f --force)-f[强制模式]'
```

### 带值选项

```zsh
'--output=[输出文件]:文件:_files'
'--format=[格式]:格式:(json yaml text)'
'-D[日期格式]:格式字符串'
```

### 长短选项合一

长选项为主条目，短选项写在描述开头括号里：

```zsh
# ✅ 正确
'(-v --version)--version[(-v) 显示版本信息]'
'(-g --global)--global[(-g) 全局操作]'

# ❌ 避免（花括号别名会导致分行显示）
{-v,--version}'[显示版本信息]'
```

互斥列表 `(-g --global)` 防止同一个选项重复使用，**必须保留**。

### 互斥列表

```zsh
'(-A -a -I)-A[包含隐藏文件（不含 . 和 ..）]'  # -A 与 -a -I 互斥
'(- *)-h[显示帮助并退出]'                    # (- *) 与所有选项互斥
```

- `-a` 要与谁互斥就写在 `(-a)` 里
- `(- *)` 表示与所有选项互斥（用于 `--help`、`--version` 等）

### 状态分发（动态补全）

```zsh
'--device=[设备]:设备:->devices'
```

配合 `case "$state"` 动态生成补全列表：

```zsh
_arguments -s $options && return

case "$state" in
  devices)
    local -a devs
    devs=( $(command --list-devices) )
    _describe '设备' devs
    ;;
esac
```

---

## 三、\_arguments 自身选项

| 选项 | 含义 | 适用场景 |
|------|------|---------|
| `-s` | 允许选项堆叠（`-abc` = `-a -b -c`） | 单字母选项多的命令 |
| `-C` | 遇到 `->state` 后修改 curcontext | 嵌套子命令（如 git） |

### `-C` 的详细说明

- **不加 `-C`**：curcontext 保持 `:completion:...:命令名`，`case "$state"` 正常命中
- **加 `-C`**：触发 `->state` 时 curcontext 变为 `:completion:...:命令名:option`

**建议：简单命令不要加 `-C`**，原因：
1. `-C` 设计初衷是嵌套子命令（`git commit` / `git checkout`），单层命令画蛇添足
2. 可能干扰用户自定义的 zstyle 样式匹配
3. `case "$state"` 已足够处理状态分发

### 关于 `-Cs` 不能合并

```zsh
# ❌ 错误 — _arguments 自身的选项不支持堆叠
_arguments -Cs "${options[@]}"

# ✅ 正确 — 必须分开
_arguments -C -s "${options[@]}"
```

原因：`_arguments` 解析自身参数的 while 循环使用 `[[ "$1" = -([AMO]*|[0CRSWnsw]) ]]` 匹配，`-Cs` 作为整体不会匹配到 `-C` 或 `-s`。

---

## 四、数组展开陷阱

```zsh
# ❌ 数组含空格元素时，$options 会单词切割
options=('(- *)-h[帮助]')
_arguments -s $options    # 展开成 (- 和 *)-h[帮助] 两个词 ✗

# ✅ 用 "${options[@]}" 保持每个元素完整
_arguments -s "${options[@]}"

# ⚠️ 若数组元素都不含空格，$options 也可用
# _gs 的选项定义不含空格，所以 $options 没问题
```

---

## 五、常见错误

### 1. 花括号别名在数组里炸开

```zsh
# ❌ 数组里不能用花括号别名
options=('(- *)'{-h,--help}'[帮助]')

# ✅ 简单写法，别名标注放描述里
options=('(- *)-h[(-h --help) 帮助]')
```

### 2. 描述中的冒号被 \_arguments 解析

```zsh
# ❌ 描述含冒号，_arguments 误认为参数分隔符
'(-c)-cols[(-c) 每行显示字节数]:列数（默认值 16...）'

# ✅ 简洁描述，参数用 = 标明
'(-c)-cols=[(-c) 每行的字节数]'
```

### 3. 选项本身在互斥列表里

```zsh
# ❌ 自互斥
'(-e -p ... -i)-e[描述]'

# ✅ 去掉自身
'(-p ... -i)-e[描述]'
```

### 4. 命令不支持 `=`

`-cols=16` 是 zsh 补全中表示「选项带值」的写法，不一定被目标命令支持。xxd 只接受 `-cols 16`（空格分隔），不接受 `-cols=16`。所以应写 `-cols[描述]:列数` 而非 `-cols=[描述]:列数`。

---

## 六、经验总结

| 概念 | 做法 |
|------|------|
| 文件结构 | `local -a options` + `_arguments -s $options && return` |
| 选项定义 | 长选项为主，短选项放描述 `(-x)` |
| 互斥列表 | 保留 `(-a -b)` 防止重复 |
| 别名 | 不要在数组里用 `{-h,--help}` |
| `-C` | 简单命令不加 |
| 数组展开 | 含空格用 `"${options[@]}"`，不含空格用 `$options` |
| 带值选项 | 用 `[desc]:msg` 或 `[desc]:msg:action` |
| 自互斥 | 互斥列表里不能有选项自己 |

> [← 查看系列目录]({% link _pages/series-command-line.md %})
