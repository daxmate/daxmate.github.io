---
layout: single
title:  "macOS 上用 launchctl 配置 Aria2 开机自启"
date:   2026-06-29 14:10:00 +0800
categories:
  - macOS
  - Tooling
tags:
  - aria2
  - launchctl
  - 下载工具
  - https
  - ssl
---

[Aria2](https://aria2.github.io/) 是一个轻量级的多协议下载工具，支持 HTTP/HTTPS、FTP、SFTP、BitTorrent、Metalink。没有 GUI，没有花里胡哨的东西，就是个命令行级的下载引擎。

在 macOS 上跑 Aria2，最顺手的做法是写成 launchd 服务——开机自启、后台运行、通过 RPC 接口调它。

本文记录完整的配置过程。

> 如果对 launchctl 不熟，可以先看 macOS 上管理后台服务：launchctl 入门。

---

## 安装

```zsh
brew install aria2
```

装完后确认：

```zsh
aria2c --version
```

---

## 写配置文件

Aria2 启动参数挺长的，全塞命令里不方便。推荐把配置写到一个文件里。

创建一个配置目录和文件：

```zsh
mkdir -p ~/.config/aria2
```

编辑 `~/.config/aria2/aria2.conf`：

```
# 下载目录
dir=/Users/dax/Downloads

# 日志
log=/Users/dax/.config/aria2/aria2.log
log-level=notice

# 启用 RPC
enable-rpc=true
rpc-listen-all=false
rpc-listen-port=6800
rpc-secret=YOUR_SECRET_HERE

# 最大连接数
max-concurrent-downloads=5
max-connection-per-server=16
continue=true

# BT
bt-enable-lpd=true
bt-max-peers=50
follow-torrent=true
```

几个要点：

- `rpc-listen-all=false` 只监听本机，不暴露给局域网
- `rpc-secret` 设一个密码，别用默认值
- 路径都用绝对路径，launchd 的环境变量和终端不一样，相对路径可能找不到

---

## 写 plist 文件

创建 `~/Library/LaunchAgents/local.aria2.plist`：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>local.aria2</string>

    <key>ProgramArguments</key>
    <array>
        <string>/opt/homebrew/bin/aria2c</string>
        <string>--conf-path=/Users/dax/.config/aria2/aria2.conf</string>
    </array>

    <key>RunAtLoad</key>
    <true/>

    <key>KeepAlive</key>
    <false/>

    <key>StandardOutPath</key>
    <string>/Users/dax/.config/aria2/aria2.log</string>

    <key>StandardErrorPath</key>
    <string>/Users/dax/.config/aria2/aria2.log</string>
</dict>
</plist>
```

说明：

- **Label** — 服务的唯一标识符，用 `local.aria2`，不和系统自带的冲突
- **ProgramArguments** — 要执行的命令和参数，路径要写全（`/opt/homebrew/bin/aria2c`），launchd 不加载 shell 的 PATH
- **RunAtLoad** — 加载后立即启动
- **KeepAlive** — 进程挂了是否自动重启。Aria2 如果正常退出不用自动复活，设为 `false`
- **StandardOutPath / StandardErrorPath** — 输出日志。这里和 Aria2 自己的日志合并到同一个文件

> 注意：如果你的 Mac 是 Intel 芯片，brew 路径是 `/usr/local/bin/aria2c`；如果是 Apple Silicon，是 `/opt/homebrew/bin/aria2c`。用 `which aria2c` 确认一下。

---

## 加载并启动

```zsh
launchctl load ~/Library/LaunchAgents/local.aria2.plist
```

确认在跑：

```zsh
launchctl list | grep aria
```

应该看到类似：

```
87654	0	local.aria2
```

前面是 PID，中间 `0` 是退出码（没退出的就是 0），后面是 label。

再看看日志：

```zsh
cat ~/.config/aria2/aria2.log
```

看到 `aria2c started` 之类说明成功了。

---

## 测试 RPC

用 curl 向 Aria2 的 RPC 接口发请求试试：

```zsh
curl -X POST http://localhost:6800/jsonrpc \
  -H 'Content-Type: application/json' \
  -d '{
    "jsonrpc": "2.0",
    "method": "aria2.getVersion",
    "id": 1,
    "params": ["token:YOUR_SECRET_HERE"]
  }'
```

返回：

```json
{"id":1,"jsonrpc":"2.0","result":{"enabledFeatures":["Async DNS", ...],"version":"1.37.0"}}
```

能拿到版本信息，说明一切正常。

### 实际下载试试

```zsh
curl -X POST http://localhost:6800/jsonrpc \
  -H 'Content-Type: application/json' \
  -d '{
    "jsonrpc": "2.0",
    "method": "aria2.addUri",
    "id": 1,
    "params": ["token:YOUR_SECRET_HERE", ["https://example.com/some-file.zip"]]
  }'
```

返回类似：

```json
{"id":1,"jsonrpc":"2.0","result":"2089b05e5b3fb826"}
```

那个 hash 就是下载任务的 GID，可以用它查状态：

```zsh
curl -X POST http://localhost:6800/jsonrpc \
  -H 'Content-Type: application/json' \
  -d '{
    "jsonrpc": "2.0",
    "method": "aria2.tellStatus",
    "id": 1,
    "params": ["token:YOUR_SECRET_HERE", "2089b05e5b3fb826"]
  }'
```

---

## 日常管理

```zsh
# 停止（但保留 plist）
launchctl kill TERM gui/$(id -u)/local.aria2

# 重新启动
launchctl kickstart gui/$(id -u)/local.aria2

# 彻底卸载
launchctl unload ~/Library/LaunchAgents/local.aria2.plist
```

如果想启用 daemon 模式（崩溃后自动重启），把 plist 里的 `KeepAlive` 改成 `<true/>`，然后重载：

```zsh
launchctl unload ~/Library/LaunchAgents/local.aria2.plist
launchctl load ~/Library/LaunchAgents/local.aria2.plist
```

---

## 可能遇到的问题

### 「找不到程序」

```zsh
launchctl list | grep aria
```

如果看到 Status 是 `78` 之类的错误码，用 `launchctl error 78` 查一下含义：

```zsh
launchctl error 78
# 2: No such file or directory
```

路径写错了。检查 plist 里 `aria2c` 的路径是否和实际一致。

### 「Connnection refused」

RPC 请求返回 `Connection refused`：

1. 确认 Aria2 在跑：`launchctl list | grep aria`
2. 检查日志：`cat ~/.config/aria2/aria2.log`
3. 看端口是不是被占：`lsof -i :6800`

### 「权限不足」

如果要把 plist 放 `/Library/LaunchDaemons/` 做成系统级服务，需要 sudo 加载。不过个人用放 `~/Library/LaunchAgents/` 就够，还省去了权限麻烦。

---

## 配合下载工具

配置好 RPC 之后，可以配合图形前端用：

- **Aria2 GUI** — 浏览器里的 Web UI
- **AriaNg** — [GitHub](https://github.com/mayswind/AriaNg)，纯前端，更现代化
- **Folx** 等第三方工具也可以设代理

AriaNg 最推荐——下载一个 `AriaNg-{version}.zip`，解压用浏览器打开 `index.html`，在设置里填 RPC 地址 `http://localhost:6800/jsonrpc` 和 secret token，就能在浏览器里管理下载任务了。

---

## 总结

| 步骤 | 操作 |
|---|---|
| 安装 | `brew install aria2` |
| 写配置 | `~/.config/aria2/aria2.conf` |
| 写 plist | `~/Library/LaunchAgents/local.aria2.plist` |
| 加载 | `launchctl load ~/Library/LaunchAgents/local.aria2.plist` |
| 检查 | `launchctl list \| grep aria` |
| 测试 RPC | `curl` 发 JSON-RPC 请求 |

配置好之后，Aria2 就在后台静静呆着了。需要下载的时候通过 RPC 发任务，不用开着终端窗口，也不用每次都敲命令。省心。
