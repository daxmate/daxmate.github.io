---
layout: single
title:  "macOS 上管理后台服务：launchctl 入門"
date:   2026-06-29 14:00:00 +0800
categories:
  - macOS
tags:
  - macOS
  - launchctl
  - 后台服务
  - 系统管理
---

macOS 没有 systemd，也没有 systemctl。它有自己的守护进程管理系统——**launchd**，而 `launchctl` 就是和它打交道的命令行工具。

如果你需要在后台跑一个服务（比如自建 Aria2、Homebrew 装的服务、或者自己写的脚本常驻），大概率会碰到它。

本文不讲怎么写 plist 文件，先讲最常用的操作——查、启、停、看日志。

---

## 基本概念

在 launchd 的世界里，服务通过 **plist 配置文件**来描述，存放在几个固定的目录：

| 目录 | 用途 |
|---|---|
| `~/Library/LaunchAgents/` | **当前用户**登录后启动的 agent |
| `/Library/LaunchAgents/` | 所有用户登录后启动的 agent（需要 root） |
| `/Library/LaunchDaemons/` | **系统级**守护进程（系统启动时加载，不需要用户登录） |
| `/System/Library/LaunchAgents/` | 系统自带的服务，别动 |
| `/System/Library/LaunchDaemons/` | 系统自带的后台服务，也别动 |

> **Agent vs Daemon：** Agent 有 GUI 环境权限（能访问窗口、剪贴板等），Daemon 纯后台运行。日常个人工具绝大多数用 LaunchAgent 就够了。

### 命名惯例

plist 文件名通常是 **反向域名** 风格，比如：

```
homebrew.mxcl.nginx.plist
com.github.aria2.plist
local.my-script.plist
```

---

## 查看服务状态

### 列出所有加载的服务

```zsh
launchctl list
```

会看到一长串。格式大致是：

```
PID	Status	Label
12345	0	homebrew.mxcl.nginx
-	0	com.apple.mDNSResponder
-	78	com.example.failed-service
```

- **PID** — 正在运行的进程 ID，`-` 表示已加载但没运行（等待触发条件）
- **Status** — 上次退出码，`0` 是正常退出，非零是错误
- **Label** — 服务的唯一标识名

### 只看当前用户的

加 `gui/$(id -u)` 过滤：

```zsh
launchctl list gui/$(id -u)
```

### 按 label 搜服务

```zsh
launchctl list | grep aria2
```

### 查单个服务的详细信息

```zsh
launchctl print gui/$(id -u)/homebrew.mxcl.aria2
```

会显示这个服务的完整配置：plist 路径、程序路径、参数、环境变量、状态等。遇到问题先跑这个看看。

---

## 启动和停止服务

### 手动加载（启动）

```zsh
launchctl load ~/Library/LaunchAgents/local.my-script.plist
```

`load` 会把服务注册到 launchd 并立即启动（如果 plist 里配了 `RunAtLoad`）。

### 手动卸载（停止）

```zsh
launchctl unload ~/Library/LaunchAgents/local.my-script.plist
```

从 launchd 注销这个服务，对应的进程也会被停止。

### 临时启停

也可以先加载了服务，之后再手动启停：

```zsh
# 启动（服务必须在已加载状态）
launchctl kickstart gui/$(id -u)/homebrew.mxcl.aria2

# 停止
launchctl kill TERM gui/$(id -u)/homebrew.mxcl.aria2
```

`kill` 在这里不是杀进程，是给 launchd 发信号让它管理这个服务。可以发 `TERM` 停掉，下一次触发条件满足时又会自动起来。如果想永久停——用 `unload`。

---

## 查看日志

服务输出（stdout/stderr）默认不走普通文件日志，走 **system log**。

```zsh
# 看某条服务的日志
log show --predicate 'subsystem == "com.apple.launchd" AND message CONTAINS "aria2"' --last 1h

# 或者直接看统一日志，grep 出来
log stream --predicate 'process == "aria2c"' --style syslog
```

如果你的 plist 里配置了 `StandardOutPath` 和 `StandardErrorPath`，那输出会写到指定的文件，直接 `cat` 或 `tail -f` 就行。

---

## 推荐做法

### 一、用 brew services

Homebrew 装的服务（nginx、mysql、redis 等），尽量用 `brew services`，不用手写 `launchctl`：

```zsh
brew services start nginx
brew services stop nginx
brew services restart nginx
brew services list
```

只是背后调的还是 launchctl。装完服务它会提示让你运行：

```
==> SUMMARY
Run 'brew services start nginx' to start nginx now!
```

### 二、自己的脚本放 ~/Library/LaunchAgents/

自制工具建议放用户目录下，不需要 sudo：

```
~/Library/LaunchAgents/local.my-script.plist
```

用 `local.` 前缀命名，不和系统的冲突。

### 三、加载完确认一下

```zsh
launchctl list | grep my-script
```

看到 PID 不为 `-` 说明在跑。

### 四、改完 plist 要重载

修改了 plist 文件，必须先 `unload` 再 `load`：

```zsh
launchctl unload ~/Library/LaunchAgents/local.my-script.plist
launchctl load ~/Library/LaunchAgents/local.my-script.plist
```

---

## 补充

### 一次搞定 load/unload

`-w` 参数可以同时启用/停用某个路径，本质上是在 Override 数据库添加一条记录：

```zsh
launchctl load -w ~/Library/LaunchAgents/something.plist
launchctl unload -w ~/Library/LaunchAgents/something.plist
```

不过日常使用，裸 `load`/`unload` 够了，`-w` 只有当你需要「禁用但保留 plist 文件不动」的时候才用到。

### 调试技巧

- `launchctl print` 看当前状态
- `launchctl error <code>` 解释退出码
- `launchctl list` 看到 Status 列有值 → 说明进程跑失败了

---

## 总结

| 操作 | 命令 |
|---|---|
| 列出所有服务 | `launchctl list` |
| 加载并启动 | `launchctl load <plist>` |
| 停止并卸载 | `launchctl unload <plist>` |
| 查详情 | `launchctl print gui/$(id -u)/<label>` |
| 看日志 | `log show --predicate ...` |

后面我们会写一篇具体的例子——用 launchctl 把 Aria2 配置成开机自启的后台服务。等下次再聊 👋
