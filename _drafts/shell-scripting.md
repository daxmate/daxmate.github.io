---
layout: single
title:  "从零开始学命令行：写一个简单的脚本"
date:   2026-07-01 14:00:00 +0800
categories:
  - Command Line
tags:
  - 命令行
  - 脚本
  - Shell Script
  - Bash
---

学到这里，你已经会在终端里敲命令了。但有些事情需要反复做——比如每天早上 `cd` 到项目目录，`git pull`，然后开几个常用的文件。每次都重复敲一遍太傻了。

脚本就是把这个过程写成一个文件，一个命令全搞定。

---

## 第一个脚本

创建一个新文件 `hello.sh`：

```zsh
#!/bin/zsh
echo "你好，命令行！"
echo "今天是 $(date +%Y-%m-%d)"
```

给它执行权限：

```zsh
chmod +x hello.sh
```

运行：

```zsh
./hello.sh
```

```
你好，命令行！
今天是 2026-07-10
```

三件事：

1. `#!/bin/zsh` 告诉系统用哪个程序来执行这个脚本（这叫 **shebang**，`#!` 的合称）
2. `chmod +x` 让这个文件变成可执行文件
3. `./hello.sh` 执行（`./` 表示当前目录，不能省——省了系统会去 `PATH` 里找，找不到就报错）

---

## 为什么用 `#!/bin/zsh` 而不是 `#!/bin/bash`

我们在用 zsh，所以写 `#!/bin/zsh`。如果想兼容更多的机器（有些服务器只有 bash），就用 `#!/bin/bash`。

一开始不用纠结这个。你在 Mac 上用哪个 Shell 就写哪个。

---

## 变量

脚本里也能设变量，跟环境变量一样：

```zsh
#!/bin/zsh
name="大象"
echo "你好，$name！"
```

注意：**等号两边不能有空格**。`name = "大象"` 是错的，Shell 会把你当成在跑一个叫 `name` 的命令。

用 `${}` 包裹变量名可以避免歧义：

```zsh
echo "${name}同学"
```

`$name同学` 里的 `$name同学` 会被当成一个变量名。`${name}同学` 明确告诉 Shell 变量名是 `name`，后面是普通文字。

---

## 接受参数

运行脚本时可以传参数进去：

```zsh
./greet.sh 大象 北京
```

脚本里用 `$1`, `$2`, `$3`... 接收：

```zsh
#!/bin/zsh
echo "你好，$1！"
echo "你在 $2。"
```

| 变量 | 含义 |
|------|------|
| `$0` | 脚本自己的名字 |
| `$1` | 第一个参数 |
| `$2` | 第二个参数 |
| `$@` | 所有参数 |
| `$#` | 参数个数 |

---

## 条件判断：`if`

```zsh
#!/bin/zsh

if [ "$1" = "hello" ]; then
    echo "你也 hello！"
elif [ "$1" = "bye" ]; then
    echo "再见！"
else
    echo "你说的是：$1"
fi
```

几个注意点：

- `[` 和 `]` 前后都要有空格（`[ "$1"` 不是 `["$1"`）
- 字符串比较用 `=`，数字比较用 `-eq`, `-lt`, `-gt`
- 条件语句以 `fi` 结尾（`if` 反过来）
- `then` 可以写在 `if` 同一行（加分号），也可以另起一行

### 文件检查

```zsh
if [ -f "$1" ]; then
    echo "$1 是一个普通文件"
fi

if [ -d "$1" ]; then
    echo "$1 是一个目录"
fi

if [ ! -e "$1" ]; then
    echo "$1 不存在"
fi
```

| 判断 | 含义 |
|------|------|
| `-f` | 是普通文件 |
| `-d` | 是目录 |
| `-e` | 存在（不管类型） |
| `-r` | 可读 |
| `-w` | 可写 |
| `-x` | 可执行 |

---

## 循环：`for`

```zsh
#!/bin/zsh

for file in *.txt; do
    echo "处理：$file"
    wc -l "$file"
done
```

对当前目录下每个 `.txt` 文件，统计行数。

---

## 实战：一个实用的脚本

写一个「备份项目」的脚本，把当前目录打包，加上时间戳存到桌面：

```zsh
#!/bin/zsh

# 备份脚本
# 用法：./backup.sh 项目名

if [ -z "$1" ]; then
    echo "用法：$0 <项目名>"
    exit 1
fi

timestamp=$(date +%Y%m%d_%H%M%S)
backup_name="${1}_${timestamp}.tar.gz"

echo "正在备份 $1 ..."
tar -czf ~/Desktop/"$backup_name" "$1"

if [ $? -eq 0 ]; then
    echo "备份完成：~/Desktop/$backup_name"
else
    echo "备份失败"
    exit 1
fi
```

几个新东西：

- `$?` 是上一条命令的**退出码**。0 表示成功，非 0 表示出错了。
- `exit 1` 让脚本以错误码退出——调用者能判断这个脚本是成了还是挂了。
- `-z` 判断字符串是否为空

---

## 脚本放哪

每次都要 `./script.sh` 太麻烦。把脚本放到 `PATH` 里的某个目录，就能在任何地方直接敲脚本名运行。

建一个自己的 bin 目录：

```zsh
mkdir -p ~/bin
```

把脚本扔进去：

```zsh
cp backup.sh ~/bin/backup
chmod +x ~/bin/backup
```

确保 `~/bin` 在 `PATH` 里（检查 `~/.zshrc` 有没有这一行，没有就加上）：

```zsh
export PATH="$HOME/bin:$PATH"
```

然后 `source ~/.zshrc`，之后在任何目录都能直接敲 `backup`。

---

## 动手试试

```zsh
cd ~/cmd_practice
mkdir scripts && cd scripts

# 第一个脚本
cat > hello.sh << 'EOF'
#!/bin/zsh
echo "你好，$USER！"
echo "当前目录：$PWD"
echo "今天是 $(date)"
EOF

chmod +x hello.sh
./hello.sh

# 带参数的
cat > greet.sh << 'EOF'
#!/bin/zsh
if [ -z "$1" ]; then
    echo "用法：$0 <名字>"
    exit 1
fi
echo "你好，$1！"
EOF

chmod +x greet.sh
./greet.sh 大象
./greet.sh    # 测试没有参数的情况

# 循环
cat > count.sh << 'EOF'
#!/bin/zsh
for f in "$@"; do
    if [ -f "$f" ]; then
        lines=$(wc -l < "$f")
        echo "$f: $lines 行"
    else
        echo "$f: 文件不存在"
    fi
done
EOF

chmod +x count.sh
echo -e "line1\nline2\nline3" > test.txt
./count.sh test.txt hello.sh
```

---

## 小结

| 要素 | 写法 |
|------|------|
| Shebang | `#!/bin/zsh` |
| 变量 | `name="值"`，用 `$name` 或 `${name}` |
| 参数 | `$1`, `$2`, `$@`, `$#` |
| 条件判断 | `if [ 条件 ]; then ... fi` |
| 循环 | `for var in list; do ... done` |
| 退出码 | `exit 0`（成功）`exit 1`（失败） |
| 上一条的退出码 | `$?` |

脚本不需要从第一行开始就把所有语法背下来。**先写出能跑的，然后慢慢加功能**。需要判断加 `if`，需要重复加 `for`，自然而然就熟了。

下一篇进入一个新领域——进程管理。怎么查看正在跑的程序、怎么关掉卡死的程序、后台运行是怎么办到的。

---

> [← 查看系列目录]({% link _pages/series-command-line.md %})
