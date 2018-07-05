---
title: Linux的shell编程详解
date: 2018-07-04 12:00:56
modified:
categories:
- Linux
tags:
- Linux
- Shell
---

### shell历史

Shell的作用是解释执行用户的命令，用户输入一条命令，Shell就解释执行一条，这种方式称为交互式（Interactive），Shell还有一种执行命令的方式称为批处理（Batch），用户事先写一个Shell脚本（Script），其中有很多条命令，让Shell一次把这些命令执行完，而不必一条一条地敲命令。Shell脚本和编程语言很相似，也有变量和流程控制语句，但Shell脚本是解释执行的，不需要编译，Shell程序从脚本中一行一行读取并执行这些命令，相当于一个用户把脚本中的命令一行一行敲到Shell提示符下执行。  

由于历史原因，UNIX系统上有很多种Shell：  
1. sh（Bourne Shell）：由Steve Bourne开发，各种UNIX系统都配有sh。  
2. csh（C Shell）：由Bill Joy开发，随BSD UNIX发布，它的流程控制语句很像C语言，支持很多Bourne Shell所不支持的功能：作业控制，命令历史，命令行编辑。  
3. ksh（Korn Shell）：由David Korn开发，向后兼容sh的功能，并且添加了csh引入的新功能，是目前很多UNIX系统标准配置的Shell，在这些系统上/bin/sh往往是指向/bin/ksh的符号链接。  
4. tcsh（TENEX C Shell）：是csh的增强版本，引入了命令补全等功能，在FreeBSD、Mac OS X等系统上替代了csh。  
5. bash（Bourne Again Shell）：由GNU开发的Shell，主要目标是与POSIX标准保持一致，同时兼顾对sh的兼容，bash从csh和ksh借鉴了很多功能，是各种Linux发行版标准配置的Shell，在Linux系统上/bin/sh往往是指向/bin/bash的符号链接。虽然如此，bash和sh还是有很多不同的，一方面，bash扩展了一些命令和参数，另一方面，bash并不完全和sh兼容，有些行为并不一致，所以bash需要模拟sh的行为：当我们通过sh这个程序名启动bash时，bash可以假装自己是sh，不认扩展的命令，并且行为与sh保持一致。  
6. zsh 的命令补全功能非常强大，可以补齐路径，补齐命令，补齐参数等。  

用户在命令行输入命令后，一般情况下Shell会fork并exec该命令，但是Shell的内建命令例外，执行内建命令相当于调用Shell进程中的一个函数，并不创建新的进程。例如cd、alias、umask、exit等命令都是内建命令，凡是用which命令查不到程序文件所在位置的命令都是内建命令，内建命令没有单独的man手册，要在man手册中查看内建命令：

> $ man bash-builtins

如export、shift、if、eval、[、for、while等等。内建命令虽然不创建新的进程，但也会有Exit Status，通常也用`0`表示成功非零表示失败，虽然内建命令不创建新的进程，但执行结束后也会有一个状态码，也可以用特殊变量`$?`读出。  

### 执行脚本

编写一个简单的脚本test.sh：

```commandline
#! /bin/sh
cd ..
ls
```

Shell脚本中用#表示注释，相当于C语言的//注释。但如果#位于第一行开头，并且是#!（称为Shebang）则例外，它表示该脚本使用后面指定的解释器/bin/sh解释执行。  

Shell会fork一个子进程并调用`exec`执行`./test.sh`这个程序，exec系统调用应该把子进程的代码段替换成./test.sh程序的代码段，并从它的_start开始执行。然而test.sh是个文本文件，根本没有代码段和_start函数，怎么办呢？其实exec还有另外一种机制，如果要执行的是一个文本文件，并且第一行用Shebang指定了解释器，则用解释器程序的代码段替换当前进程，并且从解释器的_start开始执行，而这个文本文件被当作命令行参数传给解释器。  

如果将命令行下输入的命令用()括号括起来，那么也会fork出一个子Shell执行小括号中的命令，一行中可以输入由分号;隔开的多个命令，比如：

```commandline
$ (cd ..;ls -l)
```

### 基本语法

#### 变量

按照惯例，Shell变量由全大写字母加下划线组成，有两种类型的Shell变量：

##### 环境变量

环境变量可以从父进程传给子进程，因此Shell进程的环境变量可以从当前Shell进程传给fork出来的子进程。用printenv命令可以显示当前Shell进程的环境变量。  

##### 本地变量

只存在于当前Shell进程，用set命令可以显示当前Shell进程中定义的所有变量（包括本地变量和环境变量）和函数。  
环境变量是任何进程都有的概念，而本地变量是Shell特有的概念。在Shell中，环境变量和本地变量的定义和用法相似。在Shell中定义或赋值一个变量：  

```commandline
$ VARNAME=value
```

*注意等号两边都不能有空格，否则会被Shell解释成命令和命令行参数。* 

一个变量定义后仅存在于当前Shell进程，它是本地变量，用export命令可以把本地变量导出为环境变量，定义和导出环境变量通常可以一步完成：

```commandline
$ export VARNAME=value
```

用unset命令可以删除已定义的环境变量或本地变量。  

在定义变量时不用`$`，取变量值时要用`$`。和C语言不同的是，Shell变量不需要明确定义类型，事实上Shell变量的值都是字符串，比如我们定义VAR=45，其实VAR的值是字符串45而非整数。Shell变量不需要先定义后使用，如果对一个没有定义的变量取值，则值为空字符串。  

#### 文件名代换（Globbing）：* ? []

这些用于匹配的字符称为通配符（Wildcard），具体如下：  

> \*   匹配0个或多个任意字符  
  ?   匹配一个任意字符  
  [若干字符]  匹配方括号中任意一个字符的一次出现

```commandline
$ ls /dev/ttyS*
$ ls ch0?.doc
$ ls ch0[0-2].doc
$ ls ch[012]   [0-9].doc
```

*注意，Globbing所匹配的文件名是由Shell展开的，也就是说在参数还没传给程序之前已经展开了，比如上述ls ch0[012].doc命令，如果当前目录下有ch00.doc和ch02.doc，则传给ls命令的参数实际上是这两个文件名，而不是一个匹配字符串。*  

#### 命令代换：`或 $()

由\`反引号括起来的也是一条命令，Shell先执行该命令，然后将输出结果立刻代换到当前命令行中。例如定义一个变量存放date命令的输出：  

```commandline
$ DATE=`date`
$ echo $DATE

$ DATE=$(date)
```

#### 算术代换：$(())

用于算术计算，$(())中的Shell变量取值将转换成整数，同样含义的$[]等价例如：  

```commandline
VAR=45
echo $(($VAR+3))
# $(())中只能用+-*/和()运算符，并且只能做整数运算。

# $[base#n],其中base表示进制,n按照base进制解释，后面再有运算数，按十进制解释。

echo $[2#10+11]
echo $[8#10+11]
echo $[10#10+11]
```

#### 转义字符\

和C语言类似，\在Shell中被用作转义字符，用于去除紧跟其后的单个字符的特殊意义（回车除外），换句话说，紧跟其后的字符取字面值。例如：

```commandline
$ echo $SHELL
/bin/bash
$ echo \$SHELL
$SHELL
$ echo \\
\
```

比如创建一个文件名为“$ $”的文件可以这样：

```commandline
$ touch \$\ \$
```

还有一个字符虽然不具有特殊含义，但是要用它做文件名也很麻烦，就是-号。如果要创建一个文件名以-号开头的文件，则可以：

```commandline
$ touch ./-hello
$ touch -- -hello
```

#### 单引号

和C语言不一样，Shell脚本中的单引号和双引号一样都是字符串的界定符，而不是字符的界定符。单引号用于保持引号内所有字符的字面值，即使引号内的\和回车也不例外，但是字符串中不能出现单引号。如果引号没有配对就输入回车，Shell会给出续行提示符，要求用户把引号配上对。  

#### 双引号

被双引号用括住的内容，将被视为单一字串。它防止通配符扩展，但允许变量扩展。这点与单引号的处理方式不同。  

```commandline
$ DATE=$(date)
$ echo "$DATE"
$ echo '$DATE'
```

### Shell脚本语法

#### 条件测试：test [

命令test或[可以测试一个条件是否成立，如果测试结果为真，则该命令的Exit Status为0，如果测试结果为假，则命令的Exit Status为1（注意与C语言的逻辑表示正好相反）。例如测试两个数的大小关系：

```commandline
$ var=2
$ test $var -gt 1
$ echo $?
0
$ test $var -gt 3
$ echo $?
1
$ [ $var -gt 3 ]
$ echo $?
1
```

虽然看起来很奇怪，但左方括号`[`确实是一个命令的名字，传给命令的各参数之间应该用空格隔开，比如，$VAR、-gt、3、]是[命令的四个参数，它们之间必须用空格隔开。命令test或[的参数形式是相同的，只不过test命令不需要]参数。以[命令为例，常见的测试命令如下所示：

> [ -d DIR ]              如果DIR存在并且是一个目录则为真  
  [ -f FILE ]             如果FILE存在且是一个普通文件则为真  
  [ -z STRING ]           如果STRING的长度为零则为真  
  [ -n STRING ]           如果STRING的长度非零则为真  
  [ STRING1 = STRING2 ]   如果两个字符串相同则为真  
  [ STRING1 != STRING2 ]  如果字符串不相同则为真  
  [ ARG1 OP ARG2 ]        ARG1和ARG2应该是整数或者取值为整数的变量，OP是-eq（等于）-ne（不等于）-lt（小于）-le（小于等于）-gt（大于）-ge（大于等于）之中的一个  
  
测试条件之间还可以做与、或、非逻辑运算：

```commandline
[ ! EXPR ]          EXPR可以是上表中的任意一种测试条件，!表示逻辑反
[ EXPR1 -a EXPR2 ]  EXPR1和EXPR2可以是上表中的任意一种测试条件，-a表示逻辑与
[ EXPR1 -o EXPR2 ]  EXPR1和EXPR2可以是上表中的任意一种测试条件，-o表示逻辑或
```

例如：

```commandline
$ VAR=abc
$ [ -d Desktop -a "$VAR" = 'abc' ]
$ echo $?
0
```

*注意，如果上例中的`$VAR`变量事先没有定义，则被Shell展开为空字符串，会造成测试条件的语法错误(展开为[ -d Desktop -a = 'abc' ])，作为一种好的Shell编程习惯，应该总是把变量取值放在双引号之中(展开为[ -d Desktop -a "" = 'abc' ])*  

#### if/then/elif/else/fi

和C语言类似，在Shell中用if、then、elif、else、fi这几条命令实现分支控制。这种流程控制语句本质上也是由若干条Shell命令组成的，例如：  

```commandline
if [ -f ~/.bashrc ]; then
    . ~/.bashrc
fi
```

其实是三条命令，`if [ -f ~/.bashrc ]`是第一条，`then . ~/.bashrc`是第二条，`fi`是第三条。如果两条命令写在同一行则需要用;号隔开，一行只写一条命令就不需要写;号了，另外，then后面有换行，但这条命令没写完，Shell会自动续行，把下一行接在then后面当作一条命令处理。和[命令一样，要注意命令和各参数之间必须用空格隔开。if命令的参数组成一条子命令，如果该子命令的Exit Status为0（表示真），则执行then后面的子命令，如果Exit Status非0（表示假），则执行elif、else或者fi后面的子命令。if后面的子命令通常是测试命令，但也可以是其它命令。Shell脚本没有{}括号，所以用fi表示if语句块的结束。例如：  

```commandline
#! /bin/sh

if [ -f /bin/bash ]
    then echo "/bin/bash is a file"
else 
    echo "/bin/bash is NOT a file"
fi
if :; then echo "always true"; fi
```

`:`是一个特殊的命令，称为空命令，该命令不做任何事，但Exit Status总是真。此外，也可以执行/bin/true或/bin/false得到真或假的Exit Status。例如：  

```commandline
 #! /bin/sh

echo "Is it morning? Please answer yes or no."
read YES_OR_NO
if [ "$YES_OR_NO" = "yes" ]; then
    echo "Good morning!"
elif [ "$YES_OR_NO" = "no" ]; then
    echo "Good afternoon!"
else
    echo "Sorry, $YES_OR_NO not recognized. Enter yes or no."
    exit 1
fi
exit 0
```

上例中的read命令的作用是等待用户输入一行字符串，将该字符串存到一个Shell变量中。  
此外，Shell还提供了&&和||语法，和C语言类似，具有Short-circuit特性。

#### case/esac

case命令可类比C语言的switch/case语句，esac表示case语句块的结束。C语言的case只能匹配整型或字符型常量表达式，而Shell脚本的case可以匹配字符串和Wildcard，每个匹配分支可以有若干条命令，末尾必须以`;;`结束，执行时找到第一个匹配的分支并执行相应的命令，然后直接跳到esac之后，不需要像C语言一样用break跳出。  

```commandline
#! /bin/sh

echo "Is it morning? Please answer yes or no."
read YES_OR_NO
case "$YES_OR_NO" in
    yes|y|Yes|YES)
        echo "Good Morning!";;
    [nN]*)
        echo "Good Afternoon!";;
    *)
        echo "Sorry, $YES_OR_NO not recognized. Enter yes or no."
        exit 1;;
esac
exit 0
```

#### for/do/done

Shell脚本的for循环结构和C语言很不一样，它类似于某些编程语言的foreach循环。例如：

```commandline
#! /bin/sh

for FRUIT in apple banana pear; do
    echo "I like $FRUIT"
done
```

#### while/do/done

while的用法和C语言类似。比如一个验证密码的脚本：

```commandline
#! /bin/sh

echo "Enter password:"
read TRY
while [ "$TRY" != "secret" ]; do
    echo "Sorry, try again"
    read TRY
done
```

下面的例子通过算术运算控制循环的次数：

```commandline
#! /bin/sh

COUNTER=1
while [ "$COUNTER" -lt 10 ]; do
    echo "Here we go again"
    COUNTER=$(($COUNTER+1))
done
```

### 位置参数和特殊变量

常用的位置参数和特殊变量

|参数|含义|
|:----|----|
|$0|  相当于C语言main函数的argv[0]
|$1、$2...|    这些称为位置参数（Positional Parameter），相当于C语言main函数的argv[1]、argv[2]...
|$#|  相当于C语言main函数的argc - 1，注意这里的#后面不表示注释
|$@|  表示参数列表"$1" "$2" ...，例如可以用在for循环中的in后面。
|$*| 表示参数列表"$1" "$2" ...，同上
|$?|  上一条命令的Exit Status
|$$|  当前进程号

位置参数可以用`shift`命令左移。比如shift 3表示原来的$4现在变成$1，原来的$5现在变成$2等等，原来的$1、$2、$3丢弃，$0不移动。不带参数的shift命令相当于shift 1。例如：  

```commandline
#! /bin/sh

echo "The program $0 is now running"
echo "The first parameter is $1"
echo "The second parameter is $2"
echo "The parameter list is $@"
shift
echo "The first parameter is $1"
echo "The second parameter is $2"
echo "The parameter list is $@"
```

### shell输入输出

#### echo

echo显示文本行或变量，或者把字符串输入到文件。

> echo [option] string  
  -e 解析转义字符  
  -n 不回车换行。默认情况echo回显的内容后面跟一个回车换行。  
  echo "hello\n\n"  
  echo -e "hello\n\n"  
  echo  "hello"  
  echo -n "hello"  
  
#### 管道|

可以通过管道把一个命令的输出传递给另一个命令做输入。管道用竖线表示。

#### tee

tee命令把结果输出到标准输出，另一个副本输出到相应文件。

#### 文件重定向

> cmd > file             把标准输出重定向到新文件中  
  cmd >> file            追加  
  cmd > file 2>&1        标准出错也重定向到1所指向的file里  
  cmd >> file 2>&1  
  cmd < file1 > file2    输入输出都定向到文件里  
  cmd < &fd              把文件描述符fd作为标准输入  
  cmd > &fd              把文件描述符fd作为标准输出  
  cmd < &-               关闭标准输入  
  
### 函数

和C语言类似，Shell中也有函数的概念，但是函数定义中没有返回值也没有参数列表。例如：

```commandline
#! /bin/sh

foo(){ echo "Function foo is called";}
echo "-=start=-"
foo
echo "-=end=-"
```

*注意函数体的左花括号'{'和后面的命令之间必须有空格或换行，如果将最后一条命令和右花括号'}'写在同一行，命令末尾必须有;号。*

在定义foo()函数时并不执行函数体中的命令，就像定义变量一样，只是给foo这个名字一个定义，到后面调用foo函数的时候（注意Shell中的函数调用不写括号）才执行函数体中的命令。Shell脚本中的函数必须先定义后调用，一般把函数定义都写在脚本的前面，把函数调用和其它命令写在脚本的最后（类似C语言中的main函数，这才是整个脚本实际开始执行命令的地方）。

Shell函数没有参数列表并不表示不能传参数，事实上，函数就像是迷你脚本，调用函数时可以传任意个参数，在函数内同样是用$0、$1、$2等变量来提取参数，函数中的位置参数相当于函数的局部变量，改变这些变量并不会影响函数外面的$0、$1、$2等变量。函数中可以用return命令返回，如果return后面跟一个数字则表示函数的Exit Status。

下面这个脚本可以一次创建多个目录，各目录名通过命令行参数传入，脚本逐个测试各目录是否存在，如果目录不存在，首先打印信息然后试着创建该目录。

```commandline
#! /bin/sh

is_directory()
{
    DIR_NAME=$1
    if [ ! -d $DIR_NAME ]; then
        return 1
    else
        return 0
    fi
}

for DIR in "$@"; do
    if is_directory "$DIR"
    then :
    else
        echo "$DIR doesn't exist. Creating it now..."
        mkdir $DIR > /dev/null 2>&1
        if [ $? -ne 0 ]; then
            echo "Cannot create directory $DIR"
            exit 1
        fi
    fi
done
```

### Shell脚本的调试方法

Shell提供了一些用于调试脚本的选项，如下所示：

|选项|说明|
|:----|----------|
|-n|读一遍脚本中的命令但不执行，用于检查脚本中的语法错误
|-v|一边执行脚本，一边将执行过的脚本命令打印到标准错误输出
|-x|提供跟踪执行信息，将执行的每一条命令和结果依次打印出来

使用这些选项有三种方法：
1. 在命令行提供参数
   >$ sh -x ./script.sh

2. 在脚本开头提供参数
   >\#! /bin/sh -x

3. 在脚本中用set命令启用或禁用参数
   ```commandline
    #! /bin/sh
    if [ -z "$1" ]; then
      set -x
      echo "ERROR: Insufficient Args."
      exit 1
      set +x
    fi
   ``` 
    
*其中，set -x和set +x分别表示启用和禁用-x参数，这样可以只对脚本中的某一段进行跟踪调试。*  

### 正则表达式

正则表达式：规定一些特殊语法表示字符类、数量限定符和位置关系，然后用这些特殊语法和普通字符一起表示一个模式。  

#### 基本语法

我们知道C的变量和Shell脚本变量的定义和使用方法很不相同，表达能力也不相同，C的变量有各种类型，而Shell脚本变量都是字符串。同样道理，各种工具和编程语言所使用的正则表达式规范的语法并不相同，表达能力也各不相同，有的正则表达式规范引入很多扩展，能表达更复杂的模式，但各种正则表达式规范的基本概念都是相通的。

##### 字符类

|字符|含义|例子|
|:---|---|--------|
.|   匹配任意一个字符|          abc.可以匹配abcd、abc9等
[]|  匹配括号中的任意一个字符|  [abc]d可以匹配ad、bd或cd
-|   在[]括号内表示字符范围|    [0-9a-fA-F]可以匹配一位十六进制数字
^|   位于[]括号内的开头，匹配除括号中的字符之外的任意一个字符|  [^xy]匹配除xy之外的任一字符，因此[^xy]1可以匹配a1、b1但不匹配x1、y1
[[:xxx:]]|   grep工具预定义的一些命名字符类|   [[:alpha:]]匹配一个字母，[[:digit:]]匹配一个数字

##### 数量限定符

|字符|含义|例子|
|:---|---|---------|
?|   紧跟在它前面的单元应匹配零次或一次|    [0-9]?\.[0-9]匹配0.0、2.3、.5等，由于.在正则表达式中是一个特殊字符，所以需要用\转义一下，取字面值
+|   紧跟在它前面的单元应匹配一次或多次|    [a-zA-Z0-9_.-]+@[a-zA-Z0-9_.-]+\.[a-zA-Z0-9_.-]+匹配email地址
*|  紧跟在它前面的单元应匹配零次或多次|    [0-9][0-9]*匹配至少一位数字，等价于[0-9]+，[a-zA-Z_]+[a-zA-Z_0-9]*匹配C语言的标识符
{N}| 紧跟在它前面的单元应精确匹配N次|       [1-9][0-9]{2}匹配从100到999的整数
{N,}|  紧跟在它前面的单元应匹配至少N次|     [1-9][0-9]{2,}匹配三位以上（含三位）的整数
{,M}|  紧跟在它前面的单元应匹配最多M次|     [0-9]{,1}相当于[0-9]?
{N,M}| 紧跟在它前面的单元应匹配至少N次，最多M次|   [0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}匹配IP地址

##### 位置限定符

|字符|含义|例子|
|:---|----|-------------|
|^|   匹配行首的位置|        ^Content匹配位于一行开头的Content
|$|   匹配行末的位置|        ;$匹配位于一行结尾的;号，^$匹配空行
|\\<|  匹配单词开头的位置|    \\<th匹配... this，但不匹配ethernet、tenth
|\\>|  匹配单词结尾的位置|    p\\>匹配leap ...，但不匹配parent、sleepy
|\\b|  匹配单词开头或结尾的位置|     \bat\b匹配... at ...，但不匹配cat、atexit、batch
|\\B|  匹配非单词开头和结尾的位置|   \Bat\B匹配battery，但不匹配... attend、hat ...

##### 其它特殊字符

|字符|含义|例子|
|:---|----|-------|
|\|转义字符，普通字符转义为特殊字符，特殊字符转义为普通字符|   普通字符<写成\\<表示单词开头的位置，特殊字符.写成\\.以及\写成\\\\就当作普通字符来匹配
|()|将正则表达式的一部分括起来组成一个单元，可以对整个单元使用数量限定符|    ([0-9]{1,3}\.){3}[0-9]{1,3}匹配IP地址
|&#124;|连接两个子表达式，表示或的关系|n(o &#124; either)匹配no或neither

### grep

#### 作用

Linux系统中grep命令是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹 配的行打印出来。grep全称是Global Regular Expression Print，表示全局正则表达式版本，它的使用权限是所有用户。

grep家族包括grep、egrep和fgrep。egrep和fgrep的命令只跟grep有很小不同。egrep是grep的扩展，支持更多的re元字符， fgrep就是fixed grep或fast grep，它们把所有的字母都看作单词，也就是说，正则表达式中的元字符表示回其自身的字面意义，不再特殊。linux使用GNU版本的grep。它功能更强，可以通过-G、-E、-F命令行选项来使用egrep和fgrep的功能。

#### 格式

> grep [option]

#### 主要参数

> grep --help  
  [options]主要参数：  
  -c：只输出匹配行的计数。  
  -i：不区分大小写。  
  -h：查询多文件时不显示文件名。  
  -l：查询多文件时只输出包含匹配字符的文件名。  
  -n：显示匹配行及行号。  
  -s：不显示不存在或无匹配文本的错误信息。  
  -v：显示不包含匹配文本的所有行。  
  --color=auto ：可以将找到的关键词部分加上颜色的显示。  
  
#### pattern正则表达式主要参数

> \： 忽略正则表达式中特殊字符的原有含义。  
  ^：匹配正则表达式的开始行。  
  $: 匹配正则表达式的结束行。  
  \\<：从匹配正则表达式的行开始。  
  \\>：到匹配正则表达式的行结束。  
  [ ]：单个字符，如[A]即A符合要求。  
  [ - ]：范围，如[A-Z]，即A、B、C一直到Z都符合要求。  
  .：所有的单个字符。  
  *：有字符，长度可以为0。  
  
#### grep命令使用简单实例

> $ grep 'test' d\*  
  显示所有以d开头的文件中包含test的行。  
  &emsp;  
  $ grep 'test' aa bb cc  
  显示在aa，bb，cc文件中匹配test的行。  
  &emsp;  
  $ grep '[a-z]\{5\}' aa  
  显示所有包含每个字符串至少有5个连续小写字符的字符串的行。  
  &emsp;  
  $ grep 'w\(es\)t.\*\1' aa  
  如果west被匹配，则es就被存储到内存中，并标记为1，然后搜索任意个字符(.\*)，这些字符后面紧跟着另外一个es(\1)，找到就显示该行。如果用egrep或grep -E，就不用”\”号进行转义，直接写成'w(es)t.*\1'就可以了。
  
#### grep命令使用复杂实例

> grep -i pattern files：不区分大小写地搜索。默认情况区分大小写。  
  grep -l pattern files：只列出匹配的文件名。  
  grep -L pattern files：列出不匹配的文件名。  
  grep -w pattern files：只匹配整个单词，而不是字符串的一部分(如匹配’magic’，而不是’magical’)。  
  grep -C number pattern files：匹配的上下文分别显示[number]行。  
  grep pattern1 | pattern2 files：显示匹配pattern1或pattern2的行。  
  例如：grep "abc\|xyz" testfile：表示过滤包含abc或xyz的行  
  grep pattern1 files | grep pattern2：显示既匹配pattern1又匹配pattern2的行。  
  grep -n pattern files：即可显示行号信息
  grep -c pattern files：即可查找总行数
  
用于搜索的特殊符号

> \\< 和 \\> 分别标注单词的开始与结尾。  
  &emsp;  
  例如：  
  grep man \*：匹配'Batman'、'manic'、'man'等；  
  grep '\<man' \* 匹配'manic'和'man'，但不是'Batman'；  
  grep '\<man\>' 只匹配'man'，而不是'Batman'或'manic'等其他的字符串。  
  '^'：指匹配的字符串在行首；  
  '$'：指匹配的字符串在行尾。  
  
### find

#### find命令格式

> find pathname -options [-print -exec -ok ...]

#### find命令参数

|参数|说明|
|:---|---------|
|pathname|查找的目录路径。例如用.来表示当前目录，用/来表示系统根目录，递归查找。
|-print|将匹配的文件输出到标准输出。
|-exec|对匹配的文件执行该参数所给出的shell命令。相应命令的形式为'command' {  } \;，注意{   }和\；之间的空格。
|-ok|和-exec的作用相同，只不过以一种更为安全的模式来执行该参数所给出的shell命令，在执行每一个命令之前，都会给出提示，让用户来确定是否执行。

#### find命令选项

|选项|说明|
|:---|---------|
|-name|按照文件名查找文件|
|-perm|按照文件权限来查找文件|
|-prune|使用这一选项可以使find命令不在当前指定的目录中查找，如果同时使用-depth选项，那么-prune将被find命令忽略|
|-user|按照文件属主来查找文件|
|-group|按照文件所属的组来查找文件|
|-mtime -n +n|按照文件的更改时间来查找文件，-n表示文件更改时间距现在n天以内，+n表示文件更改时间距现在n天以前。find命令还有-atime和-ctime 选项，但它们都和-m time选项|
|-nogroup|查找无有效所属组的文件，即该文件所属的组在/etc/groups中不存在|
|-nouser|查找无有效属主的文件，即该文件的属主在/etc/passwd中不存在|
|-newer file1 ! file2|查找更改时间比文件file1新但比文件file2旧的文件|
|-type|查找某一类型的文件，诸如：b - 块设备文件。d - 目录。c - 字符设备文件。p - 管道文件。l - 符号链接文件。f - 普通文件|
|-size n：[c]|查找文件长度为n块的文件，带有c时表示文件长度以字节计|
|-depth|在查找文件时，首先查找当前目录中的文件，然后再在其子目录中查找|
|-fstype|查找位于某一类型文件系统中的文件，这些文件系统类型通常可以在配置文件/etc/fstab中找到，该配置文件中包含了本系统中有关文件系统的信息|
|-mount|在查找文件时不跨越文件系统mount点|
|-follow|如果find命令遇到符号链接文件，就跟踪至链接所指向的文件|

下面三个的区别：

> -amin n   查找系统中最后N分钟访问的文件  
  -atime n  查找系统中最后n*24小时访问的文件  
  -cmin n   查找系统中最后N分钟被改变文件状态的文件  
  -ctime n  查找系统中最后n*24小时被改变文件状态的文件  
  -mmin n   查找系统中最后N分钟被改变文件数据的文件  
  -mtime n  查找系统中最后n*24小时被改变文件数据的文件  
  
#### 使用exec或ok来执行shell命令

使用find时，只要把想要的操作写在一个文件里，就可以用exec来配合find查找，很方便 在有些操作系统中只允许-exec选项执行诸如ls或ls -l这样的命令。大多数用户使用这一选项是为了查找旧文件并删除它们。建议在真正执行rm命令删除文件之前，最好先用ls命令看一下，确认它们是所要删除的文件。

exec选项后面跟随着所要执行的命令或脚本，然后是一对儿{}，一个空格和一个\\，最后是一个分号。为了使用exec选项，必须要同时使用print选项。如果验证一下find命令，会发现该命令只输出从当前路径起的相对路径及文件名。  

例如：为了用ls -l命令列出所匹配到的文件，可以把ls -l命令放在find命令的-exec选项中：

> $ find . -type f -exec ls -l {} \;

上面的例子中，find命令匹配到了当前目录下的所有普通文件，并在-exec选项中使用ls -l命令将它们列出。

在/logs目录中查找更改时间在5日以前的文件并删除它们：

> $ find logs -type f -mtime +5 -exec rm {} \;

在下面的例子中， find命令在当前目录中查找所有文件名以.LOG结尾、更改时间在5日以上的文件，并删除它们，只不过在删除之前先给出提示：

> $ find . -name "*.conf"  -mtime +5 -ok rm {  } \;

在下面的例子中使用grep命令。find命令首先匹配所有文件名为“ passwd*”的文件，例如passwd、passwd.old、passwd.bak，然后执行grep命令看看在这些文件中是否存在一个root用户：

> \# find /etc -name "passwd*" -exec grep "itcast" {  } \;

### xargs

在使用find命令的-exec选项处理匹配到的文件时， find命令将所有匹配到的文件一起传递给exec执行。但有些系统对能够传递给exec的命令长度有限制，这样在find命令运行几分钟之后，就会出现 溢出错误。错误信息通常是“参数列太长”或“参数列溢出”。这就是xargs命令的用处所在，特别是与find命令一起使用。  

find命令把匹配到的文件传递给xargs命令，而xargs命令每次只获取一部分文件而不是全部，不像-exec选项那样。这样它可以先处理最先获取的一部分文件，然后是下一批，并如此继续下去。  

在有些系统中，使用-exec选项会为处理每一个匹配到的文件而发起一个相应的进程，并非将匹配到的文件全部作为参数一次执行；这样在有些情况下就会出现进程过多，系统性能下降的问题，因而效率不高。  

而使用xargs命令则只有一个进程。另外，在使用xargs命令时，究竟是一次获取所有的参数，还是分批取得参数，以及每一次获取参数的数目都会根据该命令的选项及系统内核中相应的可调参数来确定。  

下面的例子查找系统中的每一个普通文件，然后使用xargs命令来测试它们分别属于哪类文件：

> \# find . -type f -print | xargs file

在当前目录下查找所有用户具有读、写和执行权限的文件，并收回相应的写权限：

> \# ls -l  
  \# find . -perm -7 -print | xargs chmod o-w  
  \# ls -l  
  
用grep命令在所有的普通文件中搜索hello这个词：

> \# find . -type f -print | xargs grep "hello"

用grep命令在当前目录下的所有普通文件中搜索hello这个词：

> \# find . -name \\* -type f -print | xargs grep "hello"

### sed

sed意为流编辑器（Stream Editor），在Shell脚本和Makefile中作为过滤器使用非常普遍，也就是把前一个程序的输出引入sed的输入，经过一系列编辑命令转换为另一种格式输出。sed和vi都源于早期UNIX的ed工具，所以很多sed命令和vi的末行命令是相同的。

#### sed命令行的基本格式

> sed option 'script' file1 file2 ...  
  sed option -f scriptfile file1 file2 ...  
  
#### 选项含义

|选项|说明|
|:---:|---|
|--version|            显示sed版本。
|--help|               显示帮助文档。
|-n,--quiet,--silent|  静默输出，默认情况下，sed程序在所有的脚本指令执行完毕后，将自动打印模式空间中的内容，这些选项可以屏蔽自动打印。
|-e script|            允许多个脚本指令被执行。
|-f script-file, 
--file=script-file|   从文件中读取脚本指令，对编写自动脚本程序来说很棒！
|-i,--in-place|        直接修改源文件，经过脚本指令处理后的内容将被输出至源文件（源文件被修改）慎用！
|-l N, --line-length=N| 该选项指定l指令可以输出的行长度，l指令用于输出非打印字符。
|--posix|             禁用GNU sed扩展功能。
|-r, --regexp-extended|  在脚本指令中使用扩展正则表达式
|-s, --separate|      默认情况下，sed将把命令行指定的多个文件名作为一个长的连续的输入流。而GNU sed则允许把他们当作单独的文件，这样如正则表达式则不进行跨文件匹配。
|-u, --unbuffered|    最低限度的缓存输入与输出

```commandline
# 在输出testfile内容的第二行后添加"hello"
$ sed "2a hello" ./testfile
$ sed "2,5d" testfile
```

#### 常用的sed命令

> /pattern/p  打印匹配pattern的行  
  /pattern/d  删除匹配pattern的行  
  /pattern/s/pattern1/pattern2/   查找符合pattern的行，将该行第一个匹配pattern1的字符串替换为pattern2  
  /pattern/s/pattern1/pattern2/g  查找符合pattern的行，将该行所有匹配pattern1的字符串替换为pattern2  

*使用p命令需要注意，sed是把待处理文件的内容连同处理结果一起输出到标准输出的，因此p命令表示除了把文件内容打印出来之外还额外打印一遍匹配pattern的行。要想只输出处理结果，应加上-n选项。*

例如，testfile.txt文件的内容如下：

> 123  
  abc  
  456  
  
```commandline
$ sed '/abc/p' testfile
123
abc
abc
456

$ sed -n '/abc/p' testfile
abc

$ sed '/abc/d' testfile
123
456

$ sed 's/bc/-&-/' testfile
123
a-bc-
456
# pattern2中的&表示原文件的当前行中与pattern1相匹配的字符串

$ sed 's/\([0-9]\)\([0-9]\)/-\1-~\2~/' testfile
-1-~2~3
abc
-4-~5~6
# pattern2中的\1表示与pattern1的第一个()括号相匹配的内容，\2表示与pattern1的第二个()括号相匹配的内容。sed默认使用Basic正则表达式规范，如果指定了-r选项则使用Extended规范，那么()括号就不必转义了。

$ sed  's/yes/no/;s/static/dhcp/'  ./testfile
# 使用分号隔开指令。

$ sed -e 's/yes/no/' -e 's/static/dhcp/' testfile
# 使用-e选项。
```

如果testfile.txt的内容如下：

> \<html\>\<head\>\<title\>Hello World\</title\>\</head\>
  \<body\>Welcome to the world of regexp!\</body\>\</html\>
  
```commandline
$ sed 's/<[^>]*>//g' testfile
```

### awk

sed以行为单位处理文件，awk比sed强的地方在于不仅能以行为单位还能以列为单位处理文件。awk缺省的行分隔符是换行，缺省的列分隔符是连续的空格和Tab，但是行分隔符和列分隔符都可以自定义，比如/etc/passwd文件的每一行有若干个字段，字段之间以:分隔，就可以重新定义awk的列分隔符为:并以列为单位处理这个文件。

awk的基本形式

> awk option 'script' file1 file2 ...  
  awk option -f scriptfile file1 file2 ...  
  
和sed一样，awk处理的文件既可以由标准输入重定向得到，也可以当命令行参数传入，编辑命令可以直接当命令行参数传入，也可以用-f参数指定一个脚本文件，编辑命令的格式为：

> /pattern/{actions}  
  condition{actions}  

自动变量$1、$2分别表示第一列、第二列等，类似于Shell脚本的位置参数，而$0表示整个当前行。

awk常用的内建变量

|变量名|说明|
|:---:|---|
|FILENAME|当前输入文件的文件名，该变量是只读的
|NR|当前行的行号，该变量是只读的，R代表record
|NF|当前行所拥有的列数，该变量是只读的，F代表field
|OFS|输出格式的列分隔符，缺省是空格
|FS|输入文件的列分融符，缺省是连续的空格和Tab
|ORS|输出格式的行分隔符，缺省是换行符
|RS|输入文件的行分隔符，缺省是换行符

### Linux核心命令

!["Linux核心命令"](/images/spider/Linux-core-command.png)