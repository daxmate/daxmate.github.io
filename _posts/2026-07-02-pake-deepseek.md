---
layout: single
title:  "把 DeepSeek 网页打包成桌面 App——Pake 真好用"
date:   2026-07-02 18:09:00 +0800
categories:
  - Tooling
tags:
  - macOS
  - Pake
  - DeepSeek
  - Tauri
---

一直以来我用 DeepSeek 都是在浏览器里开一个标签页，不算麻烦，但总觉得少点什么——不能 Cmd+Tab 切过去，混在一堆网页里不好找，偶尔手滑关掉标签页还得重新打开。

今天发现了一个小工具——**Pake**。它做的事很简单：把一个网页包成一个能独立打开的桌面 App。

## Pake 是什么

Pake 是一个命令行工具，用 Rust + Tauri 写成。跟 Electron 的思路类似，但区别很大：

- Electron 会给每个 App 打包一个完整的 Chromium 浏览器，体积 100MB 起步
- Pake 用的是操作系统自带的 WebView（macOS 上就是 Safari 的 WebKit），整个 App 只有几 MB

所以用 Pake 打包出来的 App 启动飞快、不占内存，而且跟原生应用体验一样——macOS 上能出现在 Dock 里、能用 Cmd+Tab 切换，Windows 和 Linux 上也同样适用。

GitHub 仓库：[tw93/Pake](https://github.com/tw93/Pake)

## 安装

最省事的方式是用 Homebrew：

```zsh
brew install pake
```

如果不用 Homebrew，也可以用 npm：

```zsh
npm install -g pake-cli
```

## 打包 DeepSeek

一行命令的事：

```zsh
pake https://chat.deepseek.com --name DeepSeek
```

等几十秒，一个叫 `DeepSeek.dmg` 的安装包出现在当前目录，自动弹出安装界面，拖进 Applications 文件夹就完事。

打开之后就是一个干净的 DeepSeek 窗口——没有浏览器地址栏、没有多余的按钮栏，就是一个纯粹的 DeepSeek 界面。跟原生 App 的体验几乎一样。

如果想把标题栏也藏掉，加个参数就行：

```zsh
pake https://chat.deepseek.com --name DeepSeek --hide-title-bar
```

## 为什么值得用

部署完之后用了半天，几个感受：

1. **Cmd+Tab 能切到了。** 这是最直观的提升——不用再在一堆浏览器标签页里找 DeepSeek
2. **不沾浏览器。** 关掉 Chrome/Safari 的时候 DeepSeek 还在，互不影响
3. **干净的界面。** 没有书签栏、扩展图标、地址栏——真正沉浸式的聊天窗口
4. **内存开销低。** 因为是 WebView 而不是 Chromium，内存占用只有几十 MB

## 几个好用的选项

上面的 `--name` 和 `--hide-title-bar` 只是冰山一角。Pake 支持的选项不少，挑几个常用的说：

```zsh
pake https://chat.deepseek.com \
  --name DeepSeek \
  --width 1200 \
  --height 900 \
  --hide-title-bar \
  --always-on-top \
  --install
```

### 窗口控制

| 选项 | 作用 |
|------|------|
| `--width 1200` | 窗口宽度 |
| `--height 900` | 窗口高度 |
| `--fullscreen` | 启动时全屏 |
| `--always-on-top` | 窗口置顶 |

### 打包方式

| 选项 | 作用 |
|------|------|
| `--install` | 直接安装到 /Applications，省去拖拽那一步 |
| `--multi-arch` | 同时打包 Intel 和 Apple Silicon 版本 |

### 外观

| 选项 | 作用 |
|------|------|
| `--icon ./logo.png` | 自定义图标（不传就用网站自己的 favicon） |
| `--zoom 110` | 缩放比例（百分比），有些网页默认太小可以调大 |
| `--hide-title-bar` | 隐藏标题栏，沉浸式（仅 macOS） |

### 其他

| 选项 | 作用 |
|------|------|
| `--inject style.css` | 注入自定义 CSS 或 JS，修改网页外观 |
| `--proxy-url http://127.0.0.1:7890` | 走代理 |
| `--debug` | 开启开发者工具，右键能调出审查元素 |

我日常用就加 `--install` 和 `--hide-title-bar` 两个，其他的按需加。

## 不止 DeepSeek

Pake 的 GitHub Release 页面上已经有一堆预打包好的 App——ChatGPT、Claude、YouTube、YouTube Music、Twitter/X……可以直接下载用。

也可以自己打包任意网页。比如把常用的在线工具、内部看板打包成桌面应用——只要是个 URL 就能打包。

## 小结

`pake <URL>` 一行命令，把一个网页变成桌面 App。适合那些每天都要开、但又不想要浏览器包袱的网页工具。

DeepSeek 的网页版体验本来就不错，套上 Pake 之后就像有了一个真正属于自己的 AI 桌面助手。
