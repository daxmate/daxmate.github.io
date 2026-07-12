---
layout: single
title:  "从零开始学命令行：sudo——借我一双管理员的手"
date:   2026-07-12 13:00:00 +0800
categories:
  - Command Line
tags:
  - 命令行
  - sudo
  - 权限
  - su
  - root
---

有些事普通用户做不了——装系统级软件、改系统配置、看别人的文件。这时候需要**超级用户权限**。

`sudo` 就是干这个的：**用别人的身份执行命令**（默认是超级用户 root）。

---

## 基本用法

```zsh
sudo 命令
```

例如：

```zsh
sudo apt install nginx          # 装软件
sudo systemctl restart nginx    # 重启系统服务
sudo vim /etc/hosts             # 改系统配置文件
```

敲完之后终端会让你输**你自己的密码**。注意不是 root 密码——`sudo` 验证的是你自己。

输过一次之后，一段时间内（默认五分钟左右）再敲 `sudo` 不用重复输密码。

---

## 什么时候需要 sudo

```zsh
# 不需要 sudo
ls /home
cat ~/notes.txt
ping 8.8.8.8

# 需要 sudo
apt install xxx           # 装系统软件
vim /etc/hosts            # 改系统文件
systemctl restart xxx     # 管理系统服务
shutdown -h now           # 关机
```

简单判断：改到**系统级**的东西就要 `sudo`，改自己的文件不用。

---

## sudo 的常见误区

### 误区 1：sudo echo 追加内容

```zsh
sudo echo "127.0.0.1 myserver" >> /etc/hosts
```

这个会报权限错误。为什么？因为 `sudo` 只提升了 `echo`，`>>` 重定向是 Shell 在当前用户下执行的。

正确做法：

```zsh
echo "127.0.0.1 myserver" | sudo tee -a /etc/hosts
```

`tee -a` 用 sudo 权限来追加写文件。

### 误区 2：不该用 sudo 的场景

```zsh
sudo vim ~/.zshrc
```

你自己的配置文件，不需要 sudo。用 sudo 改了之后，文件可能变成 root 的，`source` 都会报权限问题。

```zsh
sudo ls ~
```

也没意义——看自己的家目录不需要管理员权限。

---

## 切换到 root 用户：`sudo -i` 和 `su`

如果有一串操作都需要管理员权限，每条都加 `sudo` 很烦。可以切换到 root 用户：

```zsh
sudo -i
```

这会给你一个 root 的 Shell，敲啥都是管理员身份。退出用 `exit` 或 `Ctrl+D`。

`su` 也可以切换用户：

```zsh
su -         # 切换到 root（需要 root 密码）
su - 用户名  # 切换到指定用户
```

macOS 上默认没开 root 用户，所以 `su -` 没法直接切。用 `sudo -i` 更稳妥。

---

## 实用技巧

### 忘了加 sudo：`sudo !!`

敲了一个命令发现权限不够，不想重新打一遍：

```zsh
systemctl restart nginx
# Permission denied

sudo !!
```

`!!` 展开成上一条命令，`sudo !!` 就变成 `sudo systemctl restart nginx`。

### 以其他用户身份执行：`sudo -u`

不一定非要是 root。你也可以假装成其他用户：

```zsh
sudo -u www-data whoami    # 输出 www-data
```

服务器上排查问题时经常用到这个特点。

---

## 能执行什么：`/etc/sudoers`

`sudo` 不是所有人都能用。谁能用、能执行哪些命令——配置在 `/etc/sudoers` 里。

```zsh
sudo visudo
```

用 `visudo` 编辑，不要直接改文件。它会检查语法，防止你把自己锁在外面。

常见配置：

```
# 管理员组——Linux 上通常用 wheel，macOS 上通常用 admin
%wheel  ALL=(ALL:ALL) ALL
%admin  ALL=(ALL:ALL) ALL

# 指定某个命令不用输密码
dax     ALL=(ALL) NOPASSWD: /usr/bin/apt
```

---

## 历史小注

`sudo` 是 1980 年在纽约州立大学布法罗分校（SUNY Buffalo）诞生的，由 Bob Coggeshall 和 Cliff Spencer 编写。1994 年 Todd C. Miller 重写了整个代码，后来成为 OpenBSD 的标准组件，再后来被几乎所有 Unix 系操作系统采用。

有趣的是，最初 `sudo` 的意思是 "superuser do"，但也有人开玩笑说它是 "sudo make me a sandwich"。

---

## 动手试试

```zsh
# 看当前用户能不能用 sudo
sudo -v

# 列出当前用户能执行哪些 sudo 命令
sudo -l

# 用 root 身份查看一个有权限的文件
sudo cat /var/log/system.log | tail

# 进入 root shell 看看环境有什么不同
sudo -i
echo $HOME    # /var/root
exit          # 回到普通用户
```

---

## 小结

| 操作 | 作用 |
|------|------|
| `sudo 命令` | 以 root 身份执行 |
| `sudo -i` | 切换到 root Shell |
| `sudo -l` | 查看自己能执行哪些命令 |
| `sudo !!` | 用 sudo 重跑上一条命令 |
| `sudo -u 用户名` | 以指定用户身份执行 |
| `command \| sudo tee -a 文件` | 用 sudo 权限追加内容 |
| `visudo` | 安全编辑 sudoers 配置 |

**核心规则：**
1. 改系统的东西用 sudo，改自己的不用
2. 不要用 sudo 改自己的配置文件
3. `sudo -i` 少用——不小心更容易出事
4. 每条命令单独 `sudo` 比切到 root 安全

---

> [← 查看系列目录]({% link _pages/series-command-line.md %})
