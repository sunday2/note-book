* 终端(terminal)和shell区别

```
    很多人没有理解终端(terminal)和shell的概念，区分两个概念有助于更好的使用shell。
    终端是交互式工具，通过它与shell环境进行交互。所以终端也是一个程序，但是是一个图形化的工具程序，shell也是一个程序，它是命令的一个解释器。
    mac有原生的终端，也有iTerm2等。而shell除了默认也是一般脚本都会指定的shell是bash shell,它是 Bourne shell 的一个免费版本。所以平时打开的是终端,你在终端上执行命令时,实际解释命令的程序是shell,默认是bash shell,而且默认是标准输出。
<<<<<<< HEAD
```

=======
    Shell is an environment in which we can run our commands, programs, and shell scripts.
```



* 关于man命令

```
当遇到不熟悉的命令或者某个命令不熟悉的选项时, 可以通过man命令查看具体命令的用法.

man command in Linux is used to display the user manual of any command that we can run on
the terminal.

The manual page, associated with the arguments, is then found and displayed.

Each manual page is divided into sections: NAME, SYNOPSIS, CONFIGURATION, DESCRIPTION, OPTIONS, EXIT STATUS, RETURN VALUE, ERRORS, FILES, VERSIONS, NOTES, BUGS, EXAMPLE, AUTHORS, and SEE ALSO.

这里说所有在终端可以执行的命令都可以通过man查看其使用方法, 其实不一定, 得看能不能找到对应的manual page, 每个命令的man page包括几个部分, 但是也不一定包括所有部分, 不同命令的manual page有差异.
```



* shell的种类

```
Each flavor of shell has its own set of recognized commands and functions.

两大类:
1.Bourne shell
默认的shell提示符是$, 超级用户是#.
细分为：
(1)Bourne shell (sh)   第一个shell, 大多数unix系统都装有该shell, 一般位于/bin/sh
(2)Korn shell (ksh)
(3)Bourne Again shell (bash)  
(4)POSIX shell (sh)

2.C shell
默认的shell提示符是%.
细分为：
(1)C shell (csh)
(2)TENEX/TOPS C shell (tcsh)


Bourne shell was the first shell to appear on Unix systems, thus it is referred to as "the shell".

Bourne shell is usually installed as /bin/sh on most versions of Unix. For this reason, it is the shell of choice for writing scripts that can be used on different versions of Unix.
```



>>>>>>> b06b65982dafb31d2e5c74da2130eb80edb93db1
* shell脚本的执行

网上总结挺参差不齐的，其实只有两种方式执行.
(1)将脚本作为命令行参数运行

``` 
bash script.sh
```

(2)赋予脚本可执行权限，此种方式脚本里需指定执行的shell程序,一般为第一行

```
#!/bin/sh
```

```\chmod a+x script.sh
chmod a+x script.sh
./script.sh
```

<<<<<<< HEAD
* 命令的分割

```
通过换行或者分号区分
=======
* 命令的定界符

```
通过换行或者分号区分两条命令
>>>>>>> b06b65982dafb31d2e5c74da2130eb80edb93db1
```

* 注释符

```
字符#指明注释的开始，一直延续到行尾.
```

* bash中的变量

```
<<<<<<< HEAD
bash中每一个变量都是字符串，所以不需要指定类型
```

```
变量分为环境变量和本地变量
=======
bash中每一个变量都会被以字符串的形式存储，所以不需要指定类型
```

```
变量分为环境变量(build in)和本地变量
>>>>>>> b06b65982dafb31d2e5c74da2130eb80edb93db1
```

* 查看终端相关的所有环境变量

```
env
```

* 获取变量的值

```
变量名加$前缀，如果变量加上单引号，变量不会被扩展;如果加上双引号，那么那么若变量存在，则得到变量的值,否则什么都没有
```

* 设置环境变量

```
环境变量是未在当前进程定义，而从父进程中继承而来的变量。其中有一种环境变量叫作标准环境变量。比如常见的$PATH。可以通过export命令设置环境变量。常见的标准环境变量还有$HOME,$PWD,$USER,$UID,$SHELL
环境变量一般保存的是路径列表，一般是可执行文件或者库文件的路径。
```

```
export PATH=$PATH:"／home／user／bin"
```

* 文件描述符

```
文件描述符是与文件输入,文件输出相关联的整数。用来跟踪已打开的文件。
0，1，2是系统预留的。
0表示标准输入，
1表示标准输出,也是默认的选项。平时我们通过终端敲命令时，默认的输出就是标准输出，即控制台
2表示标准错误，其实也是输出项
```

* 纪元时

```
纪元时被定义为从世界标准时间1970年1月1日0时0分0秒起至当前时刻的总秒数,不包括闰秒。对于计算两个日期或两段时间的差值，十分有用。
```

* 调试脚本

```
调试功能是每一种编程语言都应该实现的重要特性之一，调试信息可以帮助弄清楚是什么原因使得程序发生崩溃或行为异常。shell中使用 -x 选项可以调试脚本,可以打印出每一行命令以及当前状态。
```
```
如果是使用脚本作为参数的方式执行脚本，这样启用调试功能：
```
```
$bash  -x  script.sh
```

```
如果是通过将脚本文件作为可执行文件方式，这样启用调试功能：
```
```
#!/bin/bash 修改为
#!/bin/bash -xv
```

* 读取命令的返回值

```
   写shell脚本的时候，我们肯定会遇到需要根据上一个命令返回值来判断下一步的操作，那么获取命令返回值就很有用了。
返回值被称作退出状态。执行成功，退出状态为0，否则为非0
```

```
$?
就是上一条命令的退出状态
```

* 向命令传递参数

```
实际使用中每个命令几乎都有很多可选项,有些可选项还得传递参数。
```

```
如果有一个命令，-p -v 都是可选项，-k N 是另一个可以接受参数数字的选项，同时该命令还接受一个文件名作为参数，那么下面的执行方式都是ok的:
```

```
$command -p -v -k 1 file
$command -pv -k 1 file
$command -pvk 1 file
$command file -pvk 1
```

* 上一个命令的输出作为下一个命令的输入

```
shell脚本最强大的特性之一就是可以轻松地将多个命令组合起来生成输出，一个命令的输出可以作为另一个命令的输入，依次类推。例如：
```

````
$ps -ef|grep java
````

```
例子中ps的输出作为了grep命令的输入
```

```
命令被称作过滤器，使用管道(pipe)连接每个过滤器，管道操作符是|
```

<<<<<<< HEAD
* 存储命令的输出
=======
* 获取命令的输出
>>>>>>> b06b65982dafb31d2e5c74da2130eb80edb93db1

```
注意和命令的返回值的区别，有两种方法。通过加上双引号，可以保留输出的空格和换行符
```

```
子shell法,子shell通过来定义
```

```
$var=$(command)

保留输出的空格和换行符:
$var="$(command)"
```

```
反引用号法：
```

```
$var=`command`
```



