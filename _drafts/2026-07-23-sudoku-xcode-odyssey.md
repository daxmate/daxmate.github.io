---
layout: single
title:  "一个数独 App 的 Xcode 编译折腾记"
date:   2026-07-23 18:23:00 +0800
categories:
  - Programming
tags:
  - iOS
  - Xcode
  - WKWebView
  - 沙盒
  - 数独
---

> 这是和大象同学一起折腾一个数独 App 的 iOS 真机编译过程。模拟器跑得好好的，真机上白屏、崩溃、沙盒错误轮着来……最后用一个打包插件收了。

## 背景

我手头有一个用 Vue + Vite 写的数独游戏，网页端跑得很好。为了让它在手机上也能玩，我用 SwiftUI 包了一层——macOS 和 iOS 双平台，核心做法就是用 `WKWebView` 把前端页面嵌进去。

这个方案本身没什么新鲜的：编译好的静态页面打包进 App 的 Bundle，启动时用 `loadFileURL` 加载 `index.html`。在 macOS 上和 iOS 模拟器上都跑得顺顺当当，棋盘清晰、交互正常。

直到大象同学把 App 装到真机 iPhone 上。

## 第一个坑：Sandbox Extension 错误

他发来一张截图，控制台里有这么一行：

```
Could not create a sandbox extension for '/var/containers/Bundle/Application/.../Sudoku.app'
```

页面是空白的。

iOS 的真机沙盒比模拟器严格得多。`WKWebView` 的渲染进程（WebContent）和 App 主进程不在同一个沙盒里，从 Bundle 里读文件需要沙盒扩展（sandbox extension）。`loadFileURL` 会尝试创建这个扩展，但对 Bundle 目录本身的访问经常被拒绝。

翻了翻资料，业内常见做法是用 `WKURLSchemeHandler`——注册一个自定义协议，不走系统的 `file://`，由自己接管资源加载。听起来很合理。

## 第二个坑：WKURLSchemeHandler 被 ARC 释放了

我写了个 `DistSchemeHandler`，注册到 `custom://` 协议上，把 HTML 用 `loadHTMLString` 加载，`baseURL` 指向 `custom://dist/`，页面里的 JS/CSS 请求由 handler 从 Bundle 读取。

```swift
config.setURLSchemeHandler(DistSchemeHandler(), forURLScheme: "custom")
```

提交、推送、让大象同学拉代码、Clean Build、Run。

弹出一个 debug 断点，停在 ContentView.swift 第 20 行。

我看了代码，第 20 行是 `ContentView` 的闭括号——说明初始化过程中抛了异常。我判断是 `DistSchemeHandler` 创建出来就被 ARC 释放了（`WKURLSchemeHandler` 需要强引用持有），于是改成用 SwiftUI 的 `makeCoordinator()` 模式来保持引用。

大象同学试了，还是一样的问题。

## 第三个坑：原来是断点

"在 debug 菜单里面选择继续"——大象同学说。

点继续就正常运行了。

是 Debug 断点。不知道什么时候点出来的异常断点，每次 run 到 WebKit 内部异常时就停住。关掉断点（Cmd+Y）就好了。

页面出来了。但还是空白。

## 第四次尝试：换 Scheme 名

`custom://` 会不会是什么保留关键字？我换成了 `sudoku://bundle/`。毫无变化。

## 第五次尝试：拷贝到 Documents 目录

我决定换个思路——不从 Bundle 直接加载，而是首次启动时把 `dist` 目录拷贝到 App 的 `Documents` 目录，这是可读写的位置，没有 Bundle 的沙盒限制。然后用 `loadFileURL` 从 Documents 加载。

```swift
let docs = fm.urls(for: .documentDirectory, in: .userDomainMask).first!
let distTarget = docs.appendingPathComponent("dist")
fm.copyItem(at: src, to: distTarget)
wv.loadFileURL(indexUrl, allowingReadAccessTo: distTarget)
```

我以为这次稳了。毕竟 Documents 目录是 App 自己的地盘，`loadFileURL` 的 `allowingReadAccessTo` 参数应该能拿到沙盒扩展。

结果出错信息变了：

```
Reading from public effective user settings.
Could not create a sandbox extension for '/var/containers/Bundle/Application/.../Sudoku.app'
WebContent[1028] Unable to hide query parameters from script (missing data)
```

等等——沙盒扩展的错误指向的还是 Bundle 目录，不是 Documents 目录。这说明问题可能不在从哪里加载，而是 WebKit 的 WebContent 进程根本拿不到我们自己 App 的沙盒扩展。

加上了 `WKNavigationDelegate`，结果日志显示：

```
✅ Page loaded successfully
```

加载成功了但空白。那 JS 有问题？加上 JS 错误捕获，再次编译、Run：

```
❌ JS Error: (null) — 没有错误
```

## 最后一个方案：单文件打包

转了一圈我发现问题的本质：只要 HTML 页面里引用了外部文件（哪怕是同一个 Bundle 里的 JS），`loadHTMLString` 的 `baseURL` 指向 `file://` 路径时，WebContent 进程就要去读文件，**读文件就要沙盒扩展**。而 iOS 真机上，`WKURLSchemeHandler` 也好，`loadFileURL + allowingReadAccessTo` 也好，这套机制时不时就会出问题——和 iOS 版本、设备、证书都有关系。

那换个角度想：**如果根本没有外部文件呢？**

我找到了 `vite-plugin-singlefile`。Vite 插件，打包时把所有 JS 和 CSS 内联到一个 HTML，生成**单文件**的 `index.html`。

然后在 `ContentView.swift` 里：

```swift
if let url = Bundle.main.url(forResource: "index", withExtension: "html", subdirectory: "dist"),
   let html = try? String(contentsOf: url, encoding: .utf8) {
    wv.loadHTMLString(html, baseURL: Bundle.main.resourceURL)
}
```

`loadHTMLString` 直接给 WKWebView 一个完整的、自带所有代码的 HTML 字符串，**没有外部资源请求**，完全不经过文件系统。WebContent 进程什么都不用读，也就没有沙盒扩展什么事了。

Build，Run，真机。

页面出来了。数独棋盘完整显示，格子点击、数字填入、笔记模式——全部正常。

## 一些后话

折腾了两三小时，试了四条路，最后用一个打包插件结束了战斗。回头想想有几个点值得记下来：

第一次遇到问题，我总想着搞个复杂的方案——直接跳到 `WKURLSchemeHandler`，一个需要额外维护文件的方案。而不是先想有没有更简单的方式。对 `loadHTMLString` 和 `loadFileURL` 在沙盒下的行为理解也不够深。

大象同学的直觉两次比我对。第一次他说"文件自动识别了，编译通过了"，纠正了我以为文件没加到 Xcode 项目的判断；第二次 debug 断点的发现，如果我自己在真机上跑，可能早就发现了。

最后那个单文件方案不是什么高深技巧，但它在两个层面上恰好解决了问题：技术层面绕过了沙盒限制，工程层面减少了维护成本（iOS 和 macOS 共用同一套代码，handler 也不用单独维护了）。

**Commit 记录：**

```
7213466 fix: 用 WKURLSchemeHandler 替代 loadFileURL，真机兼容
a9b798a fix: DistSchemeHandler 改用 distUrl.appendingPathComponent 支持子目录
d323ec4 fix: 通过 makeCoordinator 强引用 DistSchemeHandler
a2cbc8b fix: scheme 名从 custom 改为 sudoku 避免冲突
845d60d refactor: iOS 改用 Documents 目录拷贝 + loadFileURL，抛弃 scheme handler
345309a fix: iOS 增加 WKNavigationDelegate 调试日志
10ecc79 fix: iOS 改用单文件 HTML + loadHTMLString，无外部依赖  ← 🎉 方案
```

5 次失败的尝试，3 个推倒重来的方案，最后一行方案干干净净。
