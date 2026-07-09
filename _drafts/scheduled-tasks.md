---
layout: single
title:  "从零开始学命令行：定时任务——让电脑自己干活"
date:   2026-07-09 13:00:00 +0800
categories:
  - Command Line
tags:
  - 命令行
  - cron
  - 定时任务
  - launchd
  - macOS
---

有些事需要定时做——每天备份、每小时同步一次文件、每周清理日志。手工做太傻，应该让机器自己来。

不同系统有不同的定时任务工具：
- **Linux** 用 `cron`
- **macOS** 用 `launchd`（虽然也能装 cron，但 launchd 是亲儿子）

---

## Linux 上的 cron

### 编辑定时任务

```zsh
crontab -e
```

这会打开一个编辑器，每行定义一个任务。格式：

```
分 时 日 月 星期  命令
```

```cron
# 每天凌晨 2 点备份
0 2 * * * /home/dax/scripts/backup.sh

# 每小时运行一次
0 * * * * /home/dax/scripts/sync.sh

# 每周一上午 9 点
0 9 * * 1 /home/dax/scripts/weekly-report.sh

# 重启（@reboot 是特殊语法）
@reboot /home/dax/scripts/start-server.sh
```

五个时间字段的含义：

```
分钟（0-59） 小时（0-23） 日（1-31） 月（1-12） 星期（0-7，0和7都是周日）
```

`*` 表示"每"（每分钟、每小时……）。

### 查看和管理

```zsh
crontab -l        # 查看当前用户的定时任务
crontab -r        # 删除所有定时任务
crontab -e        # 编辑
```

---

## macOS 上的 launchd

macOS 推荐用 `launchd`。每个定时任务是一个 `.plist` 文件，放在 `~/Library/LaunchAgents/` 下。

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

加载：

```zsh
launchctl load ~/Library/LaunchAgents/com.dax.daily-backup.plist
launchctl start com.dax.daily-backup
```

想深入了解 launchd 可以等我们后面专门讲，这里先知道有这个东西就行。

---

## 实用例子

```zsh
# 每天备份
# crontab:
0 3 * * * tar -czf ~/backups/$(date +\%Y\%m\%d).tar.gz ~/Documents

# 每周清理临时文件
0 6 * * 1 find /tmp -type f -atime +7 -delete

# 每 10 分钟检查一次服务状态
*/10 * * * * /home/dax/scripts/health-check.sh
```

注意 `crontab` 里 `%` 需要转义成 `\%`。

---

## 日志和调试

```zsh
# cron 任务的输出会发到邮箱，也可以自己写日志
0 2 * * * /home/dax/scripts/backup.sh >> ~/logs/backup.log 2>&1

# 查看 cron 执行日志（Linux）
grep CRON /var/log/syslog
```

写定时任务一定要加日志重定向，不然出错了都不知道。

---

## 动手试试

```zsh
# 查看当前的 crontab（如果有的话）
crontab -l

# 添加一个测试任务——每分钟记录时间
crontab -e
```

在编辑器里加一行：

```cron
* * * * * echo "$(date): 定时任务在跑" >> ~/cron-test.log
```

等一分钟看结果：

```zsh
cat ~/cron-test.log
```

记得删掉测试任务：

```zsh
crontab -e
# 删掉刚才加的那一行
```

---

## 小结

| | cron（Linux） | launchd（macOS） |
|--|-------|---------|
| 配置格式 | 一行一个任务 | .plist XML 文件 |
| 编辑方式 | `crontab -e` | 手动创建 plist 文件 |
| 管理命令 | `crontab -l` | `launchctl load/unload` |
| 学习成本 | 低 | 高 |

**日常建议：**
- Linux 上直接用 cron
- macOS 上如果只是简单定时任务，可以装 `brew install cron` 然后用 cron
- 复杂的任务再研究 launchd

---

> [← 查看系列目录]({% link _pages/series-command-line.md %})
