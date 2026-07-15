---
layout: single
title:  "从零开始学命令行：网络工具"
date:   2026-07-14 09:10:00 +0800
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

命令行的世界不只是管管自己电脑上的文件。有时候要从网上下载东西、发 HTTP 请求、连远程服务器、查端口被谁占了——这篇把最常用的几个网络命令捋一遍。

这些工具的诞生背景都挺有意思：有人写 IRC 机器人顺便搞出了 curl，有人用声呐的原理写出了 ping，有人因为密码被偷了才写了 SSH。边学怎么用，边听听它们是怎么来的。

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

输出类似：

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

`-X` 指定 HTTP 方法，`-d` 是发送的数据。发 JSON 的话加个请求头：

```zsh
curl -X POST -H "Content-Type: application/json" \
  -d '{"name":"大象","score":100}' \
  https://example.com/api
```

### 跟随重定向和静默模式

```zsh
curl -L https://bit.ly/xxxx      # 自动跟随重定向
curl -s https://example.com       # 静默模式，不显示进度条
```

`-s` 在脚本里基本必加，没人想看进度条。

### 小故事

curl 的故事从 1996 年开始。瑞典程序员 **Daniel Stenberg** 在写一个 IRC 机器人，想加个汇率查询功能——从网站自动下载汇率数据。他找到了一个叫 HttpGet 的开源工具，开始给作者提补丁。作者不堪其扰，直接把项目扔给了他。

Stenberg 接手后不断改进。一开始只支持 HTTP 和 Gopher（还记得 Gopher 吗？），后来加入了 FTP 支持，改名叫 **cURL**——"Client for URL" 的缩写，读起来也像 "See URL"。从最初约 100 行 C 代码，到今天超过 18 万行，curl 是世界上安装量最大的软件之一。现在几乎每台电脑、每部手机、每个智能设备里都有它。

---

## 测试网络连通：`ping`

```zsh
ping google.com
```

输出类似：

```
PING google.com (142.250.80.46): 56 data bytes
64 bytes from 142.250.80.46: icmp_seq=0 ttl=117 time=43.123 ms
64 bytes from 142.250.80.46: icmp_seq=1 ttl=117 time=42.567 ms
```

`ping` 向目标服务器发 ICMP 包，目标回复了就算通。`Ctrl + C` 停止，能看到统计汇总。

只发固定次数就用 `-c`：

```zsh
ping -c 4 google.com
```

网络不通时，先用 `ping` 看是「完全连不上」还是「慢」，这是排网络故障的第一步。

### 小故事

1983 年，**Mike Muuss** 在美军弹道研究实验室（BRL）工作。当时 ARPANET（互联网的前身）经常出问题——数据包丢了、延迟高了，但没有任何工具能快速诊断。

Muuss 对二战潜艇战很感兴趣，想到了声呐的原理：潜艇发出一个 "ping" 声脉冲，听回声的时间来测距。于是他用约 1000 行 C 代码写了一个工具，发送 ICMP 包等待回复，测量往返时间——这就是 ping。

他本想叫它 "poke"（戳一下），后来还是选了 "ping"——更形象。顺便说一句，有人后来把 "ping" 解释成 "Packet Internet Groper" 这个缩写，但 **Muuss 本人明确说过这不对**。ping 就是 ping，像声呐一样。

Muuss 把代码作为公有领域发布，没有版权限制。这大概是人类历史上被下载最多的 1000 行代码之一。可惜的是，他在 2000 年因车祸去世，年仅 42 岁。

---

## 远程登录：`ssh`

```zsh
ssh 用户名@主机地址
```

比如：

```zsh
ssh dax@192.168.1.100
```

第一次连接会问要不要信任这台机器的指纹——输入 `yes` 确认。然后输入密码就进去了。敲 `exit` 退出。

### 免密登录：SSH Key

每次都输密码很烦。生成一对密钥，把公钥放到远程机器上，以后就不用输密码了：

```zsh
# 生成本机密钥对
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

### 小故事

1995 年，赫尔辛基理工大学的研究员 **Tatu Ylönen** 发现学校的网络被装了密码嗅探器——Telnet、FTP、rlogin 的密码都是明文传输，抓到就能看。

这个安全事件逼他写了一个新协议：**SSH**（Secure Shell）。他在 1995 年 7 月发布了 SSH-1，年底就有了 2 万用户，遍布 50 个国家。

端口号选 22 也是有意为之——它正好夹在 FTP（21）和 Telnet（23）之间，象征 SSH 是这两者的安全继任者。Ylönen 给 IANA（互联网数字分配机构）发了封邮件申请，当时管这个的是互联网先驱 Jon Postel 和 Joyce Reynolds，第二天就回了：「已分配端口 22 给 ssh，联系人为你。」就这么简单。

那个年代发一封邮件就能要到端口号。换成今天，走标准流程得折腾好几年。

后来 OpenBSD 团队在 1999 年做了 **OpenSSH**，开源的、自由的实现，成了 Unix 和 macOS 的标配。今天 SSH 已经是互联网基础设施的一部分——没有它，我们可能还在用 Telnet 看明文密码。

### 配置别名：`~/.ssh/config`

每次都敲 `ssh dax@192.168.1.100 -p 2222` 很烦。在 `~/.ssh/config` 里配好：

```
Host myserver
    HostName 192.168.1.100
    User dax
    Port 2222
```

然后直接：

```zsh
ssh myserver
```

登录名、端口、密钥文件、跳板机——所有连接信息写一次就好。

### 传文件：`scp`

SSH 不仅能远程登录，还能安全地传文件：

```zsh
# 本地传到服务器
scp file.txt myserver:~/backup/

# 服务器拉到本地
scp myserver:~/backup/file.txt ./

# 传整个目录
scp -r myfolder myserver:~/
```

`scp` 的用法跟 `cp` 很像，只是路径可以带 `主机名:` 前缀。

### 安全备忘

- **私钥不要外传**——谁拿到 `id_ed25519` 就能登录所有配了公钥的机器
- **给私钥加密码**——`ssh-keygen` 时输入的 passphrase，私钥丢了别人也打不开
- **不用 root 直接 SSH**——用普通用户登录 + `sudo`
- **关掉密码登录**——配好密钥后，在服务器上设 `PasswordAuthentication no`

---

## 查看网络信息：`ifconfig`

```zsh
ifconfig
```

显示所有网络接口的信息——IP 地址、MAC 地址、收发数据量。

只看某个接口（比如 macOS 上的 Wi-Fi）：

```zsh
ifconfig en0
```

`en0` 通常是 Wi-Fi（macOS 上），`en1` 或是有线网。搞不清时 `ifconfig` 不带参数全看一遍。

只提取 IP 地址：

```zsh
ifconfig en0 | grep "inet " | awk '{print $2}'
```

### 注一嘴

`ifconfig` 最早出现在 1983 年的 **4.2BSD** 里，是伴随 BSD TCP/IP 协议栈一起被引入的。但在 Linux 世界中，它已经被 **`ip` 命令**（iproute2 套件）取代了。`ifconfig` 用来快速看一下没问题，但写脚本的话，新系统上建议用 `ip addr`、`ip link` 这些。

不过 macOS 上 `ifconfig` 还在正常维护，用就是了。

---

## 检查端口：`lsof`

想知道某个端口被谁占用了：

```zsh
lsof -i :8080
```

输出类似：

```
COMMAND   PID USER   FD   TYPE    DEVICE SIZE/OFF NODE NAME
python  12345  dax    3u  IPv4  0x...      0t0  TCP *:8080 (LISTEN)
```

PID 是 12345，是 Python 在占用 8080。

只看监听（LISTEN）状态：

```zsh
lsof -i :8080 | grep LISTEN
```

### 小故事

`lsof` 是普渡大学的 **Victor Abell** 从 1991 年开始开发维护的——一个人做了近三十年，一直到 2019 年才把维护权交给 GitHub 社区。"lsof" 就是 "LiSt Open Files" 的缩写，不只是看端口——它还能看进程打开了哪些普通文件、目录、管道、库文件，甚至是被删除但还被进程占着的文件。

（Linux 上 `lsof` 也通用。不过如果你装的是新 Linux，也可以用 `ss -tlnp` 看端口监听情况——那是现代方式。）

---

## 网络连通性诊断：`nc`（netcat）

`nc` 是个简单的网络测试工具。检查某台机器的某个端口能不能连上：

```zsh
nc -zv example.com 80
```

输出类似：

```
Connection to example.com port 80 [tcp/http] succeeded!
```

`-z` 是只扫描不发送数据，`-v` 是详细输出。端口开没开，一秒就知道。

### 小故事

1995 年，一个网名叫 **Hobbit** 的安全专家（取自托尔金的《霍比特人》）把 Unix 上 `cat` 的理念搬到了网络世界——`cat` 读写文件流，`nc` 读写网络流。所以它叫 "netcat"，名字说明了一切。

后来 `nc` 分出了好几个分支（GNU 版、OpenBSD 版、Nmap 项目的 Ncat），选项不完全一样。macOS 用的是 BSD 版。不管哪个版本，基本的 `-zv` 端口检测功能都一样。

---

## 查域名：`dig`

`ping` 通了，浏览器还是打不开网站？可能是 DNS 出了问题——域名转不成 IP。

`dig` 是查 DNS 记录的工具：

```zsh
dig example.com
```

输出会包含域名对应的 IP、查询了哪个 DNS 服务器、花了多长时间。只看 IP 的话：

```zsh
dig example.com +short
```

```
93.184.216.34
```

指定 DNS 服务器查询：

```zsh
dig @8.8.8.8 example.com
```

`@8.8.8.8` 是用 Google 的公共 DNS 查——有时候本地 DNS 缓存了错误的结果，换个服务器能看出问题出在哪一步。

### 小故事

`dig` 是 BIND（Berkeley Internet Name Domain）工具集的一部分。BIND 是 80 年代加州大学伯克利分校开发的 DNS 服务器软件。现在互联网上绝大多数的 DNS 服务器跑的都是 BIND 或其衍生版本，`dig` 就这么跟着成了跑不掉的标配工具。

——"Domain Information Groper"，意思是翻 DNS 信息的。能用 `dig` 查域名，也能用它做更深入的 DNS 调试——指定记录类型（A、MX、TXT 等）、追递归查询链路——不过这篇里知道最基本的用法就够了。

---

## 动手试试

```zsh
# 下载网页前 10 行
curl -s https://example.com | head -n 10

# 看响应头
curl -I https://example.com

# ping 一下
ping -c 3 baidu.com

# 查一个域名的 IP
dig example.com +short

# 看你自己的 IP
ifconfig en0 | grep "inet "

# 看看 SSH 密钥有没有
ls -la ~/.ssh/

# 检查本地 22 端口在不在监听
lsof -i :22
```

## 小结

| 命令 | 作用 | 常用搭配 |
|------|------|----------|
| `curl` | 下载文件、发 HTTP 请求 | `-o` 保存，`-I` 响应头，`-L` 跟随重定向，`-s` 静默 |
| `ping` | 测网络连通性 | `-c 次数` 限制发包 |
| `ssh` | 远程登录 | 配 SSH Key 免密、`~/.ssh/config` 别名 |
| `ssh-copy-id` | 把公钥放远程机器 | 配一次，以后免密登录 |
| `scp` | 通过 SSH 传文件 | 路径加 `主机名:` 前缀，`-r` 传目录 |
| `ifconfig` | 查看网络接口 | Linux 上考虑用 `ip addr` |
| `dig` | DNS 查询 | `+short` 只看 IP，`@服务器` 指定 DNS |
| `lsof -i :端口` | 看端口被谁占用 | `grep LISTEN` 只看监听状态 |
| `nc -zv` | 检测端口连通 | `nc -zv host port` |

从 `curl` 到 `ping`，再到 `ssh` 和 `nc`，这些工具背后都有各自的故事——有人为了解决一个具体问题写了它们，结果成了全世界每天都在用的基础设施。这就是命令行工具的魅力：一个简单的灵感，最后改变了整个行业。

> [← 查看系列目录]({% link _pages/series-command-line.md %})
