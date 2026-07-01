---
layout: single
title:  "从零开始学命令行：网络工具"
date:   2026-07-01 14:00:00 +0800
categories:
  - Command Line
tags:
  - 命令行
  - curl
  - ping
  - ssh
  - ifconfig
  - 网络
---

命令行的世界不只是管管自己电脑上的文件。有时候要从网上下载东西、发 HTTP 请求、连远程服务器——这篇把最常用的几个网络命令捋一遍。

---

## 下载文件：`curl`

`curl` 是最常用的命令行网络工具。最基础的用法：

```zsh
curl https://example.com
```

屏幕上会打印出 example.com 的 HTML 源码。

### 下载并保存到文件

```zsh
curl -o page.html https://example.com
```

`-o` 指定保存的文件名。如果不给文件名，用 `-O`（大写）会用 URL 里的文件名：

```zsh
curl -O https://example.com/file.zip
```

### 只显示响应头

```zsh
curl -I https://example.com
```

```
HTTP/2 200
content-type: text/html
...
```

调试 API、看服务器返回了什么状态码的时候特别有用。

### 发送 POST 请求

```zsh
curl -X POST -d "name=大象&comment=不错" https://example.com/api
```

`-X` 指定 HTTP 方法，`-d` 是发送的数据。

发送 JSON：

```zsh
curl -X POST -H "Content-Type: application/json" \
  -d '{"name":"大象","score":100}' \
  https://example.com/api
```

`-H` 是添加自定义请求头。

### 跟随重定向

```zsh
curl -L https://bit.ly/xxxx
```

`-L` 会让 `curl` 自动跟随重定向，一直跟到最终地址。

### 静默模式

```zsh
curl -s https://example.com
```

`-s` 不显示进度条。配合脚本使用的时候基本都要加。

---

## 测试网络连通：`ping`

```zsh
ping google.com
```

```
PING google.com (142.250.80.46): 56 data bytes
64 bytes from 142.250.80.46: icmp_seq=0 ttl=117 time=43.123 ms
64 bytes from 142.250.80.46: icmp_seq=1 ttl=117 time=42.567 ms
```

`ping` 向目标服务器发 ICMP 包，目标回复，计算往返时间。按 `Ctrl + C` 停止并看统计。

只发固定次数：

```zsh
ping -c 4 google.com
```

发 4 个包就停。

网络不通的时候，先用 `ping` 看看是「完全连不上」还是「慢」。这是排网络故障的第一步。

---

## 远程登录：`ssh`

`ssh` 让你从命令行登录另一台机器。用法：

```zsh
ssh 用户名@主机地址
```

比如：

```zsh
ssh dax@192.168.1.100
```

第一次连接会问是否信任这台机器的指纹——输入 `yes`。然后输入密码就进去了。敲 `exit` 退出。

### 免密登录：SSH Key

每次输密码很麻烦。生成一对密钥，把公钥放到远程机器上，以后就不用输密码了：

```zsh
# 生成本机密钥对（如果还没有的话）
ssh-keygen -t ed25519 -C "your_email@example.com"
# 一路回车就行

# 把公钥复制到远程机器
ssh-copy-id dax@192.168.1.100
```

之后 `ssh dax@192.168.1.100` 直接登入。

### 在远程机器上执行一条命令

```zsh
ssh dax@192.168.1.100 "ls -la /var/log"
```

执行完就退回本地，不用登录进去再退出。

---

## 查看网络信息：`ifconfig`

```zsh
ifconfig
```

显示所有网络接口的信息——IP 地址、MAC 地址、收发数据量。

只看某个接口（比如 Wi-Fi）：

```zsh
ifconfig en0
```

macOS 上 `en0` 通常是 Wi-Fi，`en1` 或有线网。搞不清的时候 `ifconfig` 不带参数全看一遍。

只提取 IP 地址：

```zsh
ifconfig en0 | grep "inet " | awk '{print $2}'
```

---

## 检查端口：`lsof`

想知道某个端口被谁占用了：

```zsh
lsof -i :8080
```

```
COMMAND   PID USER   FD   TYPE    DEVICE SIZE/OFF NODE NAME
python  12345  dax    3u  IPv4  0x...      0t0  TCP *:8080 (LISTEN)
```

PID 是 12345，是 Python 在占用 8080。

经常用到的一个小技巧：

```zsh
lsof -i :8080 | grep LISTEN
```

只看处于 LISTEN（监听）状态的，不会被客户端连接干扰。

---

## 网络连通性诊断：`nc`（netcat）

`nc` 是个简易的网络测试工具。最简单的用法——检查某台机器的某个端口能不能连上：

```zsh
nc -zv example.com 80
```

```
Connection to example.com port 80 [tcp/http] succeeded!
```

`-z` 是「只扫描不发送数据」，`-v` 是详细输出。端口开没开，一秒就知道。

---

## 测试 API

`curl` 在写程序的时候经常用来测试 API。几个常用搭配：

```zsh
# GET 请求，显示响应状态码
curl -s -o /dev/null -w "%{http_code}" https://api.example.com/health

# GET 请求，格式化 JSON（需要装 jq）
curl -s https://api.github.com/repos/openclaw/openclaw | jq .stargazers_count

# 下载大文件，断点续传
curl -C - -O https://example.com/large-file.zip
```

---

## 动手试试

```zsh
# 下载网页
curl -s https://example.com | head -n 10

# 看响应头
curl -I https://example.com

# ping 一下
ping -c 3 baidu.com

# 看你自己的 IP 和网络接口
ifconfig en0 | grep "inet "

# 看看 SSH 密钥有没有
ls -la ~/.ssh/

# 检查一个端口（比如本地 22 端口在不在监听）
lsof -i :22
```

---

## 小结

| 命令 | 作用 | 常用搭配 |
|------|------|----------|
| `curl` | 下载文件、发 HTTP 请求 | `-o` 保存，`-I` 响应头，`-L` 跟随重定向，`-s` 静默 |
| `ping` | 测网络连通性 | `-c 次数` 限制发包 |
| `ssh` | 远程登录 | `ssh 用户@地址`，`ssh-keygen` 配免密 |
| `ssh-copy-id` | 把公钥放远程机器 | 配一次，以后免密登录 |
| `ifconfig` | 查看网络接口 | `grep inet` 提取 IP |
| `lsof -i :端口`| 看端口被谁占用 | 排查端口冲突 |
| `nc -zv` | 检测端口连通 | `nc -zv host port` |

命令行系列的最后一篇了。回头看看从 `cd` 和 `ls` 开始，一路走到了这里——能搜文件、改文本、写脚本、管进程、调网络接口。命令行的世界比你一开始想的深得多，但你现在已经有底气自己去探索了。

---

> [← 查看系列目录]({% link _pages/series-command-line.md %})
