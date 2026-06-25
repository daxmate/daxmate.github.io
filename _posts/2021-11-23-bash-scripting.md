---
hidden: true
layout: single
title:  "Bash Scripting 入门"
date:   2021-11-23
categories: "shell"
---

最近对于终端的命令编程有点感兴趣，就找了一本书来学习一下。

## 入门之前

- `bash --version`可以确认一下版本，如果版本太旧，用`brew install bash`安装一个新的版本的

- `help`是bash内置命令的帮助，`man`是系统命令的帮助

- `type`可以查看是内置命令还是系统命令，加上`-a`选项可以列出所有的结果

- `key bindings`可以通过`man bash`然后搜索`readline`部分来查阅，暂时看不懂，以后再说

- 保留字符：`| & ; ( ) < >`

- $开头的变量：

  | 变量 | 含义                                                         |
  | ---- | ------------------------------------------------------------ |
  | $0   | 当前脚本的文件名                                             |
  | $n   | 传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是$1，第二个参数是$2。 |
  | $#   | 传递给脚本或函数的参数个数。                                 |
  | $*   | 传递给脚本或函数的所有参数。                                 |
  | $@   | 传递给脚本或函数的所有参数。被双引号(" ")包含时，与 $* 稍有不同，下面将会讲到。 |
  | $?   | 上个命令的退出状态，或函数的返回值。                         |
  | $$   | 当前Shell进程ID。对于 Shell 脚本，就是这些脚本所在的进程ID。 |

- `\`转义字符
- ''单引号可以包含换行
- ""双引号里面可以用变量
- bash里面的字符串合并没有操作符，直接将两个字符串连在一起就可以了
- `；`可以连接命令，无论命令成功与否
- `$?`获取退出状态，`0`为成功，`1`为失败
- `&&`连接命令，但是如果遇到执行失败就停止
- `&` 放在命令的最后，会让命令在后台运行。如`sleep 10 &`

## 常规命令

- 终端内置命令
  - `type`: Finding what a command is
  - `echo`: Printing arguments
  - `printf`: Printing formatted arguments
  - `pwd`: Printing the current directory
  - `cd`: Changing the current directory
  - `set`: Viewing and setting shell properties
  - `declare`: Managing variables and functions
  - `test, [, [[`: Evaluating expressions

- 系统命令
  - `ls`: Listing files for users
  - `mv`: Moving and renaming files
  - `cp`: Copying files
  - `rm` and `rmdir`: Deleting files and directories
  - `grep`: Matching patterns
  - `cut`: Extracting columns from data
  - `wc`: Counting lines, words, and characters
  - `find`: Iterating through a file tree
  - `sort` and `uniq`: Sorting and de-duplicating input

- 说明
  - `printf`的用法为
    - `printf '%10s\n' -n`，字符的格式化和Python里面的类似，
    - `printf '%10s\n' one two three ` 可以在后面用列表的形式列出多个参数
    - `printf '%q\n' '$1 \tt'` 将字符转义打印

  - `cd` 如果在从脚本里面执行`cd`命令，结束之后还是会回到原来的目录，因为执行脚本运行的是字进程

  - `set`
    - 如果不带任何参数，输出当前终端实例里面的所有的变量和函数
      - TODO：*如何设置prompt*
    - 带参数，后面会提到 # TODO

  - `declare` 可以带几个打印出变量，不是很明白，后面再搞。好像`bash`和`zsh`也不一样。

  - `test` 条件测试：
    - `test -e /etc/passwd && echo 'Password file exists!'`
    - `[ -e /etc/passwd ] && echo 'Password file exists!'`
    - 因为上面的写法不容易理解，后来提供了一种新的写法`[[ ]]`，好像在zsh也可以用，但是用type显示不出来。

  - `ls` 可以用`-a`，`-l`选项。
    - `ls`命令输出的结果是给人阅读的，所以遇到特殊字符的时候提供给其他命令就会出错。这时可以使用`glob`或者`find`命令。

  - `mv`
    - `mv file path/to/directory` 移动文件到文件夹
    - `mv file1 file2`  相当于重命名，为了避免不小心覆盖原来的文件，可以用`-i`选项来提示是否覆盖
    - `mv file1 file2 dir1 path/to/directory` 如果最后一个参数是文件夹，移动前面所有的文件或者文件夹到这里面。

  - `cp`
    - `cp oldfile newfile`
    - `cp file1 file2 file3 dir` 如果最后一个参数是文件夹，复制前面所有的文件到这个文件夹里面。
    - 默认不能复制文件夹，需要`-R`选项
    - `-v`选项会显示复制的明细

  - `rm`和`rmdir`

  - `grep`对文件中的行用正则来进行搜索
    - `grep pattern files`
    - Pattern和Python的re语法差不多，不过需要用双引号括起来，
    - `-E`选项可以强制用扩展语法搜索，就和Python差不多，否则会搜索不到；
    - `-q`选项输出不重复的结果
    - `-c`选项输出计数
    - `-e`可以使用多个pattern，结果只需匹配其中之一即可
    - `-F`搜索转义字符
    - `-v`搜索不匹配的内容

  - `cut`对每行按照一定的规则进行切割

    - `-b` ：以字节为单位进行分割。这些字节位置将忽略多字节字符边界，除非也指定了 -n 标志。
    - `-c` ：以字符为单位进行分割。
    - `-d` ：自定义分隔符，默认为制表符。
    - `-f` ：与-d一起使用，指定显示哪个区域。
    - `-n` ：取消分割多字节字符。仅和 -b 标志一起使用。如果字符的最后一个字节落在由 -b 标志的 List 参数指示的
      范围之内，该字符将被写出；否则，该字符将被排除

  - `wc`单词计数，默认输出行数，单词数，字节数

    - `-l`行数
    - `-w`单词数
    - `-c`字节数
    - `-m`对于不止一个字节编码的字符计数

  - `du` 是`disk usage `的缩写，这个命令暂时不深究

  - `find` 查找文件（夹）

    - 一般用法`find folder pattern options`

    - `-name` 可以使用通配符

    - `-type` 限定文件类型

      ```
               b       block special
               c       character special
               d       directory
               f       regular file
               l       symbolic link
               p       FIFO
               s       socket
      ```

    - `-mtime` 文件修改时间， +3表示三天前，-5表示五天以内

    - `!` 否开关

    - EXAMPLES

      ```bash
      The following examples are shown as given to the shell:
      
           find / \! -name "*.c" -print
                   Print out a list of all the files whose names do not end in .c.
      
           find / -newer ttt -user wnj -print
                   Print out a list of all the files owned by user ``wnj'' that are
                   newer than the file ttt.
      
           find / \! \( -newer ttt -user wnj \) -print
                   Print out a list of all the files which are not both newer than
                   ttt and owned by ``wnj''.
      
           find / \( -newer ttt -or -user wnj \) -print
                   Print out a list of all the files that are either owned by
                   ``wnj'' or that are newer than ttt.
      
           find / -newerct '1 minute ago' -print
                   Print out a list of all the files whose inode change time is more
                   recent than the current time minus one minute.
      
           find / -type f -exec echo {} \;
      
           find -L /usr/ports/packages -type l -exec rm -- {} +
                   Delete all broken symbolic links in /usr/ports/packages.
      
           find /usr/src -name CVS -prune -o -depth +6 -print
                   Find files and directories that are at least seven levels deep in
                   the working directory /usr/src.
      
           find /usr/src -name CVS -prune -o -mindepth 7 -print
                   Is not equivalent to the previous example, since -prune is not
                   evaluated below level seven.
      
      
      ```

    - `sort` 排序
      - `-t`指定分隔符
      - `-k`列
      - `-n`按数字排序
      - `-r`倒序
      - `-c`检查是否已经排序
      - `-u`唯一值
    - `uniq` 唯一
      - `-c`计数

  ## 输入输出和重新定向

  1. 重新定向
     - `>` 将输出写入文件
     - `set -C`可以防止覆盖写入
     - `>|`强制写入
     - `>>`附在后面
  2. 理解文件权限 例：`-rw-r--r--`
     - 第一位`-`是代表类型是文件，如果是文件夹的话显示为`d`
     - 从第二位开始，第三位为一组，依次为owner、group和world(所有人)
     - 一组的三位依次代表读、写、运行
     - 其中读的值为4，写的值为2，运行1，6就代表`rw-`
     - 系统新建文件的时候，默认权限为666，但是在完成新建之前，会检查`umask`的值，来决定文件的最终权限。
     - 例比如umask的值为022，则从group和world各减掉2(写)
     - `umask 027`可以设定当前shell的值，这样从当前shell创建的文件就会直接受到影响
  3. 重新定向错误
     * 例`grep pattern myfile /nonexistent > matches 2 > errors` ，这样错误就写入errors里面了。
     * `grep pattern myfile /nonexistent > matches 2>&1`这样可以将结果和错误都收集到matches里面
     * `grep pattern myfile /nonexistent > matches 2> /dev/null` 这个会完全忽略错误，`/dev/null`文件永远是空白的。
  4. 重新定向到多个文件
     - `printf 'Copy this to output\n' | tee myfile1 myfile2 myfile3`
     - 以上主要是解决运行时复制多份的问题，一般的情况下可以运行完成后再复制到各个地方。
     - `tr a-z A-Z < myfile > myfile_cap 2> myfile.error` 翻译命令也可以从文件中读取信息然后翻译
  5. 使用`cat <<'EOF'`命令可以将下行(在脚本中)到`EOF`标记之间的所有行打印出来。**如果去掉单引号的话，可以在字符串里面使用变量**。如果用`<<-`的话，输入的前导tab就会被忽略。
  6. pipeline `|`可以将命令的结果作为后面一个命令的输入
  7. `cat file1 file2 file3 > myfile.combinded` 可以将多个文件合并到一个。
  8. `echo 'hello world' | cat file1 - file2` 其中的`-`会把前面一个命令里面的输出包括进来。或者也可以明确的将`-`写成`/dev/stdin`
  9. `sed`命令 过滤行
     - sed command file
     - 其中command和vim中差不多，比如`1d`为删除第一行；'s/a/b/'为替换
  10. `awk`命令 过滤表格
     - `awk '{ print $2}' table` 会打印出第二列
     - `awk '{ print $2, $3}' table`会打印多列
     - `awk ' NR > 1 { print $2, $3}' table` 其中NR是指数字的意思；

## 变量和模式匹配

### 变量

1. 变量申明 `name='dax'`,注意不能有空格
2. `declare -p`可以打印出所有的变量，如果加上变量的名称，可以得到详细的变量的信息；
3. 变量名称只能包含字母数字和下划线；
4. 一般情况，全大写变量是环境变量，不要轻易修改；
5. `name=` 将变量设置为空，或者用 `unset -v name` 直接处理掉变量；
6. `[[ -n $name ]]通过测试变量值的长度来确定变量是否为空；`   

