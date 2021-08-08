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

```
从键盘读取输入的时候，要按下enter来表示输入结束，但是有时候这样子很麻烦，read提供了一个不需要按回车键就能解决的方案。
read是一个很重要的bash命令，它用于从键盘或者标准输入中读取文本，并且可以以交互的形式读取来自用户的输入。
```

```
demo 1:

read -n <number_of_chars> variable_name

$ read -n 2 var

解析：
上面的命令表示从输入中读取2个字符并存入变量var中。注意的是不需要按回车键就能读取键盘的输入，2个字符后会自动结束从键盘读取，不需要按回车键。

-------------------------------------------
demo 2:
$ read -s var

解析：
用无回显的方式读取密码，存到变量var中。 有时候我们需要读取键盘输入，但是是比较敏感的数据，不需要回显，那么可以使用该命令。需要回车键作为读入的结束标志。


----------------------------------
demo 3:
$ read -p "Enter input:" var

output:
Enter input:hello

$ echo $var
output:
hello

解析：
上面的命令表示从键盘读取输入到变量var，而且有提示信息。需要回车键作为读入的结束标志。

------------------------------------
demo 4:

read -t timeout var

$ read -t 2 var

解析：
上面的命令表示在2秒内读取键盘输入到变量var中。这里不需要回车键作为结束标志了，2秒后自动结束读取，不要按回车键。


------------------------------------
demo 5:

read -d <delim_char> var

$ read -d ":" var

解析:
用特定的定界符作为读取键盘输入的结束。上面的例子中，定界符是冒号，也就是读到冒号的时候自动结束读取，不需要回车键。
```



### 运行命令直至成功 && 重试机制

```)
运行命令直至成功在实际中可能用的不多，原因如下：
(1) 如果命令里依赖了第三方的服务，那么意味着第三方服务是不可控的，没人能保证第三方服务一定可靠。
(2) 不成功一直重试发请求，有可能会把第三方服务搞挂了。

实际使用更多的是有重试次数的限制，设置重试计数器，达到阈值就不重试了。
```

* 定义重复函数

```
(1)
function repeat()
{
  while true
  do
    $@ && return
  done
}

(2)
function repeat()
{
  while :
  do
    $@ && return
  done
}

上面两种写法在功能上是等价的，但是第二种写法性能上会稍微好一点。在于while后面跟的是true还是内建的:命令。
在大多数现代系统种，true是作为/bin中的一个二进制文件来实现的，也就意味着true会单独生成一个进程，这需要耗费资源和时间，而:命令是shell内建的命令，类似于一个函数，而true是一个单独完整的程序。:命令总是返回0的退出码(注意，shell中退出码为0表示成功，也就是true)。

还有一点就是repeat函数成功后的退出是通过&&操作符，然后return来终止循环的，而不是break。可以发现命令之间通过||和&&连接可以达到简化代码的目的，注意两种的区别。

上面例子中，如果$@命令执行成功了才会执行&&连接着的return命令，否则不会执行，也就是回一直循环。
```

* 增加延时

```
function repeat()
{
  while :
  do
    $@ && return
    sleep 30
  done
}

解析:
可以看到在循环体里面加上了sleep 30秒的缓冲时间，意思是命令执行不成功，先休眠30秒再进入下一次的循环体。
主要考虑的是如果命令里有类似网络请求或者io操作，起到缓冲作用。这是一种很常规的编程范式，或者说编程思想。
```

* 添加重试次数而不是无限

```
function repeat()
{
  local cout=0;
  while :
  do
    if [[ count -ge 3]];
    then
      return;
    fi
    $@ && return;
    count++;
    sleep 30;
  done
}

解析：
上面的例子在循环体中加上了重复执行命令的次数判断，执行了3次后就直接返回了，实际中可以打印下日志从而区分是函数是重试后还是失败了。
重试机制也是一种编程思想，一种编程范式。
```

* demo

```
demo 1:

#!/bin/bash

function repeat()
{
  local cout=0;
  while :
  do
    if [[ count -ge 3]];
    then
      return;
    fi
    $@ && return;
    count++;
    sleep 30;
  done
}

repeat wget -c http://xxxxx

解析：
在wget命令前加上repeat函数，也就是将wget命令传递给repeat函数。
```



### 字段分隔符和迭代器

```
内部字段分隔符(Internal Field Separator, IFS)是shell脚本编程中的一个重要概念。特别是在处理文本的时候作用很大。

IFS是特定用途的定界符(delimeter)。其它定界符还有命令的定界符是分号;和换行符/n。

IFS是存储定界符的环境变量。

也就是说是和环境变量PATH，UID，SHELL一样的，在终端通过echo $IFS可以看到该定界符是什么。(由于默认的IFS是空白字符，也就是可能是空格，水平制表符，换行符这种，所以肉眼看不出)。
```



* IFS的使用

```
demo 1：
#!/bin/bash
data="name,sex,rollno,location"
OLD_IFS=$IFS
IFS=, #now,

for item in $data;
do
    echo item: $item
done

IFS=$OLD_IFS #reset

解析：
脚本里有一个字符串，字符串中的内容是以逗号分隔，我们称这种数据为逗号分割型数值(comma seperated value,CSV)。
我们想要迭代该字符串，那么可以将IFS设置为逗号，shell则会将逗号看作内部定界符，直接可以对字符串进行迭代。

注意的是，迭代完之后，需要将IFS恢复为原来的默认值，所有修改IFS之前要保留拷贝，处理完之后需要恢复原设置。、


---------------------------------------------------
demo 2:
#!/bin/bash
line="root:x:0:0:root:/root:/bin/bash"
OLD_IFS=$IFS
IFS=":"
count=0
for item in $line;
do
    [ $count -eq 0 ] && user=$item;
    [ $count -eq 6 ] && shell=$item;
    let count++
done;
IFS=$OLD_IFS
echo $user\'s shell is $shell;

output:
root's shell is /bin/bash

解析：
脚本里的line字符串其实来自于系统里的/etc/passwd文件，文件里每一行包含了每一位系统用户的相关属性，每一行格式是固定的，以冒号作为定界符。第一个元素是用户名，最后一个元素是该用户对应的默认shell。

和demo 1一样通过迭代字符串获取对应的值，这里字符串定界符是冒号。
```



### 循环语句（loop statement）

```
无论哪种编程语言，循环语句都是必要的。在shell script，有三种loop的格式。

(1)for的形式，细分为两种。
for var in list;
do
    commands; #使用变量$var
done

解析：
list可以是一个字符串，也可以是一个序列。字符串的化注意当前IFS是什么。

for((i=0;i<10;i++))
{
	commands; #使用变量$i
}


(2)while循环
while condition;
do
	commands;
done

解析：
用true或者冒号命令作为循环条件，可以产生无限循环。



(3)until循环
#/bin/bash
x=0;
until [ $x -eq 9 ];  #条件是[ $x -eq 9 ]
do
	let x++;j echo $x;
done

解析：
until循环会一直循环，一直到给定的条件为真。
```

* 生成连续的数字列表或者字母列表

```
{<num1>..<num2>}可以生成从num1到num2的数字列表。num1<num2.
{<letter1>..<letter2>}可以生成从letter1到letter2的字母列表。letter1和letter2要么都小写要么都大写，letter1在字母表中排在letter2前面。

demo 1:
$ echo {1..10}

output:
1 2 3 4 5 6 7 8 9 10

解析:
可以看到生成了1到10的数字列表，元素之间是空格作为定界符。(说明默认的IFS是空格?)

demo 2:
$ echo {a..j}

output:
a b c d e f g h i j

解析:
生成了字母列表，元素之间空格作为定界符。


demo 3:
$ echo {A..J}

output:
A B C D E F G H I J

解析:
生成了A到J的字母列表，空格作为定界符。
```



### if语句（比较与测试）

```
在编程语言中，if语句也是必要的，它控制着程序的流程。
```

* 模板

```
(1)
if condition;
then
	commands;
fi

解析:
简单的if，没有else也没有嵌套。


(2)
if conditon;
then
	commands;
else if condition; then
	commands;
else 
	commands;
fi

解析：
if else。
```



* test命令[]

```
test命令用于判断条件，其符号为方括号[].

格式如下:
[ condition ]
等价于
test condition

注意的是使用方括号的方式，条件和方括号之间有空格，不能紧挨着放括号，否则报错。

demo 1：
$ if [ $var -eq 0 ]; then echo "True"; fi
等价于
$ if test $var -eq 0; then echo "True"; fi
```



* 逻辑运算符简化条件判断

```
有时候if的条件判断部分会特别长，可以考虑逻辑运算符简化。

(1)
[ condition ] && <action>: &&是逻辑与运算符。如果condition为真，则继续执行后面的action；否则，不执行后面的action

(2)
[ condition ] || <action>: ||是逻辑或运算符。如果conditon为真，则不执行后面的action; 否则，执行后面的action。

这是一个很有用的技巧，而且每一条命令本身也是一个条件。 即命令本身就是条件，命令状态返回码为0表示成功，等价于true，命令返回码非0表示失败，等价于false。
```



* 比较运算

```
1.算术比较
demo 1:
$ [ $var -eq 0 ] 

解析：
-eq是-equals的缩写。所以当$var等于0的时候，test命令返回真；当不等于0的时候，test命令返回假。

----------------------------

demo 2:
$ [ $var -ne 0 ]

解析:
-ne是-not equals的缩写。所以当$var非0的时候，test命令返回真；当等于0的时候，test命令返回假。


--------------------------------
demo 3:
$ [ $var1 -ne 0 -a $var2 -gt 2 ]

解析:
可以使用逻辑与-a结合多条件进行判断。-a即-and。


--------------------------------
demo 4:
$ [ $var1 -ne 0 -o $var2 -gt 2 ]

解析；
可以使用逻辑或-o结合多条件判断。-o即-or。


其它常见的算术比较符如下：
-gt: greater than, 大于
-lt: less than, 小于
-ge: greater than or equals, 大于或等于
-le: less than or equals, 小于或等于
```



````
2.文件系统相关的测试
可以使用不同的条件标志测试文件系统相关的不同属性。

demo 1:
$ [ -f $file_var ]

解析：
如果给定的变量$file_var包含正常的文件路径或文件名，则返回真。

----------------------------------
demo 2:
$ [ -x $var ]

解析：
如果给定的变量包含的文件是可执行文件，则返回真。

------------------------------------
demo 3:
$ [ -d $var ]

解析：
如果给定的变量是目录文件，则返回真。

------------------------------------
demo 4：
$ [ -e $var ]

解析：
如果给定的变量包含的文件存在，则返回真。

-------------------------------------
demo 5:

$ [ -c $var ]

解析：
如果给定的变量包含的是一个字符设备文件的路径，则返回真。


----------------------------------------
demo 6:
$ [ -b $var ]

解析：
如果给定的变量包含的是一个块设备文件的路径，则返回真。

------------------------------------------
demo 7:
$ [ -w $var ]

解析：
如果给定的变量包含的是一个可写的文件的路径，返回真。


---------------------------------------
demo 8：
$ [ -r $var ]

解析：
如果给定的变量包含的是一个可读的文件的路径，返回真。

-------------------------------------------
demo 9：
$ [ -L $var ]

解析：
如果给定变量包含的是一个符号链接，返回真。
````



```
3.字符串比较
字符串比较时，test符最好使用双中括号，因为有时候单个中括号会产生错误，最好避开它们。

demo 1:
$ [[ $str1 = $str2 ]]

等价于
$ [[ $str1 == $str2 ]]

解析:
判断两个字符串是否相等。单个=或者双等号都可以，单等号注意左右两边都有空格，此时才表示相等，如果没有空格预留则表示赋值操作。

------------------------------------------------------------ 
demo 2:
$ [[ $str1 != $str2 ]]

解析：
判断两个字符串不相等，不相同返回真，否则返回假。

-------------------------------------------------------------
demo 3:
$ [[ $str1 > $str2 ]]

$ [[ $str1 < $str2 ]]

解析：
判断两个字符串的字母序。

-------------------------------------------------------------
demo 4:
$ [[ -z $str1 ]]

$ [[ -n $str1 ]]

解析:
上面两条命令分别判断$str1变量是空字符串还是非空字符串。

```

* 多条件组合

```
通过逻辑运算符&&和||组合多个条件。
demo 1:
#!/bin/bash

str1="Not empty"
str2="''"
if [[ -n $str1 ]] && [[ -z $str2 ]];
then 
	commands;
fi
```



### cat

```
cat用于拼接多个文件并输出到stdout，其截取自单词concatenate(拼接)。多个文件指的是1到多个。
man:
concatenate files and print on the standard output.

基本用法:
cat file1 file2 file3
```

```
demo 1:
$ cat one.txt

output:
This is line from one.txt

$ cat two.txt

output:
This is line from two.txt

$ cat one.txt two.txt

output:
This is line from one.txt
This is line from two.txt


解析：
可以看到cat根据文件的顺序依次将文件的内容拼接在前一文件的最后一行中，并输出到stdout。

-----------------------------------------
demo 2:
$ echo "Text through stdin" | grep - one.txt

output:
Text through stdin
This is line from one.txt

解析：
可以看到cat同样可以读取stdin的内容，或者说stdin本身也是文件的概念，注意的是用-符号来表示stdin。


-----------------------------------------
demo 3:
$ cat lines.txt

output:
line1
line2
line3

$ cat -n lines.txt

output:
1 line1
2 line2
3 line3

解析：
加上-n选项可以看到可以在stdout中加上行号。

----------------------------------------
demo 4:
$ cat file.py

output:
def function():
	var = 5
	    next = 6
	third = 7
	
$ cat -T file.py

output:
def funcion():
^Ivar = 5
        next = 6
^Ithird = 7^I

解析:
将制表符标记为^I, 在排除缩进错误的时候特别有用。因为有些文本对缩进要求很严格，比如yaml格式，python脚本等。这时可以考虑使用cat -T选项。注意的是水平制表符对应的是键盘上的tab键，得留意linux的当前tab键的宽度。

上面的例子中，显然系统中的制表符的宽度是4的，那么通过cat -T我们可以发现在next = 6这一行缩进是有问题的，那么就会最终导致python脚本语法错误。

注意！！！水平制表符tab键和n个空格并不是等价的，tab是一个符号，但是并不等价于n个空格。如果文本里的缩进是使用的tab来达到缩进的目的，那么在不同编辑器下可能会有差异，因为有的编辑器tab符合的长度是4，有些设置的是8，会容易引起版本冲突。

所以，一般缩进我都是用空格而不是tab，所以cat -T这条命令可能就帮不上忙了。

reference:
tab的宽度是terminal的属性而不是shell的属性，可以通过tabs命令设置tab的宽度。
1.https://stackoverflow.com/questions/10782699/how-to-set-4-space-tab-in-bash

------------------------------------------
demo 5:
$ cat multi_blanks.txt

output:
line 1


line 2



line 3




line 4

$ cat -s multi_blanks.txt

output:
line 1

line 2

line 3

line 4


分析：
cat -s可以压缩多余的空白行，或者说美化在stdout的输出。上面的例子中加上-s后，所有行之间都是以一行空白行作为分割。
```



### script & scriptreplay(录制并回放终端会话)

```
当你需要为别人在终端演示某些操作或是准备一个命令行教程时，也许你可以一遍手动输入命令，一遍演示，这是有一种实时的演示方式；也可以录制一段屏幕演示视频，然后再回放出来。

显然回放这种可以达到reuse的目的，个人觉得更好。
利用script和scriptplay命令，可以录制命令的次序以及时序，记录在文本文件中，最后可以回放的目的。
```



### find(文件查找和文件列表)

```
find命令沿着文件层次结构向下遍历，匹配符合条件的文件，执行相应的操作。

并且可以根据文件的一些属性(名字，文件类型，大小，时间戳等)作搜索的限制。
```



* 通过文件名搜索(-name/-iname)

```
demo 1:

$ find <basepath> [-print]

$ find . -print

output:
.
./site_values_1.11.0-118.yaml
./site_values_1.11.0-118.yaml.bak
./edit_site_value_file.log
./edit_site_value_file.py
./foo.template
./foo.conf
./eccd-env.yml.template
./sitefile
./sitefile/site_values_1.11.0-118.yaml
./sitefile/site_values_1.11.0-118.yaml.bak
./test.md
./3
./study
./study/cat-test.txt
./test.sh

解析：
列出当前目录和子目录下所有的文件和文件夹。
-print选项用于打印匹配的文件和文件夹的路径，并且使用\n, 即换行符作为输出的定界符。可以忽略-print, 默认也是\n作为输出定界符。特殊情况下，可以指定-print0指定\0,即null作为输出定界符，特别是文件名包含换行符的时候很有用。

---------------------------------------------
demo 2:
$ find  /home/slynux -name "*.txt" -print

$ find  /home/slynux -iname "*.txt" -print


$ ls

output:
example.txt EXAMPLE.txt file.txt

$ find . -iname "example*" -print

output:
./example.txt
./EXAMPLE.txt

解析:
根据文件名在某个目录下进行搜索。
-name或者-iname选项，表示根据文件名进行搜索。两者的区别在于前者不忽略文件名的大小写，后者忽略文件名的大小写问题。
-print选项可以忽略，一般忽略不写。
注意的是文件名可以使用通用符也就是*达到模糊搜索的目的。


---------------------------------------------
demo 3:
$ ls 

output:
new.txt some.jpg text.pdf

$ find . \( -name "*.txt" -o -name "*.pdf"\) -print

output:
./new.txt
./text.pdf

解析：
通过OR运算符可以匹配多个文件名条件中的一个。
\(以及\)用于将多个文件名条件作为一个整体。

----------------------------------------------
demo 4:
$ ls 

output:
list.txt new.py new.txt next.jpg test.py

$ find . ! -name "*.txt"

output:
./next.jpg
./test.py
./new.py

解析：
通过感叹号!可以否定参数的含义。
```



* 通过文件路径搜索(-path)

```
-path选项将整个文件路径作为一个整体进行匹配, 同样，可以使用通配符*.

demo 1:
$ find /home/users -path "*/slynux*" -print

output:
/home/users/list/slynux.txt
/home/users/slynux/eg.css

解析：
根据文件路径进行匹配。
```



* 通过正则表达式搜索(-regex和-iregex)

```
-regex和-path有点类似，都是匹配文件路径，区别是-regex比-path更灵活，毕竟可以使用正则，而-path只是可以使用通配符。-iregex表示可以忽略大小写。

demo 1:
$ ls

output:
new.PY next.jpg test.py

$ find . -regex ".*\(\.py\|\.sh\)$"

output:
./test.py

$ find . -iregex ".*\(\.py\|\.sh\)$"

output:
./test.py
./new.PY

解析：
匹配符合正则表达式定义的文件路径。
```



* 限制搜索的深度(-maxdepth和-mindepth)

```
find命令默认使用时，会遍历所有的子目录，可以通过-maxdepth和-mindepth来限制搜索的深度。
注意的是-maxdepth，-mindepth是指从基础目录向下遍历的最大深度和最小深度，而不是从原始的目录开始数。

demo 1:
$ find . -maxdepth 1 -name "f*"

解析:
指定最大深度是1，所以该命令列出基础目录也就是当前目录下所有文件或文件夹以f打头的，即使有子目录，也不会被打印或遍历。

-----------------------------------
demo 2:
$ find . -mindepth 2 -name "f*"

output:
./dir1/dir2/file1
./dir3/dir4/file2

解析:
指定了mindepth为2，也就是至少从距离基础目录也就是当前目录两个子目录才开始搜索。


tips:
注意如果要使用-maxdepth或-mindepth, 那么最好作为find的第三个参数出现。
因为和其它选项比如-type搭配使用时，前后顺序会影响find的搜索效率。
```



* 根据文件类型搜索(- type)

```
类Unix系统中将一切都视为文件。文件具有不同的类型，比如普通文件，目录，字符设备，块设备，符号链接，硬链接，套接字，FIFO等。
下面是类型参数列表：
(1)普通文件: f
(2)符号链接: l
(3)目录: d
(4)字符设备: c
(5)块设备: b
(6)套接字: s
(7)FIFO: p


demo 1:
$ find . -type d

解析：
搜索当前路径下的目录文件。

---------------------------------
demo 2:
$ find . -type f

解析:
搜索当前路径下的普通文件。

_---------------------------------
demo 3:
$ find . -type l

解析:
搜索当前路径符合的符号链接文件。
```



* 根据文件时间搜索(-atime, -mtime, -ctime/-amin,-mmin,-cmin)

```
类Unix系统中每一个文件都有三个时间戳。注意，类Unix系统中文件没有创建时间的概念。可以通过stat命令查看文件的三个时间戳。find命令的时间戳选项对编写系统备份和维护脚本很有帮助！！!
(1)访问时间(access): 用户最近一次访问文件的时间。
(2)修改时间(modify): 文件内容最后一次被修改的时间。
(3)变化时间(change): 不要和修改时间混淆，修改时间指的是文件内容的修改时间；变化时间指的是权限或所有权的变化时间，指的是文件的属性的变化。

-xtime和-xmin的区别是时间单位不一样，-xtime的时间单位是天；-xmin的时间单位是分钟。

这些选项的值带有+或者-号或者不带。值带+表示大于，带-表示小于，不带表示等于刚刚好。

demo 1:
$ find . -type f -atime -7

解析:
该命令表示在当前路径下搜索普通文件，且是最近7天访问过的.


---------------------------------------------
demo 2:
$ find . -type f -atime 7

解析:
该命令表示在当前路径下搜索普通文件，且是刚好7天前访问过的.

----------------------------------------------
demo 3:
$ find . -type f -atime +7

解析:
该命令表示在当前路径下搜索普通文件，且是访问时间超过7天的。


-------------------------------------
demo 4:
$ find . -type f -amin +7

解析:
该命令表示在当前路径下搜索普通文件，且是访问时间超过7min的。

---------------------------------------
demo 5：
$ find . -type f -newer file.txt

分析:
-newer选项，比较的是文件的修改时间更近的。

上面的例子是搜索当前路径下的所有普通文件，找出比file.txt文件的修改时间，注意是修改时间不是变化时间也不是访问时间更近的文件。(更近也就是更后面发生的)
```



* 基于文件大小的搜索(-size)

```
根据文件大小进行搜索。

大小的单位如下:
b: 块(512字节)
c: 字节
W: 字(2字节)
k: 1024字节
M: 1024k字节
G: 1024M字节

带减号表示小于，带+号表示大于，不带表示大于等于

demo 1:
$ find . -type f -size +2k

解析:
在当前路径搜索大于2KB的普通文件。

--------------------------
demo 2:
find . -type f -size -2k

解析:
在当前路径搜索小于2KB的普通文件。

----------------------------
demo 3:
find . -type f -size 2k

解析：
在当前路径搜素大于等于2KB的普通文件。
```



* 删除匹配到的文件(-delete)

```
通过-delete选项可以删除find命令匹配到的文件。

demo 1:
$ find . -type f -name "*.swp" -delete

解析:
删除当前目录下所有的.swp文件。
通过在find命令最后加上-delete选项，可以删除匹配到的文件。
```



* 通过文件权限和所有者的搜索(-perm和-user)

```
通过-perm可以搜索匹配文件权限的文件。
通过-user可以搜索匹配文件所有者(创建者)的文件, 值可以传递user name或者uid。
```



```
demo 1:
$ find . -type f -perm 644

解析:
在当前目录搜索文件权限为644的普通文件。

-------------------------------
demo 2:
$ find . -type f -name "*.php" ! -perm 644

解析:
在当前目录搜索.php的普通文件，并且文件权限不是644的。
注意，可以发现在属性选项(-name, -type, -perm等)前面加上感叹号表示否定操作。

--------------------------------
demo 3:
$ find . -type f -user slynux

解析：
在当前目录搜索文件所有者(创建者)为slynux的普通文件。注意，-user选项的参数可以是user name也可以是user id。
```



* 结合其它命令处理find命令的搜索结果(-exec)

```
-exec算的上find命令最强大的特性之一，借助这个选项find命令可以与其它命令相结合从而处理搜索后的结果。

注意的是，搜索结果的每一项只能被-exec选项后跟着的单个命令处理，需要被多个命令可以考虑将多个命令写进脚本呢，以脚本形式作为单个命令。

demo 1:
$ find . -type f -user root -exec chown slynux {} \;

解析:
假设我们需要找出某位用户(比如root)的所有普通文件，然后将其所有权更改为另一位用户，那么就可以用-user找出root的所有普通文件，然后结合-exec和chown命令更改其所有权。

{}是一个与-exec选项搭配使用的特殊字符，其相当于占位符，会被搜索结果的每一个匹配文件所替换。

比如上面的find的搜索结果是两个文件，test1.txt，test2.txt，那么上面的命令会被解析为:
-exec chown slynux test1.txt \; chown slynux test2.txt \;


-----------------------------------------
demo 2:
$ find . -type f -name "*.c" -exec cat {} \; >all_c_files.txt

解析:
将给定目录中的所有C程序文件拼接起来写入单个文件all_c_files.txt中。注意这里-exec后面跟着多个命令，
搜索结果是先被cat命令处理，所有项处理完后，才将所有结果重定向到文件中，野因此用的是截断方式重定向而不是追加的方式。
-----------------------------------------
demo 3:
$ find . -type f -mtime +10 -name "*.txt" -exec cp {} OLD \;

解析:
将搜索结果中的每一项拷贝到OLD目录中。
---------------------------------------
demo 4:
$ find . -type f -mtime +10 -name "*.txt" -exec ./commands.sh {} \;

解析：
将搜索结果的每一项依次执行脚本文件。

--------------------------------------
demo 5:
$ find . -type f -name "*.txt" -exec printf "Text file: %s\n" {}

解析:
结合printf生成有用的输出信息。


--------------------------------------
demo 6:
$ find devel/source_path \( -name ".git" -prune \) -o \( -type f \) 

解析：
搜索时排除一些子目录(修剪)，可以提高搜索性能。例如搜索git仓库，那么一般.git目录是没有必要搜索的(里面存储的是版本控制信息)。
上面的命令搜索devel/source_path目录，并且排除了.git子目录。
排除的语句块放在第一个位置，类似(\ -name xxxx -prune \); 
实际要执行的过滤器放在第二个位置，类似\( -type f \).
```



### xargs

```
管道操作能够将前一命令的stdout重定向到后一命令的stdin。stdin是数据流，但是有些命令不能接受数据流，只接受命令行参数的形式作为数据. 这时候可以使用xargs先对stdin数据流进行处理。

xargs擅长将stdin数据流转换成命令行参数.(将从stdin接收到的数据重新格式化)

再强调一下，管道相当于重定向，是将原本的stdout 1重定为下一命令的stdin 0，所以前一命令的stderr 2是不会被重定向的。
e.g.,
command | xargs [command]
```

* 单行命令

```
Bash用户都爱单行命令。单行命令是一个命令序列，命令之间不使用定界符分号，而是使用管道操作符进行连接。
单行命令更高效，更简捷地完成任务。

xargs是构建单行命令的重要组件之一。
```

```
demo 1:
$ cat example.txt 

output:
1 2 3 4 5 6
7 8 9 10
11 12

$ cat example.txt | xargs

output:
1 2 3 4 5 6  7 8 9 10 11 12

解析:
将多行输入转换为单行输出。可以看出xargs输出的默认字符串间的定界符是空格, 输出默认为单行输出(所以会将换行符替换为空格)。换行符是多行文本的默认定界符。

-------------------------------
demo 2:
$ cat example.txt | xargs -n 3

output:
1 2 3
4 5 6
7 8 9
10 11 12

解析:
-n <num>选项可以限制xargs命令的输出单行的元素个数。这里指定了单行最多3个。

--------------------------------
demo 3:
$ echo "splitXsplitXsplitXsplit" | xargs -d X

output:
split split split split

解析:
通过-d <sepearator>可以为输入指定一个定制的定界符，注意指定的是输入的定界符，xargs的输出定界符仍然是默认的空格。

思考:
在上面的多行输入转单行中，xargs默认的输入定界符是什么呢？xargs默认的输入定界符是内部字段分隔符(IFS), i.e.,空格。

————————————————————————————————————
demo 4:
$ echo "splitXsplitXsplitXsplit" | xargs -d X -n 2

output:
split split
split split

解析:
通过-d, -n选项相结合控制输出格式。

-----------------------------
demo 5:
(1)
#!/bin/bash
#文件名: cecho.sh
echo $*'#'

(2)
$ ./cecho.sh arg1 arg2

output:
arg1 arg2 #

(3)
$ cat args.txt

output:
arg1
arg2
arg3

(4)
$ cat args.txt|xargs -n 1 ./cecho.sh

output:
arg1 #
arg2 #
arg3 #

(5)
$ cat args.txt|xargs -n 2 ./cecho.sh

output:
arg1 arg2 #
arg3 #

(6)
$ cat args.txt|xargs ./cecho.sh

output:
arg1 arg2 arg3 #

解析:
重点观察(4),(5),(6)，理解xargs是如何将stdin格式化后传递给命令的(这里是cecho.sh脚本).
之所以自定义了一个脚本，是为了更好的观察不同的格式化参数传递给命令造成的影响(脚本执行一次后输出有#结尾)。

可以发现，xargs格式化的参数输出有几行，传递给的对应的脚本(命令)就会执行几次。
上面的(4)相当于这样字，
因为xargs格式化stdin后输出有三行，i.e.,
arg1
arg2
arg3

传递给脚本(命令)后，就相当于:
$ ./cecho.sh arg1
$ ./cecho.sh arg2
$ ./cecho.sh arg3

所以脚本执行了三次，(5),(6)同理。

-----------------------------
demo 6:
$ cat args.txt|xargs -I {} ./cecho.sh -p {} -l

output:
-p arg1 -l #
-p arg2 -l #
-p arg3 -l #

解析:
前面的例子中，脚本cecho.sh(命令)的参数全部来自于args.txt文件，但是有些命令的参数是固定的，比如上面例子中的-p，-l这些选项，那么xargs需要如何传递给命令呢？
xargs有个-I选项，可以定义命令中的占位符，格式化后的参数会依次传递给命令中的占位符，或者说命令中的占位符会被替换成xargs格式化后的参数。
这意味着使用了xargs的-I选项后，每次只能传递一个参数给到命令，有多少个参数，命令就会依次执行多少次(相当于遍历，每一次命令中的{}都会被替换成对应的参数)。

----------------------------
demo 7:
$ find . -type f  -name "*.txt" -print|xargs rm -f

$ find . -type f -name "*.txt" -print0|xargs -0 rm -f

解析:
上面例子中的第一条命令不是很严谨，而且因为xargs后面跟着的是rm -f操作，所以会很危险，可能会误删了一些文件。
因为find命令跟着的是-print选项，也就是以换行符作为输出的定界符，然后由于文件名是可以包含空格的，也就是可能包含空格; xargs命令没有指定输入的定界符，那么默认是IFS，i.e., 空格。那么对于输出中的文件hell test.txt来说，会被当成两个文件hell和test.txt而被删除掉，hell文件属于被误删。

所以，第二条命令进行了优化。find命令指定了\0, i.e., null作为输出定界符。接着xargs通过-0选项指定了输入定界符为\0, 这样就不会导致误删了文件。

所以我们得出结论，find命令和xargs结合使用时，一定要为find命令指定输出定界符\0, 为xargs指定输入定界符，防止歧义。

follow up:
空字符和空格的区别
(1)两者的ascii码值不一样，空字符也就是null的值是0；空格对应的是32.
(2)空字符被用来表示字符串结尾的标志\0, 作为标志性字符，用来表示一个字符串已经结束。

上面例子中有人觉得一个文件名中也有可能包含空字符而导致被误删了？我试了一下，发现创建一个包含空格的文件是很容易的，但是我试着创建一个包含空字符的文件，还没创建成功过。


----------------------------
demo 8:
$ find source_code_dir_path -type f -name "*.c" -print0|xargs -0 wc -l

解析:
统计某个目录下所有c程序文件的行数(LOC, Line of Code)，这个对于程序员来说可能会用到。通过wc命令来统计单词数，行数，字节数等。

follow up:
如果想获得更多的关于源代码更多的统计信息，可以考虑SLOCCount的工具，一般Linux发行版都有包含该软件包。

---------------------------
demo 9:
$ cat files.txt | ( while read arg; do cat $arg; done )

等价于

$ cat files.txt | xargs -I {} cat {}

解析:
上面例子中的第一条命令，通过循环和子shell的方式，可以对同一个参数执行多条命令，而不需要借助管道。子shell操作符内部的多条命令作为一个整体来运行。这个技巧适用于各种问题场景。

注意的是，子shell中的类似cd这种命令，不会改变父shell的当前工作目录。
```



###  tr(转换)

```
tr, i.e., translate。 可以用来对来自stdin的内容进行字符替换，字符删除，重复字符压缩等，在编写单行命令的时候很有用。

man:
translate or delete characters.

注意的是，tr命令的输入只能来自stdin，而无法通过命令行参数的方式。

基本格式:
表示替换时,
command | tr [options]  <set1> <set2>

表示删除时,
command | tr  -d <set1>


表示将stdin的字符从set1映射到set2，而且是一一映射(替换)的方式。set1和set2表示字符集或字符类(本质上都是集合的概念)，由于是一对一的映射，所以理论上set1和set2的集合长度是一样的，这样才能一对一映射(替换)。
当set1长度大于set2时，set2会不断重复其最后一个字符一直到和set1长度一样；当set1长度小于set2时，set2长度超出set1的那部分字符会被忽略，不会有映射。
```

* 定义字符集

```
通过:
'起始字符-终止字符'
可以定义字符集，如果起始字符和终止字符不是一个字符序列，那么其会当作三个元素的集合，i.e.: 起始字符，-，终止字符。当然，无法通过连续字符序列表示时，穷举所有集合的元素也可以。

(1)'A-Z': 表示大写字母A到大写字母Z的序列，共26个字符。
(2)'a-z': 表示小写字母a到小写字母z的序列，共26个字符。
(3)'0-9': 表示数字0到9的字符序列，共10个字符。
(4)'a-z0-9': 表示小写字母a到小写字母z的字符序列，以及数字0到9的字符序列，共36个字符。
(5)'ABD-{': 表示由A,B,D,-,{构成的字符集合，穷举的方式。
```



```
demo 1:
$ echo "HELLO WHO IS THIS" | tr 'A-Z' 'a-z'

output:
hello who is this

解析:
典型的将大写字母转小写。

---------------------------------
demo 2:
$ echo 12345 | tr '0-9' '9876543210'

output:
87654 #已加密


$ echo 87654 | tr '9876543210' '0-9'

output:
12345  #已解密

解析:
通过替换达到类似加密的效果。加密和解密的过程中，set1和set2要互相调换位置。

---------------------------------
demo 3:
$ echo "tr came, tr saw, tr conquered." | tr 'a-zA-Z' 'n-za-mN-ZA-M'

output:
ge pnzr, ge fnj, ge pbadhrerq.  #加密

$ echo "ge pnzr, ge fnj, ge pbadhrerq." | tr 'a-zA-Z' 'n-za-mN-ZA-M'

output:
tr came, tr saw, tr conquered.

解析:
ROT13是著名的加密算法，著名在于set1 set2加密和解密不需要调换位置。

-------------------------------------
demo 4:
$ tr '\t' ' ' < file.txt

解析:
将水平制表符替换为空格，输入还重定向了.

-------------------------------------
demo 5:
$ echo "Hello 123 world 456" | tr -d '0-9'

output:
Hello world

解析:
带-d选项表示将stdin中的数字字符删除。

---------------------------------------
demo 6:
$ echo hello 1 char 2 next 4 | tr -d -c '0-9 \n'

output：
1 2 4

解析:
字符集前带有-c选项表示的补集。i.e., 上面的例子，表示除了数字，空格，换行符外的所有字符都删除掉。


--------------------------------------
demo 7:
$ echo "GNU  is        not UNIX. Recursive    right ?" | tr -s ' '

output:
GNU is not UNIX. Recursive right ?

解析:
通过-s选项，可以压缩连续的重复的字符为单个字符。上面的例子中，将连续的空格压缩为单个空格。


-----------------------------------------
demo 8：
$ cat multi_blank.txt | tr -s '\n'

output:
line1
line2
line3
line4

解析:
压缩多余的换行符。

---------------------------------------------
demo 9:
$ cat sum.txt

output:
1
2
3
4
5

$ cat sum.txt| echo $[ $(tr '\n' '+') 0 ]

output:
15

解析:
将换行符替换成加号+, 末尾补0。通过$[]进行算术运算。
```

* 字符类

```
tr命令内部预定于了一些类(集合)，通过类名可以直接表示字符集的含义。

alnum: 字母和数字。
alpha: 字母
cntrl: 控制(非打印)字符。
digit: 数字
graph: 图形字符
lower: 小写字母
print: 可打印字符
punct: 标点字符
upper: 大写字母
xdigit: 十六进制字符

使用格式；
$ tr [:class:] [:class:]
```

```
demo1:
$ echo abcd | tr [:lower:] [:upper:]

output:
ABCD

解析:
将小写字母映射为大写字母。
```



### 校验和与核实(md5sum/sha1sum)

```
  checksum程序用来从文件中生成校验和密钥，然后利用这个校验和密钥核实文件的完整性。特别是在网络传输过程中(大文件), CD光盘等存储介质的损坏场景中，需要校验文件的完整性。
  如果文件没有损坏或者被修改过，那么两个时间点的文件的校验和应该是一样的。
  最知名和最广泛的校验和技术应该是md5sum和sha1sum，它们使用不同的算法来生成文件的校验和。
```

```
demo 1:
$ md5sum <filename>

output:
f2791474ee93be2b7154299f02303992 <filename>

$ md5sum <filename> 1>file_sum.md5

解析:
可以发现md5sum命令的stdin只有一行信息，包括md5的检验和，还有被校验的文件的路径。
md5sum是一个32个字符的十六位进制串。

-----------------------------------
demo 2:d
$ md5sum file1 file2 file3

output:
checksum1 file1
checksum2 file2
checksum3 file3
checksum4 file4

解析:
可以发现可以传递多个文件名作为命令参数给md5sum，stdin会输出对应的多条记录(checksum+文件名)，所以本质上还是对单个文件生成校验和，也只能对单个文件作为粒度进行校验和。

------------------------------------
demo 3:
$ md5sum -c file_sum.md5

output:
<filename>: OK

解析:
通过-c, i.e.,--check, 可以检查也就是比较文件当前的校验和与之前的校验和是否相等。
注意的是要保证md5文件和需要被校验的源文件的路径是否可达。

-------------------------------------
demo 4:
$ md5sum -c *.md5

解析:
可以一次校验多个文件，本质上还是一个一个文件进行比较。

---------------------------------
demo 5:
对于sha1sum，将上面的md5sum修改成sha1sum即可。


------------------------------------
demo 6:
$ md5deep -r1 directory_path > directory.md5
#-r使用递归的方式
#-1使用相对路径。默认情况下，md5deep会输出文件的绝对路径。

$ find dirctory_path -type f -print0 | xargs -0 md5sum >> directory.md5

解析:
上面例子的目的都是对某个文件夹下的文件生成校验和。
第一个命令需要先安装md5deep或者sha1deep软件包。
第二条命令结合find命令和xargs来进行校验和。
```



### 正则表达式

```
正则表达式是由字面文本和具有特殊意义的符号组成的，是一种用于文本匹配的形式小巧，具有高度针对性的编程语言。
即:
正则表达式 = 字面文本(普通字符) + 特殊意义的符号.

follow up:
(1)不同语言的正则表达式
https://blog.teamtreehouse.com/regular-expressions-10-languages
(2)java中有两层转换，你所需要的正则表达式串是string的输出，所以对于java中的string来说，有些字符也是特殊字符，所以需要两层转换。可以先写出你所需要的正则表达式，然后再看这个正则表达式哪些字符对于java的string来说也是特殊字符，那么就需要再对其进行转义。
tips: java中可以通过在控制台打印，验证String的输出是否符合预期的正则表达式要求。
```

* 特殊意义的符号

```
(1)^: 行起始标记. ^tux匹配以tux起的行.
(2)$: 行尾标记. tux$匹配以tux结尾的行.
(3).: 匹配任意一个字符, 即.表示任意一个字符. Hack.匹配Hackl和Hacki，但是不能匹配Hackl2，Hackil，它只能匹配单个字符。
(4)[]: 匹配包含在[字符]中的任意一个字符，即表示[字符]字符集中的任意一个字符。coo[kl]匹配cook和cool。
(5)[^]: 匹配除[^字符]之外的任意一个字符，即表示除[^字符]外的任意一个字符。9[^01]匹配92，93，不能匹配90或91.
(6)[-]: 匹配[]中指定范围内的任意一个字符，即表示[]范围内的任意一个字符。[1-5]匹配1到5的任意一个数字。
(7)?: 匹配之前的项1次或0次，限定符。colou?r匹配colour或color，不能匹配colouur。
(8)+: 匹配之前的项1次或多次，限定符。 Rollno-9+匹配Rollno-9，Rollno-99，但是不能匹配Rollno-
(9)*: 匹配之前的项0次或多次，限定符。 co*l匹配cl，col，cool等。
(10)(): 创建一个用于匹配的子串。 ma(tri)?x匹配max或matrix。
(11){n}: 匹配之前的项n次，限定符。 [0-9]{3}匹配任意一个三位数，可以扩展成[0-9][0-9][0-9]。
(12){n,}: 匹配之前的项至少n次，限定符。 [0-9]{2,}匹配一个两位或更多位的数。
(13){n,m}: 匹配之前的项最少n次，最多m次，限定符。 [0-9]{2,5}匹配从两位数到五位数之间的任意一个数字。
(14)|: 交替匹配|号两边的任意一项。 Oct (1st|2nd)匹配Oct 1st或Oct 2nd。 
(15)\: 转意符，可以将上面的特殊字符转义。 a\.b匹配a.b，但不能匹配ajb。通过在.前加上转意符, 使特殊字符.变成了普通字符; 也可以使普通字符变成个特殊字符，比如\w。


小结:
可以发现特殊符号可以归为几大类
(1)瞄点，表示位置，不匹配任何字符。比如^表示行首，$表示行尾这类的。
(2)字符集，默认表示匹配字符集里的任意一个字符。比如[], [^], [-]这类，点.号可以看作是特殊的字符集。字符集后面往往紧跟着限定符，没有跟限定符默认表示限定数量为1。
(3)限定符，表示匹配的字符的数量。比如, ?, *, +, 还有比较灵活的自定义的方式{n}, {n,m}, {n,}。限定符其实可以理解为都有最小和最大值，?表示最小是0, 最大是1; *表示最小是0，最大是无穷; +表示最小是1，最大时无穷；{n}表示最小是n，最大也是n; {n,}表示最小是n, 最大是正无穷;...
(4)其它类型的特殊字符，比如()定义匹配的字串，|表示二选一，\表示转义。

我们也可也发现卷括号{}, 方括号[], 圆括号在正则的上下文语义里都是特殊字符。
```

* 可视化正则表达式

```
由于正则表达式有时很难理解，人们容易理解带有图示的事物。
因此出现了将正则表达式可视化的一些工具。
比如:
https://regexper.com/
```

* 处理特殊字符

```
在正则表达式上下文(正则表达式解释器)中，由于预定义了一些特殊字符，那么如果想要匹配这些特殊字符，可以通过转义符，即反斜线对其进行转义，从而将特殊字符变成普通字符。所以，关键在于得知道哪些对于正则引擎来说是预定义的特殊字符。
```

* 贪婪模式&非贪婪模式&独占模式

```
贪婪: 表示次数的量词，默认是贪婪的，默认尽可能多的去匹配(保证整体匹配的情况下), 通过回溯实现。
非贪婪: 量词后面加上?, 找出长度最小且满足匹配的，通过回溯实现。
独占模式: 量词后面加上加号+, 和贪婪模式类似，但是不会进行回溯，会直接匹配失败。

reference:
https://cloud.tencent.com/developer/article/1660522
```



### grep

```
在文件中进行搜索是文本处理的一项重要工作。也许我们会遇到需要在多达上千行的文件中查找所需的数据。
通过grep命令可以提升搜索的效率，其可以接受正则表达式。
默认情况下，会在stdout输出匹配文本所在的行。

在之前的使用中，总是以stdin作为grep的数据源，其实grep更普遍的数据源是文件，以文件名作为参数，还支持多个文件，甚至是目录！这一点需要记得。
```



```
demo 1:
$ grep pattern filename

$ grep "pattern" filename


解析:
搜索某个文件包含特定文本的行。

-----------------------------------
demo 2:
$ echo -e "this is a word\nnext line" | grep word

output:
this is a word

解析:
从stdin读取文本作为grep的参数。

----------------------------------
demo 3:
$ grep "match_text" file1 file2 file3 ...

解析:
对多个文件进行搜索。

--------------------------------
demo 4:
$ grep <word>  <filename> --color=auto

解析:
着重标记匹配到的单词。

------------------------------
demo 5:
$ grep -E "[a-z]+" filename

$ egrep "[a-z]+" filename

解析:
grep只解释match_text中的某些特殊字符(注意是某些，不是所有)，如果要使用正则表达式，需要添加-E选项。这意味着使用扩展(extended)正则表达式。或者可以使用默认允许正则表达式的grep命令-egrep。


-----------------------------
demo 6:
$ echo this is a line.|grep -o "[a-z]+\."

output:
line.

解析:
只输出文件中匹配到的文本部分，而不是所在的整行，可以使用-o选项。

----------------------------
demo 7:
$ grep -v <match_pattern>  <file>

解析:
只输出匹配所在的行之外的所有行，可以通过选项-v进行反转(invert)。

----------------------------
demo 8:
$ grep -c "text" <filename>

$ echo -e "1 2 3 4\nhello\n5 6"|egrep -c "[0-9]"

output:
2

解析:
统计文件中匹配行的数量，注意是匹配文本所在行的数量，并不是匹配的次数。上面的例子中有6个匹配项，但是输出2，因为只有两个匹配行。

------------------------------
demo 9:
$ echo -e "1 2 3 4\nhello\n5 6"|egrep -o "[0-9]"| wc -l

解析:
通过一些技巧可以统计匹配项的数量。

------------------------------
demo 10:
$ cat sample1.txt

output:
gnu is not unix
linux is fun
bash is art

$ cat sample2.txt

output:
planetlinux

$ grep linux -n sample1.txt

output:
1:linux is fun

$ cat sample1.txt | grep linux -n 

$ grep linux -n sample1.txt sample2.txt

output:
sample1.txt:2:linux is fun
sample2.txt:2:planetlinux

解析:
可以发现，通过-n选项可以添加匹配文本所在行的行号。有了行号，可以结合vim跳转到指定行。并且支持多个文件，多文件的情况下会自动在stdout添加所在文件的文件名。 显示行号是真的很方便。

--------------------------------
demo 11:
$ echo gnu is not unix | grep -b -o "not"

output:
7:not

解析:
通过-b选项可以打印匹配文本在所在行的偏移量，偏移量起始值是0，从第一个字符开始计算。
选项-b总是和-o配合使用。

-------------------------------
demo 12:
$ grep -l linux sample1.txt sample2.txt

output:
sample1.txt
sample2.txt

解析:
搜索多个文件并找出匹配文本在哪一个文件中。将这些文件名输出。和-l相反的选项是-L，它输出不匹配的文件列表。

--------------------------------
demo 12:

$ grep text -r -n .

$ cd src_dir

$ grep test_function() -r -n .

output:
./micsutiles/test.c:16:test_function();


解析:
通过-r选项可以递归搜索某个目录。 上面例子中递归搜索当前目录。-R和-r功能一样。-r常和
-n搭配使用。
开发人员使用最多的命令之一！！！可以查找某些文本位于某些文件中。



---------------------------------
demo 13:
$ echo hello world | grep  -i "HELLO"

output:
hello

解析: 通过-i(ignore)选项可以忽略字符的大小写。

--------------------------------
demo 14:
$ grep  -e "pattern1" -e "pattern2"

$ echo this is a line of text | grep -e "this" -e "line" -o 

output:
this
line

解析：
通过-e选项(注意和-E区别)，可以指定多个模式。 
还可以通过指定模式文件的方式达多个模式，在模式文件中逐行写下模式，然后通过-f选项执行。


$ grep -f pattern_file source_file

$ cat pat_file

output:
hello
cool

$ echo hello this is cool | grep -f pat_file

output:
hello this is cool


---------------------------------
demo 15:
$ grep "main()" . -r --include *.{c, cpp}

$ grep "main()" . -r --exclude "README"

解析:
通过--include和--exclude选项可以指定或者排除某些文件，减少搜索范围，从而提高搜索效率。
--exclude-dir可以排除目录。
<some>{string1,string2}会被扩展成<some>string1, <some>string2。所以*.{c, cpp}会被扩展成*.c, *.cpp。


--------------------------------
demo 16:
$ echo test1 > file1.txt

$ echo cool > file2.txt

$ echo test1 > file3.txt

$ grep test1 file* -Z

output:
file1.txttest1
file3.txttest1


$ grep test1 filer* -Zl

output:
file1.txtfile3.txt

$ grep test1 -Zl | xargs -0 rm

解析:
之前介绍过如果stdout是文件列表，通过管道传递给xargs处理时，需要注意文件名包含空格的情况。
grep通过-Z选项可以指定以0字节字符(空字符)作为stdout文件列表的定界符，xargs通过-0选项指明以空字符作为stdin的定界符，从而避免包含空格的文件名被当成两个文件处理的情况。

由于grep默认输出文件名以及匹配文本，所以-Z选项通常结合-l使用，从而只输出文件而不输出匹配文本。

-------------------------------
demo 17:

#!/bin/bash
# 文件名: silent_grep.sh
# 用途: 测试文件是否包含特定的文本内容

if [ $# -ne 2 ]; then
  echo "Usage: $0 <match_text> filename"
  exit 1
fi

match_text=$1
filename=$2

grep -q "$match_text" $filename

if [ $? eq 0 ]; then
  echo "The text exists in the file."
else
  echo "Text does not exist in the file."
fi


$ ./silent_grep.sh student student_data.txt

output:
The text exists in the file.
解析:
该脚本中grep带了-q, 即quiet选项，则grep命令不会有任何输出到stdout。通过$?获取grep命令的返回状态。其实
没有-q选项，有匹配，grep的返回状态为0；无匹配，grep命令的返回状态为1。这个似乎和是否带-q选项无关。

---------------------------------
demo 18:
$ seq 10 | grep 5 -A 3

output:
5
6
7
8

$ seq 10 | grep 5 -B 3

output:
2
3
4
5

$ seq 10 | grep 5 -C 3

output:
2
3
4
5
6
7
8

$ echo -e "a\nb\nc\na\nb\nc" | grep a -A 1

output:
a
b
--
a
b

解析:
基于上下文的打印是grep的特色之一。
通过-A, -B, -C选项可以分别打印出匹配文本以及其后面的n行，匹配文本以及其前面的n行，匹配文本以及其前后面的n行。如果多个匹配，那么会以--作为定界符。

tips: A即After, B即Before, C即Context。
```





### sed(文本替换)

```
sed, i.e., stream editor, 流编辑器的缩写。可以配合正则表达式使用，众所周知的一个用法是进行文本替换，在文本处理中是不可或缺的工具。

man:
stream editor for filtering and transforming text.
```



### 磁盘监视命令(df, du)

```
计算机硬件资源主要包括cpu，内存，磁盘。得牢牢记住这些资源都是有限的，包括磁盘。
随着计算机上运行的软件越多，这些硬件资源会慢慢变得不足从而可能影响软件的正常运行。当发现磁盘资源不足的时候，一般得考虑删除或移走大文件。


```

* du

```
du, 即disk usage的缩写。可以统计文件占用磁盘空间的详情。使用du命令时，当前用户需要有目标文件的读权限。
```

```
demo 1:
$ du filename1 filename2 ...

$ du test-grep.txt

output:
4	test-grep.txt

解析:
参数为具体某个文件，统计结果默认是以字节作为计量单位。

----------------------------
demo 2:
$ du -a directory

$ du -a .

output:
4	./test-grep.txt
4	./grep-test/file3.txt
4	./grep-test/file2.txt
4	./grep-test/file1.txt
16	./grep-test
4	./test-grep-1.txt
28	.

$ du .
16	./grep-test
28	.


解析:
参数为某个目录，带上-a选项，可以显示目录下各个文件(包括目录类型的文件)的磁盘占用详情。
不带-a选项，只会打印目录类型文件的磁盘占用情况。
所以，显示各个文件磁盘占用详情一定得带上-a选项。

-------------------------
demo 3:
$ du -h filename1 filename2 ...

$ du -h test-grep.txt 

output:
4.0K	test-grep.txt

解析:
带上-h选项，可以以标准计量单位KB, MB, GB显示文件占用磁盘情况，可读性更高。
-h一般会和其它选项一起结合使用！为了可读性。

-----------------------
demo 4
$ du -ah directory

$ du -ah .

output:
4.0K	./test-grep.txt
4.0K	./grep-test/file3.txt
4.0K	./grep-test/file2.txt
4.0K	./grep-test/file1.txt
16K	./grep-test
4.0K	./test-grep-1.txt
28K	.


解析:
带上-h选项，可以以标准计量单位KB, MB, GB显示文件占用磁盘情况，可读性更高。

-----------------------
demo 5:
$ du -c filename1 filename2 ...

$ du -c test-grep.txt

output:
4	test-grep.txt
4	total

$ du -ch test-grep.txt 

output:
4.0K	test-grep.txt
4.0K	total

解析:
通过-c选项，可以输出目标文件集合的磁盘占用情况的总计。stdout输出结果末尾多了一行total总计。
c可以理解为count。

------------------------
demo 6:

$ du -c directory

$ du -c .

output:
16	./grep-test
28	.
28	total

$ du -ch .

16K	./grep-test
28K	.
28K	total

解析:
通过-c选项，可以输出目标文件集合的磁盘占用情况的总计。stdout输出结果末尾多了一行total总计。

---------------------
demo 7:
$ du -s filename1

$ du -s directory

$ du -s test-grep.txt 

output:
4	test-grep.txt

$ du -sh test-grep.txt

output:
4.0K	test-grep.txt


$ du -s .

output:
28	.

$ du -sh .

output:
28K	.

解析:
通过-s选项，即summarize，会只是输出目标文件的分别的合计数据。多个文件作为参数输出的是各个文件对应的合计。一般只带单个文件或者单个目录作为参数。


-------------------
demo 8:
$ du [-b| -k | -m | -B <BLOCK_SIZE>] filename1 filename2 ...

$ du [-b| -k | -m | -B <BLOCK_SIZE>] directory

$ du -k test-grep.txt

output:
4	test-grep.txt

解析:
可以通过选项指定特定的单位打印文件磁盘占用情况。
-b: 字节(默认输出)
-k: KB为单位
-m: MB为单位
-B <BLOCK_SIZE>: 以BLOCK_ZISE作为计量单位，BLOCK_SIZE以字节为单位。

注意的是，如果原文件很小，但是指定了一个很大的单位，那么这个统计结果就很粗糙，所以一般得指定和实际大小匹配的单位。

----------------------
demo 9:
$ du -exclude "WILDCARD" directory

$ du -exclude "*.txt" files

$ du --exclude-from <exclude.txt> directory
# exclude.txt包含了需要排除的文件列表。

解析:
统计时可以排除部分文件。

---------------------
demo 10:
$ du --max-depth 2 directory

解析:
通过遍历深度来排除部分文件。


----------------------
demo 11:
$ find . -type f -exec du -k {} \; | sort -nkr 1 | head


解析:
通过上面的命令序列可以统计指定目录中最大的10个文件(不包括目录，只是普通文件类型)。一定要记住这条命令序列，很有用！！！
如果没有find直接du结合sort，由于du命令会把目录类型也输出，所以有干扰。
通过find的-type选项过滤出目标目录普通文件类型的文件，然后再对这些文件执行du命令，并且通过-k指定计量单位统一为KB，方便sort命令进行排序，sort根据第一列也就是磁盘占用大小进行倒排; head在stdout显示前10行。
```



* df

```
df提供磁盘可用空间信息。-f选项会以易读的计量单位为格式打印磁盘空间信息。
```

```
demo 1:
$ df -h

output:
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        486M     0  486M   0% /dev
tmpfs           496M     0  496M   0% /dev/shm
tmpfs           496M   57M  440M  12% /run
tmpfs           496M     0  496M   0% /sys/fs/cgroup
/dev/vda1        24G  2.2G   21G  10% /
tmpfs           100M     0  100M   0% /run/user/0

解析:
第一列也就是Filesystem，类似于windows的c盘，d盘...
```











































































