---
title: Linux的shell编程
date: 2018-02-04 12:00:56
modified:
categories:
- Linux
tags:
- Linux
- Shell
---

### 创建shell脚本文件

在创建shell脚本文件时，必须在文件的第一行指定要使用的shell。其格式为

> #!/bin/bash
  注意：shell只会去处理第一行注释行，并不会处理其他的注释行。

创建shell脚本文件后，给文件添加执行权限

> chmod u+x filename

如果想把脚本中的文本字符串和命令输出显示在同一行中，可以用echo语句的-n参数

> echo -n "The time and date are: "
  date

使用等号将值赋给用户变量时，在变量、等号和值之间不能出现空格。

> testing=\`date\`
  testing=$(date)

引用一个变量值时需要使用$，而引用变量来对其进行赋值时则不要使用$：

> value1=10
  value2=$value1
  \# 注意：没有使用$，shell会将变量名解释成普通的文本字符串。

#### 命令替换

有两种方法可以将命令输出赋给变量：

1. 反引号字符\`\`

2. $\(\)格式

  ```
  testing=`date`
  testing=$(date)
  echo "The date and time are: " $testing

  today=$(date +%y%m%d)
  ls /usr/bin -al > log.$today
  ```

#### 重定向

重定向包括输出重定向`>  >>`以及输入重定向`<  <<`

> 内联输入重定向

#### 执行数学运算

##### expr命令

```
expr 1 + 5
6
```

##### 使用方括号

如果需要将一个数学运算结果赋给某个变量，可以用美元符和方括号：

```
var1=100
var2=50
var3=30
var4=$[$var1 * ($var3 - $var2)]
```

> 需要注意的是，bash shell数学运算符只支持整数运算。

##### 浮点解决方案

在脚本中使用bc，基本格式如下：

> variable=$(echo "options; expression" | bc)

```
var1=20
var2=3.14159
var3=$(echo "scale=4; $var1 * var2" | bc)
```

```
var1=10.46
var2=43.67
var3=33.2
var4=71

var5=$(bc << EOF
scale = 4
a1 = ($var1 * $var2)
b1 = ($var3 * $var4)
a1 + b1
EOF
)

echo $var5
```

#### 退出脚本

退出状态码的查看`$?`

|状态码|描述|
|:---:|:---:|
|0|命令成功结束|
|1|通用未知错误|
|2|误用Shell命令|
|126|命令不可执行|
|127|没找到命令|
|128|无效退出参数|
|128+x|Linux信号x的严重错误|
|130|命令通过Ctrl+C控制码越界|
|255|退出码越界|

exit命令


### 结构化命令

#### if-then

```
if command; then
  commands
fi
```

```
if pwd; then
  echo "It worked"
fi
```

#### if-then-else

```
if command
then 
  commands
else
  commands
fi
```

#### 嵌套if

```
if command1
then
  commands
else
  if command2
  then
    more commands
  else
    more commands
  fi
fi
```

```
if command1
then
  commands
elif command2
then
  more commands
fi
```

#### test命令

如果test命令中列出的条件成立，test命令就会退出并返回退出状态码0。

> test condition

或者可以用在`if-then`语句中

```
if test condition
then
  commands
fi
```

也可以使用方括号进行测试

```
if [ condition ]
then
  commands
fi
```

##### 数值比较

|比较|描述|
|:---:|:---:|
|n1 -eq n2|检查n1是否等于n2|
|n1 -ge n2|检查n1是否大于等于n2|
|n1 -gt n2|检查n1是否大于n2|
|n1 -le n2|检查n1是否小于等于n2|
|n1 -lt n2|检查n1是否小于n2|
|n1 -ne n2|检查n1是否不等于n2|

##### 字符串比较

|比较|描述|
|:---:|:---:|
|str1 = str2|检查字符串是否相同|
|str1 != str2|检查字符串是否不同|
|str1 < str2|检查str1是否比str2大|
|str1 \> str2|检查str1是否比str2小|
|-n str1|检查str1的长度是否非0|
|-z str1|检查str1的长度是否为0|

> 需要注意的是，大于号和小于号在使用时需要转义

##### 文件比较

|比较|描述|
|:---:|:---:|
|-e file|如果file存在，则为真|
|-d file|如果file存在并为目录，则为真|
|-f file|如果file为常规文件，则为真|
|-r file|如果file存在并可读，则为真|
|-s file|如果file存在并非空，则为真|
|-w file|如果file存在并可写，则为真|
|-x file|如果file存在并可执行，则为真|
|-O file|如果file存在并属于当前用户，则为真|
|-G file|如果file存在并且默认组与当前用户相同，则为真|
|file1 -nt file2|如果file1比file2新，则为真|
|file1 -ot file2|如果file1比file2旧，则为真|

##### 复合条件测试

if-then可以用`&&`和`||`来组合测试

#### if-then的高级特性

##### 使用双括号

双括号命令允许你在比较过程中使用高级数学表达式。其格式如下：
> ((expression))

双括号命令符号包括以下：
val++，val--，++val，--val，!，~，**(幂运算)，<<，>>，&，|，&&，||

##### 使用双方括号

双方括号命令提供了针对字符串比较的高级特性。其格式如下：
> [[expression]]

```
if [[ $USER == r* ]]
```

#### case命令

```
case variable in
pattern1 | pattern2) commands1;;
pattern3) commands2;;
*) default commands;;
esac
```

#### for命令

```
for var in list
do
  commands
done
```

for循环假定每个值都是用空格分割的。
可以使用内部字段分隔符`IFS`来自定义分隔符，例如

> IFS=$'\n'
  IFS=$'\n':;"

##### C语言分隔的for命令

命令格式：
> for (( variable assignment ; condition ; iteration process ))

#### while命令

while命令的基本格式：

```
while test command
do
  other commands
done
```

#### until命令

until命令和while命令正好相反，只有测试命令的退出状态码不为0，bash shell才会执行循环中列出的命令。

命令格式：

```
until test commands
do
  other commands
done
```

#### 循环处理文件数据

需要使用嵌套循环以及修改IFS环境变量

```
IFS.OLD=$IFS
IFS=$'\n'
for entry in $(cat /etc/passwd)
do
  echo "Value in $entry"
  IFS=:
  for value in $entry
  do
    if [ -n "$value" ]
    then
      echo "$value"
    fi
  done
done
```

#### 控制循环

##### break命令

可以用break退出任意的循环，包括while和until循环。
也可以使用`break -n`跳出外部循环，默认为1。

##### continue命令

continue命令可以提前终止某次循环中的命令，但并不会完全终止整个循环。
也可以使用`continue -n`继续执行哪一层循环。

#### 处理循环的输出

可以对循环的输出使用管道或重定向。通过在done命令之后添加一个处理命令来实现。

> done > output.txt

#### 例子

创建多个用户账户

```
input="user.csv"
while IFS="," read -r userid name
do
  echo "adding $userid"
  useradd -c "$name" -m "userid" 
done < "$input"
```

### 处理用户输入

#### 命令行参数

向shell脚本传递数据的最基本方法是使用命令行参数。
命令行参数允许在运行脚本时向命令行添加数据。

```
./test.sh 10 abc
```

bash shell会将一些位置参数的特殊变量分配给输入到命令行中的所有参数。这也包括shell所执行的脚本
命令。其中：`$0`是程序名，`$1`是第一个参数，`$2`是第二个参数，一直到`$9`。

在使用时，需要用空格将每个命令行参数分隔开，例如:

```
count=$[ $1 * $2 ]
```

注意，当命令行参数中出现空格时，需要使用引号：

```
./test1.sh 'hello world'
```

如果命令行参数的数量超过9个时，从第十个参数开始，需要使用花括号，例如`${10}`、`${11}`等。

使用`basename`命令可以返回不包含路径的脚本名，例如：

```
echo "$0"
echo "$(basename $0)"
```

在使用命令行参数的脚本中，需要先检测其中是否存在数据：

```
if [ -n "$1" ]
```

#### 特殊的参数变量

特殊变量`$#`含有脚本运行时携带的命令行参数的个数，例如：

```
echo There are $# parameters supplied.
```

使用$&#123;!#&#125;可以返回最后一个命令行参数的值，如果命令行参数的个数为0，则返回当前脚本的名称。

使用`$*`和`$@`可以访问所有的参数。其中`$*`变量会将命令行上提供的所有参数当作一个单词保存。这个单词包括了命令行中出现的每一个参数值；`$@`变量会将命令行上所有参数当作同一个字符串中的多个独立的单词，然后通过遍历获取所有的参数值。

```
count=1
for param in "$*"
do
  echo "\$* parameter #count = $param"
  count=$[ $count + 1 ]
done
echo
count=1
for param in "$@"
do
  echo "\$@ parameter #count = $param"
  count=$[ $count + 1 ]
done
```

#### 移动变量

bash shell的`shift`命令可以用来操作命令行参数，shift命令会根据命令行参数的相对位置来移动。
在默认情况下，会将每个参数变量向左移动一个位置。其中$2的值会移动到$1，而$1的值则会被删除，$0不会改变。

```
while [ -n "$1" ]
do
  echo "$1"
  shift
done
```

也可以一次性移动多个位置

> shift n

#### 处理选项

选项是跟在单破折号后面的单个字母，可以用来改变命令的行为。
可以使用`--`来表明选项列表结束。

##### getopt命令

getopt命令能够识别命令行参数，从而在脚本中解析更加方便。

命令格式：

> getopt optstring parameters
  其中optstring定义了命令行有效的选项字母，还定义了哪些选项字母需要参数值。

```
getopt ab:cd -a -b test1 -cd test2 test3
-a -b test1 -c -d -- test2 test3
```

> 如果指定了一个不再optstring中的选项，默认情况下，getopt命令会产生一条错误消息。可以在命令后加-q忽略。

在脚本中使用getopt

> set -- $(getopt -q ab:cd "$@")

##### getopts命令

命令格式如下：

> getopts optstring variable

如果选项需要跟一个参数值，`OPTARG`环境变量就会保存这个值。`OPTIND`环境变量保存了参数列表中getopts正在处理的参数位置。

#### 获取用户输入

使用`read`命令获取用户输入。

read命令从标准输入或另一个文件描述符接收输入。在收到输入后，read命令会将数据放进一个变量。

```
read -p "Enter your name: " first last
echo $last, $first
```

如果在read命令行中不指定变量，那么read命令会将它收到的任何数据都放进特殊环境变量`REPLY`中：

```
read -p "Enter your name: "
echo $REPLY
```

> 在使用read命令时，需要注意超时时间，可以使用-t选项来指定计时器

```
read -t 5 -p "Please enter your name: " name
```

如果需要隐藏输入的值时，可以使用`-s`选项。
read在读取文件时，每次从文件中读取一行文本，当文件中没有内容时，read命令将会退出并返回非零退出状态码。

```
cat test | while read line
```

### 呈现数据

|文件描述符|缩写|描述|
|:---:|:---:|:---:|
|0|STDIN|标准输入|
|1|STDOUT|标准输出|
|2|STDERR|标准错误|

默认情况下，STDERR文件描述符和STDOUT文件描述符指向同样的地方。

只重定向错误：

> ls -al errorfile 2> test

重定向错误和数据：

> ls -al test.txt errorfile 2>test 1>test1

同时重定向STDERR和STDOUT：

> ls -al test.txt errorfile &>test2

如果需要重定向到某个文件描述符时，必须在文件描述符之前加一个`&`：

```
echo "This is an error" >&2
```

可以使用`exec`命令进行永久重定向：

```
exec 1> testout
exec 0< testin
```

关闭文件描述符：

```
exec 3>&-
```

如果需要阻止命令输出，可以将STDERR重定向到`/dev/null`。

也可以在/tmp目录中创建临时文件或文件夹。使用`mktemp`在本地目录创建临时文件。创建时，只需要指定一个临时模板：

```
#在当前目录创建临时文件，并返回文件名
mktemp testing.xxxxxx
#在系统临时目录创建临时文件，并返回全路径
mktemp -t tmp.xxxxxx
#在当前目录创建临时文件夹，并返回文件夹名
mktemp -d dir.xxxxxx
```

使用`tee`命令可以将输出同时发送到显示器和日志文件中

> date | tee testfile
  \#将文件进行追加，使用-a选项
  date | tee -a testfile

#### 实例

```
#!/bin/bash

outputfile="person.sql"
IFS=','

while read name age address
do
  cat >> $outputfile << EOF
  insert into person(name,age,address) values('$name','$age','$address');
  EOF
done < $1
```

> ./test.sh testfile

### 脚本控制

#### 处理信号

常用的Linux信号如下：

|信号|值|描述|
|:---:|:---:|:---:|
|1|SIGHUP|挂起进程|
|2|SIGINT|终止进程|
|3|SIGQUIT|停止进程|
|9|SIGKILL|无条件终止进程|
|15|SIGTERM|尽可能终止进程|
|17|SIGSTOP|无条件停止进程，但不是终止进程|
|18|SIGTSTP|停止或暂停进程，但不终止进程|
|19|SIGCONT|继续运行停止的进程|

`Ctrl+Z`组合键会生成一个`SIGTSTP`信号，停止shell中运行的任何进程。停止进程会让程序继续保留在内存中，并能从上次停止的位置继续运行。

使用`exit`退出停止的作业，使用`ps -l`查看已停止的作业。

可以使用`trap`命令来捕获信号，其命令格式为：

> trap command signals

或者在trap命令后加上`EXIT`信号来捕获脚本的退出：

```
trap "echo end..." EXIT
```

也可以重新使用带有选项的trap命令在脚本中的不同位置进行不同的捕获处理。

#### 以后台模式运行脚本

直接在命令后面添加`&`即可：

> ./test.sh &

注意：在ps命令的输出中，每一个后台进程都和终端会话终端联系在一起。如果终端会话退出，那么后台进程也会随之退出。

此时可以使用`nohup`命令来阻止后台进程在终端退出时也一起终止：

> nohup ./test.sh &

#### 作业控制

作业控制：启动、停止终止以及恢复作业的功能。

使用`jobs`命令查看shell当前正在处理的作业。jobs命令参数如下：

|参数|描述|
|:---:|:---:|
|-l|列出进程的PID以及作业号|
|-n|只列出上次shell发出的通知后改变了状态的作业|
|-p|只列出作业的PID|
|-r|只列出运行中的作业|
|-s|只列出已停止的作业|

如果要以后台模式重启一个作业，可以用`bg`命令加上作业号：

> bg 2

如果要以前台模式重启一个作业，可以用`fg`命令加上作业号：

> fg 2

#### 调整优先级

调度优先级：内核分配给进程的CPU时间。
调度优先级是一个整数值，从-20(最高优先级)到+19(最低优先级)。默认情况下以优先级0来启动所有进程。

使用`nice`命令设置命令启动时的调度优先级，例如：

> nice -n 10 ./test.sh > test.out &

使用`renice`命令改变运行中进程的PID，例如：

> renice -n 10 -p 8888

#### 定时运行作业

使用`at`命令来计划执行作业。at命令会将作业提交到队列中，指定shell何时运行该作业。
at命令的格式如下：

> at [-f filename] time

当作业在运行时，Linux系统会将提交该作业的用户的电子邮件地址作为STDOUT和STDERR。所以需要在脚本中对输出进行重定向。

使用`atq`命令查看系统中有哪些作业在等待。

使用`atrm`命令来删除等待中的作业。

##### 使用cron程序

Linux系统使用cron程序来安排要定期执行的作业。cron程序会在后台运行并检查一个特殊的表，以获知已安排执行的作业。

cron时间表的格式如下：

> min hour dayofmonth month dayof week command

如果想在每天下午15:20执行某个文件，可以设置为：

> 20 15 \* \* \* \* /home/zyj/shell/tesh.sh > test.out

使用crontab命令来处理cron时间表。

### 函数

#### 基本函数

创建函数

```
function name {
    commands
}
```

#### 在函数中使用变量

向函数传递参数
在脚本中指定函数时，必须将参数和函数放在同一行，例如：

> func1 $value1 10

```
#!/bin/bash

function add {
  if [ $# -eq 0] || [ $# -gt 2]
  then
    echo -1
  elif [ $# -eq 1 ]
  then
    echo $[ $1 + $1 ]
  else 
    echo $[ $1 + $2 ]
  fi
}

value1=$(add 10 15)
value2=$(add 10)
value3=$(add)
value4=$(add 5 10 15)
echo $value1 $value2 $value3 $value4
```

#### 全局变量和局部变量

使用`local`关键字定义局部变量。

> local temp

local关键字保证了变量只局限在该函数中。如果脚本中在该函数之外有同样名字的变量，那么shell将会保持这两个变量的值是分离的。

#### 数组变量和函数

向函数传递数组参数

```
#!/bin/bash

function testArr {
    local newArr
    newArr=("$@")
    echo "The new array is ${newArr[*]}"
}

myArr=(1 2 3 4 5)
testArr ${myArr[*]}
```

从函数返回数组

#### 函数递归

计算阶乘

```
#!/bin/bash

function factorial {
  if [ $1 -eq 1 ]
  then
    echo 1
  else
    local temp=$[ $1 - 1 ]
    local result=$(factorial $temp)
    echo $[ $result * $1 ]
  fi
}

read -p "Enter a number:" value
result=$(factorial $value)
echo "The result is: " $result
```


