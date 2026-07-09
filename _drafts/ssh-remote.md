---
layout: single
title:  "从零开始学命令行：远程连接——一台电脑控制另一台"
date:   2026-07-09 13:00:00 +0800
categories:
  - Command Line
tags:
  - 命令行
  - SSH
  - 远程
  - scp
---

想连上服务器？想在另一台电脑上敲命令？**SSH（Secure Shell）** 是干这个的标准工具。

它让你在本地终端里操作远程电脑，就跟坐在那台电脑前一样。

---

## 基本连接

```zsh
ssh 用户名@服务器地址
```

```zsh
ssh dax@192.168.1.100
```

第一次连某个服务器会看到：

```
The authenticity of host '192.168.1.100 (192.168.1.100)' can't be established.
Are you sure you want to continue connecting (yes/no)?
```

这是 SSH 在确认对方是不是真的那台机器。确认一次之后就会记住，下次不会再问。

输入密码就连上了。敲 `exit` 或 `Ctrl+D` 断开。

---

## 免密码登录：SSH Key

每次输密码很烦。用 SSH Key 可以免密码登录。

### 1. 生成密钥对

```zsh
ssh-keygen -t ed25519
```

一路回车。会生成两个文件：
- `~/.ssh/id_ed25519` — **私钥**，自己留着，绝不外传
- `~/.ssh/id_ed25519.pub` — **公钥**，放到服务器上

### 2. 把公钥放到服务器

```zsh
ssh-copy-id dax@192.168.1.100
```

输一次密码，之后就不用再输了。`ssh-copy-id` 会把你的公钥追加到服务器的 `~/.ssh/authorized_keys` 里。

---

## 别名登录：写进 `~/.ssh/config`

每次敲 `ssh dax@192.168.1.100 -p 2222` 太长。在 `~/.ssh/config` 里配好：

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

还能配多个主机、指定密钥、自定义端口——所有连接信息写一次就好。

---

## 传文件：scp

```zsh
# 本地传到服务器
scp file.txt myserver:~/backup/

# 服务器拉到本地
scp myserver:~/backup/file.txt ./

# 传目录
scp -r myfolder myserver:~/
```

`scp` 的用法跟 `cp` 很像，只是路径可以带 `主机名:` 前缀。

---

## 常用的快捷操作

```zsh
# 在远程执行一个命令
ssh myserver "df -h | head -5"

# 保持连接的 SSH
ssh -o ServerAliveInterval=60 myserver

# 退出远程
exit
```

---

## 安全注意事项

- **私钥不要传给任何人**，谁拿到它就能登录你的服务器
- 给密钥加上密码（`ssh-keygen` 时输入的 passphrase）——私钥丢了别人也打不开
- 不要用 root 直接 SSH 登录，用一个普通用户 + `sudo`
- 不需要密码登录之后，可以关掉服务器的密码登录（`PasswordAuthentication no`）

---

## 动手试试

```zsh
# 检查有没有 SSH key
ls -la ~/.ssh/

# 生成密钥对（如果没有的话）
ssh-keygen -t ed25519

# 查看公钥
cat ~/.ssh/id_ed25519.pub

# 配置 config 文件
echo "Host test-server
    HostName 192.168.1.100
    User dax" >> ~/.ssh/config

# 连接
ssh test-server
```

---

## 小结

| 操作 | 命令 |
|------|------|
| 连接远程 | `ssh 用户@地址` |
| 生成密钥 | `ssh-keygen -t ed25519` |
| 上传公钥 | `ssh-copy-id 用户@地址` |
| 传文件 | `scp 文件 用户@地址:路径` |
| 配置别名 | `~/.ssh/config` |

用 SSH Key + `~/.ssh/config` 之后，日常就是敲 `ssh myserver` 连上去，跟开本地终端一样快。

---

> [← 查看系列目录]({% link _pages/series-command-line.md %})
