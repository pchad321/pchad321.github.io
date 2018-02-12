---
title: Linux之awk详解
date: 2018-02-03
modified:
categories:
- Linux
tags:
- Linux
- awk
---

awk是一个强大的文本分析工具，相对于grep的查找，sed的编辑，awk在其对数据分析并生成报告时，显得尤为强大。简单来说awk就是把文件逐行的读入，以空格为默认分隔符将每行切片，切开的部分再进行各种分析处理。在awk中可以定义变量来保存数据，使用算术和字符串操作符来处理数据，使用结构化编程概念来为数据处理增加处理逻辑，以及提取文件的数据并做处理生成报告。
awk有3个不同版本: awk、nawk和gawk，未作特别说明，一般指gawk，gawk是AWK的GNU版本。

#### 命令格式

> awk '{pattern + action}' {filenames}

其中pattern表示AWK在数据中查找的内容，而action是在找到匹配内容时所执行的一系列命令。花括号({})不需要在程序中始终出现，但它们用于根据特定的模式对一系列指令进行分组。pattern就是要表示的正则表达式，用斜杠括起来。

#### 调用方式

awk的调用方式通常分为以下三种：

##### 命令行方式

> awk [-F  field-separator]  'commands'  input-file(s)

其中，commands 是真正awk命令，[-F域分隔符]是可选的。 input-file(s) 是待处理的文件。
在awk中，文件的每一行中，由域分隔符分开的每一项称为一个域。通常，在不指名-F域分隔符的情况下，默认的域分隔符是空格。

##### shell脚本方式

将所有的awk命令插入一个文件，并使awk程序可执行，然后awk命令解释器作为脚本的首行，一遍通过键入脚本名称来调用。

> 相当于shell脚本首行的：#!/bin/sh
  可以换成：#!/bin/awk

##### 将所有的awk命令插入一个单独文件，然后调用

> awk -f awk-script-file input-file(s)

其中，-f选项加载awk-script-file中的awk脚本，input-file(s)跟上面的是一样的。

#### 入门实例

1. 显示最近登录的5个账号名

  ```
  last -n 5 | awk '{print $1}'
  ```
  > awk工作流程是这样的：读入有'\n'换行符分割的一条记录，然后将记录按指定的域分隔符划分域，填充域，$0则表示所有域,$1表示第一个域,$n表示第n个域。默认域分隔符是"空白键" 或 "[tab]键",所以$1表示登录用户，$3表示登录用户ip,以此类推。

2. 只显示/etc/passwd的账户名

  ```
  cat /etc/passwd | awk -F':' '{print $1}'
  ```

  > 这种是awk+action的示例，每行都会执行`action{print $1}`。
    -F指定域分隔符为`:`。

3. 只显示/etc/passwd的账户和账户对应的shell,并且账户与shell之间以tab键分割

  ```
  cat /etc/passwd | awk -F':' '{print $1"\t"$7}'
  ```

4. 只显示/etc/passwd的账户和账户对应的shell,账户与shell之间以逗号分割,并且在所有行前添加列名name,shell,在最后一行添加"blue,/bin/nosh"。

  ```
  cat /etc/passwd | awk -F':' 'BEGIN{print "name,shell"} {print $1","$7} END{print "blue,/bin/nosh"}' 
  ```

  > awk工作流程是这样的：先执行BEGIN，然后读取文件，读入有/n换行符分割的一条记录，然后将记录按指定的域分隔符划分域，填充域，$0则表示所有域,$1表示第一个域,$n表示第n个域,随后开始执行模式所对应的动作action。接着开始读入第二条记录······直到所有的记录都读完，最后执行END操作。

5. 搜索/etc/passwd有root关键字的所有行

  ```
  cat /etc/passwd | awk -F':' '/root/'
  ```

  > 这种是pattern的使用示例，匹配了pattern(这里是root)的行才会执行action(没有指定action，默认输出每行的内容)。
    搜索支持正则，例如找root开头的: `awk -F: '/^root/' /etc/passwd`

6. 获取/etc/passwd有root关键字的所有行，并显示对应的shell

  ```
  cat /etc/passwd | awk -F: '/root/{print $7}'
  ```

#### awk内置变量

awk有许多内置变量用来设置环境信息，这些变量可以被改变，下面给出了最常用的一些变量。

|变量名|说明|
|:---:|:---|
|ARGC|命令行参数个数|
|ARGV|命令行参数排列|
|ARGIND|当前文件在ARGV中的位置|
|ENVIRON|支持队列中系统环境变量的使用|
|ERRNO|当读取或关闭输入文件发生错误时的系统错误号|
|FILENAME|awk浏览的文件名|
|FNR|当前数据文件中的数据行数|
|FS|设置输入域分隔符，等价于命令行 -F选项|
|NF|数据文件中的字段总数|
|NR|已处理的输入记录数|
|OFS|输出域分隔符|
|ORS|输出记录分隔符|
|RS|控制记录分隔符|
|FIELDWIDTHS|由空格分隔的一列数字，定义了每个数据字段确切宽度；一旦设置，会忽略FS变量|

##### 练习

1. 统计/etc/passwd:文件名，每行的行号，每行的列数，对应的完整行内容

  ```
  awk -F':' '{print "filename: " FILENAME ", linenumber: " NR ", columns: " NF ", linecontent: " $0}' /etc/passwd
  ```

#### print和printf
awk中同时提供了print和printf两种打印输出的函数。
其中print函数的参数可以是变量、数值或者字符串。字符串必须用双引号引用，参数用逗号分隔。如果没有逗号，参数就串联在一起而无法区分。这里，逗号的作用与输出文件的分隔符的作用是一样的，只是后者是空格而已。
printf函数，其用法和c语言中printf基本相似,可以格式化字符串,输出复杂时，printf更加好用，代码更易懂。

|控制字母|描述|
|:---:|:---:|
|c|将一个数作为ASCII字符显示|
|d|显示一个整数值|
|i|显示一个整数值(跟d一样)|
|e|用科学计数法显示一个数|
|f|显示一个浮点数|
|g|用科学计数法或浮点数显示(选择较短的格式)|
|o|显示一个八进制值|
|s|显示一个文本字符串|
|x|显示一个十六进制值|
|X|显示一个十六进制值，单用大写字母|

> 可以将上面的例子1改写为以下形式：

```
  awk -F':' '{printf("filename: %10s, linenumber: %s, columns: %s, linecon
tent: %s\n", FILENAME, NR, NF, $0)}' /etc/passwd
```

#### awk编程

##### 变量和赋值

除了awk的内置变量，awk还可以自定义变量。

1. 统计/etc/passwd的账户人数

  ```
  awk 'BEGIN{count=0} {count++} END{print "the number of users is ",count}' /etc/passwd
  ```

2. 统计某个文件夹下的文件占用的字节数

  ```
  ls -l | awk 'BEGIN{size=0} {size+=$5;} END{print "[end]size is ",size}'
  ```

##### 条件语句

awk中的条件语句是从C语言中借鉴来的，声明方式如下：

> if (expression) {
    statement;
    statement;
    ... ...
  }

或

> if (expression) {
    statement;
  } else {
    statement2;
  }

或

> if (expression) {
    statement1;
  } else if (expression1) {
    statement2;
  } else {
    statement3;
  }

1. 统计某个文件夹下的文件占用的字节数,过滤4096大小的文件

  ```
  ls -l | awk 'BEGIN{size=0} {if($5==4096){size+=$5}} END{print "the size of file is ",size}'
  ```

##### 循环语句

awk中的循环语句同样借鉴于C语言，支持while、do/while、for、break、continue，这些关键字的语义和C语言中的语义完全相同。

##### 数组

因为awk中数组的下标可以是数字和字母，数组的下标通常被称为关键字(key)。值和关键字都存储在内部的一张针对key/value应用hash的表格里。由于hash不是顺序存储，因此在显示数组内容时会发现，它们并不是按照你预料的顺序显示出来的。数组和变量一样，都是在使用时自动创建的，awk也同样会自动判断其存储的是数字还是字符串。一般而言，awk中的数组用来从记录中收集信息，可以用于计算总和、统计单词以及跟踪模板被匹配的次数等等。

1. 显示/etc/passwd的账户

  ```
  awk -F':' 'BEGIN{count=0;} {name[count]=$1;count++} END{for(i=0;i<NR;i++){print i, name[i]}}' /etc/passwd
  ```

2. 删除数组变量

  > delete array[index]


#### 练习

1. 取出/etc/passwd中的账户名以a或b开头的行并排序

  ```
  awk '$1~/^(a|b)/{print $0}' /etc/passwd | sort
  ```

  > 这里需要注意`~`是进行对`$1`的模糊匹配的意思。

2. 取出常用服务及其端口号

  ```
  cat /etc/services | awk -F'[ /]+' '$1~/^(ssh|ftp|https)/{print $1,$2}'| uniq
  ```

3. 获取文件中空行的数量

  ```
  awk 'BEGIN{count=0} {if($0==""){count++}} END{print count}' filename
  或
  awk 'BEGIN{count=0} /^$/{count++} END{print count}' filename
  ```

4. 从链接中获取域名

  ```
  awk -F'/' '{array[$3]++}END{for(key in array){print key,array[key]}}' filename
  ```

5. 获取非root用户的信息

  ```
  awk -F":" '$1 !~ /root/{print $1, $NF}' /etc/passwd
  ```
