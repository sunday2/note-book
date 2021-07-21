###  echo

```
 echo用于向标准输出打印内容. 命令每次调用后会自动添加换行符, 可以通过-n选项禁止自动添加换行符.

 man:
 Echo the STRING(s) to standard output
```

* demo 1

```
$ echo Hello World !

output:
Hello World !
```

* demo 2

````
$ echo 'Hello World !'

output:
Hello World !
````

* demo 3

```
$ echo "Hello World \!"

output:
Hello World !
```



* 小结

```
可以看到, echo后面参数可以不带单引号, 双引号; 带单引号; 带双引号. 三者各有其副作用, 根据不同的需求选择.

(1)不带引号:
没法在要显示的的文本中使用分号(;), 因为分号在shell中是关键字, 用作命令定界符.
(2)带单引号:
变量替换在单引号无效,也就是单引号括住的内容不会进行扩展.
(3)带双引号:
shell预先定义的特殊字符需要进行转义.
```



### printf

```
同样也是向标准输出打印内容. 每次调用不会自动添加换行符.
man:
Print ARGUMENT(s) according to FORMAT, or execute according to OPTION.
```

* demo 1

```
$ printf hello world!

output:
hello

所以对于字符串包含空格, 不能不加引号, 而echo可以不加.
```



* demo 2

```
$ printf 'hello world!\n'

output:
hello world!
```



* demo 3

```
$ printf "hello world! \n"

output:
hello world!
```



* demo 4

```
$ printf "%-5s %-10s %-4s\n" No Name Mark; printf "%-5s %-10s %-4.2f\n" 1 Sarah 80.3456 


output:
No   Name    Mark
1    Sarah   80.34


这个才是printf的真正灵魂所在.

%s, %f, %d这些称为格式替换符号. 

%-5s指明了一个格式为左对齐,宽度为5的字符串替换,-表示左对齐,不指定-表示右对齐,内容不足5个字符的,会被空格填充;
%-4.2f指明了一个格式为左对齐, 宽度为4, .2f表明对给出的浮点数保留了两位.
```



### variable&&environment variable

```
在bash中, 变量的值都是以字符串形式存储的(无论有没有引号), 所以不需要声明类型.
```

* variable

```
1.变量赋值的格式:
<var>=<value>

注意区分:
<var> = <value>

赋值操作等号两边没有空格, 等号两边有空格的是相等操作.
value根据需求可分别不带引号, 带单引号, 带双引号.


2.变量的引用
(1)echo $var

(2)echo ${var}
建议自定义的普通变量使用该种, 防止出现意想不到的结果.
```



* environment variable(环境变量)

```
在shell或操作系统环境提前定义好的变量称为环境变量.
环境变量是未在当前进程定义, 而从父进程继承而来.
故名思意，灵魂在于环境二字，即上下文，上下文之间是可以嵌套的。或者说作用域，大的作用域里还有小的作用域。
```



```
1.查看环境变量

(1)查看所有与终端相关的环境变量
$ env

(2)查看某个进程运行时的环境变量
$ cat /proc/$PID/environ

所以需要先获取对应进程的进程id, 然后才能查询其运行时的环境变量有哪些
获取进程id的参考命令:
$ ps -ef|grep <进程名>

$ gprep <进程名>
```



```
2.常见的环境变量
HOME, PWD, USER, UID, SHELL, PATH

查看具体的环境变量:
demo 1:
$ echo $PATH

---------------------------------------------------------

demo 2:
$ echo $SHELL

output:
/bin/bash

说明当前使用的shell是/bin/bash


-----------------------------------------------------------
demo 3:
$ echo UID

output:
0

说明当前的登录用户是root用户, 对于某些需要root用户才能执行的脚本,可以使用该变量判断当前用户是否符合, 有无权限执行脚本.

```



```
3.环境变量PATH在哪里定义

PATH环境变量一般定义在.
三者的加载顺序是/etc/environment>/etc/profile>~/.bashrc
所以后加载的会覆盖前面加载的,本质上是作用域的问题.

(1)/etc/environment:  
system-wide configuration file used by all users, root owner.
需要root权限才能修改该文件. 对所有用户有效,理解为操作系统启动的时候执行,即开机的时候执行.
/etc/environment is used for setting variables for programs which are usually not started from a shell.

(2)/etc/profile:   
global initialization scripts for all users.
同样对所有用户有效, 用户登录的时候执行, 比如通过ssh登录.

(3)~/.bashrc:
user's personal shell initialization scripts.
用户独有的配置文件,修改了不会影响其它用户, 每次打开终端之前都会执行一遍该配置文件
.bashrc runs on every interactive shell launch.

The preferred place to set PATH depends on which users you need to set it for and when and how you want it to be set. Part of your decision will be whether you want an environment variable set for all users or on a per-user basis. If you're not sure, then I recommend setting it for just one user (e.g., your account) rather than systemwide

这些配置文件的区别reference:
1.https://askubuntu.com/questions/866161/setting-path-variable-in-etc-environment-vs-profile
2.https://unix.stackexchange.com/questions/129143/what-is-the-purpose-of-bashrc-and-how-does-it-work
```



```
4.设置环境变量
和普通变量不同的是, 环境变量通过export命令设置. 一般修改用户自己的配置文件~/.bashrc来修改PATH.
export命令的含义是be exported to the child processes.

我们打开终端其实就创建了一个shell进程了，~./bashrc文件是在打开终端的时候被执行的，其实在shell的环境了已经设置了一些变量，环境变量这里的环境指的就是当前终端shell进程的环境。我们在终端执行命令的时候，命令也会创建进程，相当于终端shell环境创建的子进程，所以这些子进程会继承来自父进程(终端shell)的变量。


demo 1:
export PATH=$PATH:/home/user/bin
```



### 数学运算

```
在bash shell中, 可以使用let, (()), []执行基本的算术操作, 高级别的可以使用expr和bc这两个工具.
```



### 文件描述符&重定向

```
    file descriptor(FD)是操作系统中文件和其它输入输出资源的唯一的标识符(包括管道和网络socket),这标识符使用整数来表示,比如0, 1, 2... 
    一句话总结就是是IO资源的标识.
    文件描述符用处可大了, 因为在类unix系统中, 一切皆文件, 即使是硬件设备, 在操作系统中都有一一对应的文件抽象, 这类文件称为设备文件.	 
```

* 操作系统预定义的文件描述符

```
操作系统预定义了三个文件描述符,分别是0, 1, 2.
也就是0其实标识的是键盘输入, 1,2标识的都是显示器输出.显然对于显示器输出进行了细分,细分为了标准输出和标准错误.
(1)0:
stdin,标准输入文件描述符
(2)1:
stdout,标准输出文件描述符
(3)2:
stderr,标准错误文件描述符


综上, 我们其实应该将stdin, stdout, stdrrr都理解为文件!!!特殊的设备文件.
```

* 命令的本质

```
命令本质上也是别人写好的程序. 不管什么语言写的程序, 程序内部的代码避免不了有类似标准输出, 标准输入, 标准错误, (即使是java也是如此). 重定向就是将程序内部原先对某些文件的写操作, 重定向到其它文件中.
```

* 输出重定向

```
输出,即写操作.写即得有写入目标文件.
输出重定向, 也就是原先写入目标文件F1, 调整为写入目标文件F2.

也就是重定向就是将命令(程序)内部代码原先默认写入文件F1, 调整为写入文件F2, 所以必定有两个文件, 也就是两个文件描述符, 没有只是省略了. 默认被重定向的文件的是标准输出, 也就是描述符1, 所以可以省略.

重定向放在命令的最后面.

输出重定向操作符:
(1)截断重定向>:
每次会清空重定向的目标文件中的内容,然后再写入.
(2)追加重定向>>:
不会清空目标文件中已经存在的内容, 在已有的内容追加新的内容.
```



* 输入重定向

```
输入重定向, 也就是读操作, 读也得有读入的源文件.
输入重定向, 也就是原先源文件F1读取数据, 调整为从源文件F2读取数据.

输入重定向操作符:
(1)<
(2)<<
```



* 黑洞文件

```
有一个特殊的设备文件:/dev/null
被称为"黑洞", 因为对其写入的内容一去不复返, 都被吞噬了.

用法:
一般如果对于一些stderr太多了,不想在显示屏上显示, 就可以将stderr重定向到/dev/null中.
```



* 管道pipe|

```
管道的作用其实是将前一条命令的stdout(注意,不包括stderr)作为下一条命令的stdin.

所以管道其实也是一种类似的重定向作用, 将stdout重定向到stdin.
```



* 常见例子

```
默认被重定向的是标准输出, 也就是描述符1, 所以可以省略.

demo 1:
$ echo "This is test 1." 1> temp.text
等价于:
$ echo "This is test 1." > temp.text

output:
This is test 1.

截断重定向. 执行echo命令, 并将echo命令的标准输出重定向到temp.text文件中(temp.text中原先的内容会被先被清空), 所以显示屏不会出现"This is test 1.", 因为内容被重定向到temp.text上了.

------------------------------------------------------
demo 2:
基于demo 1, 执行下面命令.

$ echo "This is test 2." 1>> temp.text
等价于:
$ echo "This is test 2." >> temp.text

output:
This is test 1.
This is test 2.

追加重定向. 执行echo命令, 并将其标准输出以追加的模式重定向到temp.text文件. 所以temp.text文件会出现上面的内容, 而显示屏无输出.

---------------------------------------------------
demo 3:
将stderr转换为stdout, 使的stderr和stdout一起重定向到同一个文件中.

ls + 2>&1 output.text
等价于:
ls + &> output.text

output:
ls: cannot access +: No such file or directory

执行命令ls, 并将其标准错误转换为标准输出, 重定向到output.text.
ls + 执行肯定会有标准错误, 因为+不符合ls的参数定义.


----------------------------------------------------

demo 4:
将标准错误重定向到"黑洞文件"中.

假设当前目录下有三个文件a1, a2, a3. 文件内容分别是它们的文件名. 当前用户对a1文件没有读权限.
$ cat a* 2>/dev/null

output:
a1
a2

因为对a1没有读权限, 如果没有stderr重定向,理论上如下.
output:
cat: a1: Permission denied
a2
a3

stderr因为被重定向了, 所以显示屏显得干净了许多.


---------------------------
demo 5:
重定向一般只能重定向到一个文件, 能不能重定向到多个文件呢. 可以的, 结合tee命令即可.

结合demo 4背景.
$ cat a* | tee out.text | cat -n

output:
(1)终端:
cat: a1: Permission denied
1 a2
2 a3

(2)out.text:
a2
a3

tee命令read from stdin and write to file and the stdout.

所以可以看到cat a* 执行后的stdout作为tee命令的stdin被写入到out.text和stdout, stdout又作为cat -n的stdin, 在终端输出.

一开始也许会比较疑惑, 终端的输出第一行是这个,
cat: a1: Permission denied

这显然是stderr, 因为管道只会将stdin作为下一命令的stdin, 所以该stderr是cat a*命令执行后产生的.


---------------------------------
demo 6:
$ cat a* | tee -a out.text | cat -n

类似于demo 5, 只是tee多了个选项, 是以追加的模式写入out.text文件


----------------------------------
demo 7:
$ echo who is this | tee -

output:
who is this
who is this

解析:
stdin可以作为命令的参数选项, 以破折号-作为文件参数选项即可, 因为stdin也是文件抽象.
记住tee的作用就行, read from stdin, write to the file and stdout.

写入文件, 这里的文件就是stdin, 以及写入stdout. 所以终端会显示两行who is this.
```



* 文件重定向到命令(EOF)

```
前面说的都是stdout, stderr重定向到其它文件, 那么反过来, 文件能不能重定向到命令呢, 也就是文件能不能重定向到stdin呢? 肯定可以的, 也就是前面说的输入重定向. 

$ cmd < file

常见的就是cat <<EOF这种, The cat <<EOF syntax is very useful when working with multi-line text in Bash. 这种格式其实被称为heredoc.This is called heredoc format to provide a string into stdin. Basically <<Here tells the shell that you are going to enter a multiline string until the "tag" Here. You can name this tag as you want, it's often EOF or STOP.

here doc格式:

[n]<<word
    here-document
delimiter

EOF被称为here tag, 也可以理解为定界符, 可以自定义为任何单词, 只不过EOF比较符合语义, 它告诉shell, 将会输入多行文本字符串一直到here tag出现.

---------------------------------------------
demo 1:
Assign multi-line string to a shell variableAssign multi-line string to a shell variable.

$ sql=$(cat <<EOF
SELECT foo, bar FROM db
WHERE foo='baz'
EOF
)

解析:
上面例子中, 多行文本将会赋值给变量sql, 平时赋值一般都是单行字符串的赋值, 多行的时候可以考虑该操作.
原本cat的输入参数一般都是一个文件名, 现在输入被重定向为here doc.

------------------------------------------------
demo 2:
Pass multi-line string to a file in Bash.

$ cat <<EOF > print.sh
#!/bin/bash
echo \$PWD
echo $PWD
EOF

output:
(1)print.sh文件内容:
#!/bin/bash
echo $PWD
echo /home/user

解析:
上面例子中首先将here doc作为cat命令的stdin, 然后输出重定向,将执行cat命令后的stdout重定向到print.sh文件中.


--------------------------------------------------
demo 3:


EOF的reference(解释的很清楚):
1.https://stackoverflow.com/questions/2500436/how-does-cat-eof-work-in-bash
```



* 自定义文件描述符(exec命令)

```
文件描述符是文件的符号标识, 其实可以细分为输入文件描述符和输出文件描述符. 预先定义的stdin(0)是输入文件描述符,  stdout(1), stderr(2)是输出文件描述符.
```



```
demo 1:
$ exec 3<input.tex
$ cat<&3

解析:
第一条命令自定义了一个输入文件描述符3, 第二条命令将输入文件描述符3作为cat的原先的stdin, 也就是stdin被重定向为输入文件描述符3. 
注意的是自定义文件描述符使用一次后就得重新定义了, 不能反复使用!!!也就是定义一次,使用一次. 后面要使用得再定义. 自定义文件描述符的重定向比系统预先定义文件描述符的重定向多了个&.

----------------------------

demo 2:
$ exec 4>output.txt
$ echo "hello world." 1>&4

解析:
第一条命令自定义了一个输出文件描述符4, 并且是截断输出的模式; 第二条命令将echo的stdout重定向到了输出文件描述符4, 也就是output.txt中.

---------------------------
demo 3:
$ exec 5>>output.txt
$ echo "the second line." 1>&5

解析:
第一条命令自定义了一个输出文件描述符5, 并且是追加输出的模式; 第二条命令将echo的stdout重定向到了输出文件描述符5中, 也就是output.txt中.
注意的是这里的追加重定向使用的还是>&, 而不是>>&, 这是和预先定义的描述符不一样的地方.


小结:
1.自定义描述符的重定向和预先定义描述符的重定向有一些不一样的地方. 自定义文件描述符的重定向操作符就两种
分别是>&, <&. 而预先定义的文件描述符的重定向操作符有四种, 分别是>, >>, <, <<.
自定义文件描述符输出重定向是以截断模式写入还是追加模式写入在定义的时候就已经决定了.
2.自定义文件描述符定义一次, 使用一次. 再次使用得重新定义.


```



### tee

```
tee命令常用于重定向结合管道符使用.
它从stdin读取数据, 然后写入stdout和文件中.注意的是写入文件可以写多个文件, 而不仅仅是一个文件,还一定会写入stdout; 另外一个就是tee命令只能从stdin读取, 所以stderr就无能为力了(可以折中先将stderr转换为stdin).
所以非常适用于当你需要将某个命令的stdout变成多个数据拷贝的时候.

man:
read from standard input and write to standard output and files.
Copy standard input to each FILE, and also to standard output.

```



````
demo:
tee命令的demo见文件描述符重定向部分.
````



### 数组array

```\
    shell中的数组类型可以分为两种, 一种是普通数组, 以连续的整数作为索引; 一种在shell中称为关联数组(associative array), 使用字符串作为索引.
    关联数组在Bash 4.0版本中开始使用. 从关联数组的定义来看, 有点类似数据结构中的哈希表. 哈希表的底层实现也是基于数组, 字符串索引经过哈希函数处理后,最后得到一个整数作为索引存储到数组中.
```



* 普通数组

```
普通数组的定义, 有两种方式.

demo 1:
$ array_var=(1 2 3 4 5 6)

解析:
这是一种直接给出数组元素的定义方法. 元素之间以空格作为隔离符. 元素会分别存储到以0为索引的连续位置上.

---------------------------------------------

demo 2:
$ array_var[0]="test1"
$ array_var[1]="test2"
$ array_var[2]="test3"
$ array_var[3]="test4"
$ array_var[4]="test5"
$ array_var[5]="test6"

解析:
通过"索引-值"的方式进行定义.
```



```
普通数组的常见操作.
demo 1:
$ echo ${array_var[0]}

解析:
通过echo命令在控制台打印特定索引的数组元素内容.

-------------------------------------------
demo 2:
$ index=2
$ echo ${array_var[$index]}

解析:
通过echo命令在控制台打印特定索引的数组元素内容, 索引是个变量.

------------------------------------------
demo 3:
$ echo ${array_var[*]
等价于
$ echo ${array_var[@]}

outout:
test1 test2 test3 test4 test5 test6

解析:
索引值为*或者@符号, 表明所有索引. 通过echo命令打印出数组所有元素的值, 元素之间以空格作为分隔符. 


------------------------------------------
demo 4:
$ echo ${#array_var[*]}

output:
6

解析:
获取数组的长度, 同样适用于关联数组.
```



* 关联数组(associative array)

```
关联数组的定义只有一种方式.

demo 1:
$ declare -A fruits_value

解析:
通过declare -A <ass_array>的方式声明.
关联数组需要通过declare命令先声明一个变量为关联数组, 选项-A表明是一个关联数组. -a表明是一个普通数组.
```



```
关联数组的赋值操作有两种方式.
demo 1:
$ fruits_value=([apple]='100 dollars' [orange]='150 dollars')

解析:
通过内嵌"索引-值"列表的方式赋值. 列表内元素之间同样以空格作为分隔符.(普通数组由于索引是预先定义好的连续整数, 所以列表赋值不需要提供索引是显而易见的).

-------------------------------------

demo 2:
$ fruits_value[apple]='100 dollars'
$ fruits_value[orange]='150 dollars'

解析:
通过索引-值的方式赋值.(和普通数组一样, 只不过普通数组的索引是整数)
```



```
关联数组常见操作.
demo 1:
$ echo ${!fruits_value[*]}
等价于
$ echo ${!fruits_value[@]}

output:
apple orange

解析:
上面的命令列出了数组中的所有索引. 同样适用于普通数组.

----------------------
demo 2:
$ echo ${fruits_arr[*]}
等价于
$ echo ${fruits_arr[@]}

output:
150 dollars 100 dollars

解析:
同普通数组. 注意的是value之间是以空格作为分隔符, 所以如果value本身也包含空格的话, 会有歧义.
```



### alias

```
alias用来为一大串命令序列设置别名, 从而提供一种快捷方式. 
```

```
demo 1:
$ alias install='sudo apt-get install'

解析:
通过alias <new command>=[command sequence]
我们可以创建别名.
上面的例子中我们为sudo apt-get install创建了别名install. 这样
install gidpin就等价于sudo apt-get install gidpin.

注意的是, 创建别名时, 如果已经存在该别名, 原有别名会被新的覆盖.

------------------------------------------------------
demo 2:
$ echo 'alias <new command>=["command sequence"]'' >> ~/.bashrc

解析:
alias命令的作用是短暂的, 或者说alias的作用域是当前打开的shell进程, shell进程结束后alias就失效了.之前了解过环境变量相关的三个配置文件. 
~/.bashrc配置文件是每次打开shell时(新的shell进程), 就会执行一下该文件(理解为打开shell时一些需要预执行的命令放在这里). 所以可以把常用的alias配置在该文件中. 虽然退出shell就会失效, 但是每次打开新的shell就会执行, 从而达到不用人工每次设置的目的.


----------------------------------------------------------
demo 3:
$ \<command alias>

解析:
对别名进行转义.
alias创建的命令序列别名可能会导致一些安全问题. 有些可能是无意的, 有些可能是恶意的.
毕竟别名的实际对应的命令可能是一些会导致坏结果的命令, 被别有用心之人伪装.
比如, 
rm -rf /* 命令可能被伪装成我们常用的pwd命令, 那么你习惯性的执行pwd, 可能就导致删库跑路的后果.

所以, 提供了对别名的转义, 这样即使有人伪装了新的pwd命令, 对pwd命令进行转义, 就能使用最原始的那个pwd命令.

另外, 也告诉我们正常来说, 高危敏感的操作, 特别是在生产环境, 比如rm之类的, 尽量就不要为其创建别名了.
```



### tput && stty(获取和处理终端的设置)

```
写shell脚本时, 有时避免不了要处理当前终端的信息, 比如行数, 列数, etc.

tput和stty是两款终端处理工具(命令), 可以帮助采集和处理终端的设置.
```

```
demo 1:
$ tput setb <n>
$ tput setf <n>

解析:
上面这两条命令分别可以设置终端的背景色,终端文本前景色, n的取值范围为从0到7.

------------------------------------------
demo 2:
$ tput cols
$ tput lines

解析:
上面两条命令分别用于采集终端的列数和行数.

```



### date

```
无论哪一门编程语言, 不可避免得和日期打交道.
```

* Unix时间

```
  类Unix系统中, 时间被存储成一个整数. 其大小为自世界标准时间(UTC)1970年1月1日0时0分0秒起所流逝的秒数, 这种计时方式称为纪元时或Unix时间.
  当时计算两个时间之差时, 纪元时间/Unix时间非常有用.
```

* 常见例子

```
demo 1:
$ date

output:
Sun Jul 18 20:42:04 CST 2021

分析:
该命令获取系统当前的时间.


-----------------------------------------------
demo 2:
$ date +%s

output:
1626612199

分析:
该命令获取系统当前时间, 并且以纪元时的格式.

---------------------------------------------
demo 3:
$ date --date="Thu Nov 18 08:07:21 IST 2010" "+%s"

output:
1290047841


$ date --date="Jan 20 2001" "+%A"

output:
Saturday


$ date "+%d %B %Y"

output:
18 July 2021


解析:
上面的例子中属于同一个范畴, 可以将某个格式的具体日期通过格式串作为date命令的参数转换为另外一种格式的日期.
给出的具体日期可以是当前日期时间, 若要被转换的日期时间是当前日期则不需要给出, 因为默认就是当前日期时间.
当需要某个日期时间时, 则需要指定要被转换的日期时间.


-----------------------------------------------
demo 4:
$ date -s "21 June 2009 11:01:22"

解析:
date -s <格式化的日期字符串>
设置系统的时间.
上面的例子通过-s/--set选项可以设置当前系统的日期时间.

-----------------------------------------------
demo 5:
#!/bin/bash
start=$(date +%s)
<command>
end=$(date +%s)
delta=$(( end-start ))
echo Time taken to excute the command is ${delta} seconds.

解析:
上面的脚本片段中通过两个纪元时计算出了两者之间的命令所耗费的时间.

如果是需要计算整个脚本的执行时间可以通过time命令, 即time<scriptpath> 来得到整个脚本的执行耗费时间.
```



### sleep

```
sleep命令让程序休眠，基本编程语言都有类似的实现。

demo 1:
$ sleep <seconds>

解析：
sleep可以让脚本睡眠n秒后才执行。

demo 2:
#!/bin/bash
echo -n Count:
tput sc
count=0
while true;
do
  if [ $count -lt 40 ];
  then
      let count++;
      sleep 1
      tput rc
      tput ed
      echo -n $count
  else exit 0;
  fi
done

解析：
根据经验，编写循环方式运行的监视脚本，一般每次都需要休眠时间，或者说冷却期之类的。这个脚本里还使用了tput的终端工具，由于平时比较少用，需要花点时间理解下。

tput sc: sc即store cursor, 存储当前光标的位置.
tput rc: rc即reset cursor, 重置当前光标的位置.
tput ed: 清除当前光标位置到行尾之间的所有内容.

顺序为sc, rc, ed. 即记住，重置，清除。

上面的脚本执行后类似：
Count:<n>

n从0到40之间连续跳动。
```



### 调试脚本

```
调试脚本的四种方式。
(1)bash -x <script>
或者
sh -x <script>

(2)在脚本里第一行如下：
#!/bin/bash -xv


(3)局部调试
在需要调试的命令部分
前面加上
set -x
<command>
set +x

解析：
-x: 在执行时显示参数和命令, 将脚本中执行过的每一行都输到stdout中。
+x: 禁止调试.
-v: 当命令进行读取时显示输入.
+v: 禁止打印输入.


(4)自定义显示调试信息
前面三种方法都是bash里build in的debug方式，会以固定的格式生成调试信息。

有些时候我们也许需要一些不一样的调试信息，那么我们可以自定义一个DEBUG函数，放在脚本中的每条命令前面，提供一个debug选项来控制是否需要debug。

下面是一个自定义debug的脚本例子:
-------------------------------------
#!/bin/bash
function DEBUG(){
   [ "$_DEBUG" == "on" ] && $@ || : 
}

for i in (1..10)
do
    DEBUG echo $i
done

--------------------------------------
debug模式调用脚本：
$ _DEBUG=on ./script.sh

解析：
自定义debug关键的就是自定义的DEBUG函数. 在DEBUG函数中，test命令测试_DEBUG标志是不是on，是则继续执行
后续命令，$@是输出DEBUG函数所接收的所有参数，实际就是命令序列， 因为我们看到下面这行:
DEBUG echo $i

所以$@的值就是echo xxx的命令序列。

注意的是命令序列后面跟着的是||运算符，:命令告诉shell什么都不需要做，有点类似于python中的pass.

实际中可以根据自己需要输出自己想要的信息。

-------------------------------------

sleep命令中的脚本通过
bash -x <script>的部分输出如下：

+ echo -n Count:
Count:+ tput sc
+ count=0
+ true
+ '[' 0 -lt 40 ']'
+ let count++
+ sleep 1
+ tput rc
+ tput ed
+ echo -n 1
1+ true
+ '[' 1 -lt 40 ']'
+ let count++
+ sleep 1
+ tput rc
+ tput ed
+ echo -n 2
2+ true
+ '[' 2 -lt 40 ']'
+ let count++
+ sleep 1
+ tput rc
+ tput ed
+ echo -n 3
3+ true
+ '[' 3 -lt 40 ']'
+ let count++
+ sleep 1
+ tput rc
+ tput ed
+ echo -n 4
4+ true


解析：
可以看到脚本中执行过的命令都输出到stdout中，以加号'+'作为前缀。
命令: echo -n $count
$count变量被解析成具体的值出现在stdout中。
并且执行结果也在stdout中。
```



### 自定义函数

```
编程语言里基本都支持函数，shell script也不例外。
```

* 函数的定义

```
两种方式。
(1)
function fname()
{
   statements;
}

(2)
fname()
{
   statements;
}
```

* 函数的调用

```
(1)无参调用
$ fname;


(2)有参调用
$ fname arg1 arg2;
```

* 函数参数的访问

```
$1: 第一个参数
$2: 第二个参数
...
$n: 第n个参数

$@: 被扩展成"$1" "$2" "$3"...
$*: 被扩展成"$1c$2c$3", 其中c是IFS的第一个字符。

tips: $@要比$*使用的频率高，因为$*是将所有参数当作单个的字符串体。

----------------------------------------
demo 1：
function fname()
{
  echo $1, $2;  #访问参数1，参数2
  echo "$@";    #以列表的方式一次性打印所有参数
  echo "$*";    #类似于$@, 但是参数被作为单个实体
  return 0;     #返回值
}


小结：
可以发现参数是通过$1-$n的方式进行访问的。
对于脚本也是类似，可以通过$1-$n访问传递给脚本的参数。
注意区分脚本作用域(全局作用域)和函数作用域。
```

* 导出函数

```
类似环境变量，函数也可以被导出。也就是可以在/etc/environment,/etc/profile,~/.bashrc这些文件中定义函数并且使用export命令导出这些函数。

------------------------
demo 1:
export -f fname

解析:
那么该函数的作用域就能扩展到child process, 注意这里的选项比导出环境变量多了个-f.
```



* 读取命令返回值(状态)

```
注意这里是命令command的返回值，不是指我们自定义的函数（虽然函数也可以理解为command）。而且是上一条命令的返回值。

即:
---------------------------------------
demo 1:
#!/bin/bash
cmd;
echo $?;

解析：
$?存储了前一条命令的返回值，返回值被称为退出状态，退出状态也许更容易理解。如果命令成功退出，那么退出状态为0，否则为1。注意这里成功的状态是为0，不要受惯性思维影响，想当然以为是1.

-----------------------------------------
demo 2：
#!/bin/bash
CMD="command" ##指代你要检测退出状态的命令
${CMD}
if [ $? -eq 0 ];
then
  echo "$CMD executed successfully"
else
  echo "$CMD terminated  unsuccessfully"
fi

解析：
这是一个很接近实际的一个例子。在脚本里检查某条命令的执行结果，在stdout输出相关的信息。
```



*  向命令传递参数

```
命令传递参数是一个很常见的场景，注意这里是说命令传递参数而不是函数传递参数。

命令的参数可以以不同的格式进行传递。参数也有了其它的细分化的说法或者说概念，比如可用选项，参数选项，参数。

可用选项：
指的是标志位，由于要么真，要么是假。所以要么给出该选项，要么不给出该选项即能表达语义，不需要带参数。
类似要么带-d，要么不带-d。

参数选项：
选项后面还得跟参数才能表明语义。类似-d <arg1>这种格式。

参数：
参数就是普通的参数，比如文件路径这种具体的参数。

---------------------
demo 1:
假设-p, -v是可用选项, -k N是另外可接受数字的选项，同时该命令还接受一个文件名作为参数， 那么以下几种写法等价。

$ command -p -v -k 1 file
$ command -pv -k 1 file
$ command -pvk 1 file
$ command file -pvk 1

解析：
记住这几种写法是等价的。
```



### 命令序列的输出读入变量(管道)

``` 
其实前面也有涉及到管道的知识点。因为管道是shell脚本最棒的特性之一，而且使用频率很高，有必要单独总结一下。

这些命令被称为过滤器，管道连接每个过滤器，管道操作符是|.

格式如下:
$ cmd1 | cmd2 | cmd3

前面有提到管道操作符有点像重定向。
它将前一条cmd的stdout作为下一条cmd的stdin。

所以注意了，管道操作符不会将前一条cmd的stderr作为下一条cmd的stdin。
```



```
demo 1:
$ ls | cat -n > cut.txt

解析：
ls命令的stdout，也就是当前目录的内容列表会作为下一条命令，也就是cat -n的stdin，最后还有一个stdout的重定向操作，cat命令的stdout被重定向到cut.tet文件中。

-------------------------------------------------------------------
demo 2:
#!/bin/bash
CMD_OUTPUT=$(ls | cat -n)
echo ${CMD_OUTPUT}


等价于：
#!/bin/bash
CMD_OUTPUT=`ls | cat -n`
echo ${CMD_OUTPUT}

解析：
这里有一个在script中经常用到的知识点，就是获取命令的输出.通过
$(<commands>)
可以获取到命令的输出，这种方式被称为子shell。

为什么称为子shell呢，说明里面的commands是在当前shell创建的子shell，也就是另外的进程中执行的！！！

还有一种是通过:
`<commands>`
的方式，这种被称为反引用(反标记)。
```



* 获取命令序列的输出

```
共有两种方式，这里的输出包括stderr和stdout。

见上面的demo 2。
```



* 利用子shell生成一个独立的进程

```
    要保持有操作系统进程的意识，透过现象看本质。也许表面只是执行了一下command，但是command本身也是程序，所以肯定在操作系统层面有进程上的体现。而父shell有时候也会创建子shell(一个新的进程)，让command在子shell中执行。
    所以，有时执行ps -ef|grep xxx的时候你会发现显示的其中一个进程是grep命令导致的进程。面试题问进程间如何通信的时候，想当然的回到到管道，很多人也许没有发现平时就在使用管道来进行进程间的通信。
    ps -ef|grep xxx这条命令序列中，ps程序创建其进程，grep程序也创建了其对应的进程，管道将ps进程执行结果传递给了grep进程，这不就是活生生通过管道进行进程间的同行吗。
    总结就是在shell中无时不在创建新的进程，要有透过现象看本质的意识。进程可能来源于各种command，可能是子shell。
    
    
---------------------------------------
demo 1:
#!/bin/bash
pwd;
(cd /bin; ls);
pwd;

解析：
上面脚本中通过()定义了一个子shell，子shell本身就是一个独立的进程。故子shell中的cd命令和ls命令都在独立的一个进程运行。

需要注意的是子shell中执行的命令，其变化不会影响到父shell，所有的改变仅限于子shell的环境内！！！
所以子shell里虽然cd命令进入到了另外一个目录，但是对于父shell来说，当前目录还是不会变。



----------------------------------------
demo 2:
(1)
$ cat test.txt
output:
1
2
3

(2)
$ out=$(cat test.txt)
$ echo $out
output:
123   #Lost \n spacing in 1,2,3

(3)
$ out="$(cat test.txt)"
$ echo $out
output:
1
2
3

解析：
从上面三个例子中可以发现，通过子shell的方式获取命令的输出，如果不带双引号的，输出的换行符和空格会丢失。

```



### read



























