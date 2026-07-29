---
layout: single
title:  "从零开始学命令行：定时任务——让电脑自己干活"
date:   2026-07-15 17:05:00 +0800
categories:
  - Command Line
tags:
  - 命令行
  - cron
  - 定时任务
  - launchd
  - macOS
---

有些事需要定时做——每天凌晨备份一次、每小时同步下文件、每周清理临时文件。手工做太傻，应该让机器自己来。

这里就引出一个问题：谁在管"什么时候该做什么"？答案取决于系统。

---

## Linux 上的 cron

cron 是 Linux 世界做定时任务用的。名字来自希腊语"chronos"（时间），是 Ken Thompson 在 1975 年的贝尔实验室写的——没错，就是写 Unix 和 B 语言的 Ken Thompson。

cron 的配置简单直白：一行定义一个任务。

顺带一提——Ken Thompson 后来说 cron 名字其实是个拼写失误，本来想写"chrono-"（跟时间有关的词根），少打了个 h 就变成了 cron。一个打错的字成了全世界最常用的定时任务工具。

### 编辑定时任务

```zsh
crontab -e
```

进入编辑器后，每一行是一种安排：

```text
# 每天凌晨 2 点备份
0 2 * * * /home/dax/scripts/backup.sh

# 每小时运行一次
0 * * * * /home/dax/scripts/sync.sh

# 每周一上午 9 点
0 9 * * 1 /home/dax/scripts/weekly-report.sh

# 重启时运行（@reboot 是特殊语法）
@reboot /home/dax/scripts/start-server.sh
```

五个时间字段的规则：

```text
分钟（0-59） 小时（0-23） 日（1-31） 月（1-12） 星期（0-7，0 和 7 都是周日）
```

`*` 表示"每"——`* * * * *` 就是每分钟跑一次。`*/10` 是"每 10"。

### 查看和管理

```zsh
crontab -l        # 查看当前定时任务
crontab -r        # 删除全部定时任务
crontab -e        # 编辑
```

### 日志和调试

cron 任务的输出默认会发邮件。更实际的做法是自己写日志：

```zsh
0 2 * * * /home/dax/scripts/backup.sh >> ~/logs/backup.log 2>&1
```

```zsh
# 查看 cron 执行日志（Linux）
grep CRON /var/log/syslog
```

写定时任务一定要加日志重定向——不然出错了都不知道。

---

## macOS 上的 launchd

macOS 不是 Linux。它自带的定时任务系统叫 `launchd`，是苹果 Mac OS X 10.4 Tiger（2005 年）引入的。作者是 **Dave Zarzycki**，当时苹果想让一个系统服务同时管进程启动、定时任务、守护进程——把 cron 和 init 的事情合并了。

思路很好，但对于只想"每天 2 点跑个备份"的普通用户来说，有点重。

每个定时任务是一个 `.plist` 文件，放在 `~/Library/LaunchAgents/` 下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.dax.daily-backup</string>
    <key>ProgramArguments</key>
    <array>
        <string>/Users/dax/scripts/backup.sh</string>
    </array>
    <key>StartCalendarInterval</key>
    <dict>
        <key>Hour</key>
        <integer>2</integer>
        <key>Minute</key>
        <integer>0</integer>
    </dict>
</dict>
</plist>
```

### 更多 launchd 配置

除了定时运行，`launchd` 还能做很多事。以下是一些常用的配置键：

**登录时启动**

```xml
<key>RunAtLoad</key>
<true/>
```

配上这个，登录后自动运行——相当于 macOS 的「登录项」。

**崩溃后自动重启**

```xml
<key>KeepAlive</key>
<true/>
```

适合想一直跑着的服务（如代理、同步工具）。脚本退出后 `launchd` 会自动再拉起来。

**每隔 N 秒运行**

不想用 `StartCalendarInterval`（指定几点几分），可以用 `StartInterval` 指定间隔：

```xml
<key>StartInterval</key>
<integer>3600</integer>
```

3600 就是每小时跑一次，不用算分钟和小时。

**文件变化时触发**

```xml
<key>WatchPaths</key>
<array>
    <string>/Users/dax/Documents</string>
</array>
```

监听的文件或目录变了就自动运行。适合「这个文件夹里一有动静就执行某操作」的场景。

### 别忘了环境变量

`launchd` 跑任务时不会继承你的 shell 环境变量（包括 `PATH`），如果脚本里用了 `brew` 安装的命令，需要在 plist 里加上：

```xml
<key>EnvironmentVariables</key>
<dict>
    <key>PATH</key>
    <string>/usr/local/bin:/usr/bin:/bin</string>
</dict>
```

### 管理命令

```zsh
# 加载（macOS 新版推荐 bootstrap）
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.dax.daily-backup.plist

# 查看在不在跑
launchctl list | grep com.dax

# 手动触发一次
launchctl start com.dax.daily-backup

# 停掉并卸载
launchctl bootout gui/$(id -u)/com.dax.daily-backup
```

---

## 实用例子

```zsh
# 每天凌晨 3 点打包备份 Documents
# cron 写法：
0 3 * * * tar -czf ~/backups/$(date +\%Y\%m\%d).tar.gz ~/Documents

# 每周一早上 6 点清理临时文件
0 6 * * 1 find /tmp -type f -atime +7 -delete

# 每 10 分钟检查一次服务健康
*/10 * * * * /home/dax/scripts/health-check.sh
```

注意 `crontab` 里 `%` 需要转义成 `\%`。

---

## 动手试试

```zsh
# 看看有没有 crontab
crontab -l

# 如果有一台 Linux 机器——加上测试任务试试
crontab -e
```

加一行：

```text
* * * * * echo "$(date): 定时任务在跑" >> ~/cron-test.log
```

等一分钟看结果：

```zsh
cat ~/cron-test.log
```

测试完记得删掉：

```zsh
crontab -e
# 删掉刚才加的那一行
```

---

## 小结

| | cron（Linux） | launchd（macOS） |
|--|-------|---------|
| 配置格式 | 一行一个任务 | .plist XML 文件 |
| 定时运行 | `0 2 * * *` 五字段 | `StartCalendarInterval` 或 `StartInterval` |
| 登录启动 | 无 | `RunAtLoad` |
| 崩溃重启 | 无 | `KeepAlive` |
| 文件监听 | 无 | `WatchPaths` |
| 编辑方式 | `crontab -e` | 手动创建 plist |
| 管理命令 | `crontab -l` | `launchctl bootstrap/list/bootout` |
| 学习成本 | 低 | 高 |
| 诞生 | 1975, Ken Thompson 贝尔实验室 | 2005, Apple, 代替 cron+init |

两个思路不一样：cron 简单直接，一行搞定；launchd 功能强大但门槛高。在 Linux 上用 cron 就够了，在 macOS 上如果只是跑个定时脚本，也可以装 `brew install cron` 来用 cron。

---

> [← 查看系列目录]({% link _pages/series-command-line.md %})
