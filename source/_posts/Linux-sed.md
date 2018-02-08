---
title: Linux之sed详解
date: 2018-02-03
modified:
categories:
- Linux
tags:
- sed
- Linux
---

sed是一种流编辑器，它是文本处理中常营的工具，能够完美的配合正则表达式使用。处理时，把当前处理的行存储在临时缓冲区中，称为“模式空间”（pattern space），接着用sed命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。接着处理下一行，这样不断重复，直到文件末尾。文件内容并没有 改变，除非你使用重定向存储输出。Sed主要用来自动编辑一个或多个文件；简化对文件的反复操作；编写转换程序等。

### 命令格式
sed [-nefri] ‘command’ 输入文本

### 常用选项
-n∶使用安静(silent)模式。在一般 sed 的用法中，所有来自STDIN的资料一般都会被列出到屏幕上。但如果加上`-n`参数后，则只有经过sed特殊处理的那一行(或者动作)才会被列出来。
-e∶直接在指令列模式上进行sed的动作编辑；或者需要在sed命令行上执行多个命令；例如`sed -e 's/brown/green/; s/dog/cat/' test.txt`；
-f∶直接将sed的动作写在一个档案内，`-f filename`则可以执行 filename 内的sed 动作；
-r∶sed的动作支援的是延伸型正规表示法的语法。(预设是基础正规表示法语法)
-i∶直接修改读取的档案内容，而不是由屏幕输出。 

### 常用命令
a：新增，a的后面可以接字串，而这些字串会在新的一行出现(目前的下一行)
c：取代，c的后面可以接字串，这些字串可以取代n1,n2之间的行
d：删除，由于是删除，所以d后面通常不接任何符号
g：新文本将会替换所有匹配的文本
i：插入，i的后面可以接字串，而这些字串会在新的一行出现(目前的上一行)
l：打印数据流中的文本和不可打印的ASCII字符
p：打印，即将某个匹配的字符串打印。通常p会与参数 sed -n 一起使用
r：将一个独立文件中的数据插入到数据流中
s：替换，直接将匹配的字符串进行替换。通常与正则表达式一起使用
w file：将替换的结果写到文件中

### 使用文本过滤器
格式如下：

> /pattern/command

```
sed '/jimmy/s/bash/csh/' /etc/passwd
```
### sed进阶

#### 多行命令

N：将数据流中的下一行加进来创建一个多行组来处理；
D：删除多行组中的一行；注意：D命令会强制sed编辑器返回到脚本的顶部，而不读取新的行；
P：打印多行组中的一行。

next命令

```
sed '/head/{n ; d}' data.txt		#删除含有head行的下一行
sed '/first/{N ; s/\n/ /}' data.txt     #将含有first的行以及下一行当作一行处理
sed 'N ; s/System.Administrator/Desktop User' data.txt
```

多行删除

```
sed '/^$/{N ; /header/D}' data.txt	#首先找到空白行，然后将下一行合并到模式空间内，由于D只会删除模式空间里的第一行，所以只会删除含有header行前的空白行
```

多行打印

```
sed -n 'N ; /System\nAdministrator/P' data.txt  #只打印多行模式空间中的第一行
```

#### 保持空间

模式空间是一块活跃的缓冲区，在sed编辑器执行命令时它会保存待检查的文本。但它并不是sed编辑器保存文本的唯一空间。

sed编辑器有另一块称作保持空间的缓冲区域。在处理模式空间中的某些行时，可以用保持空间来临时保存一些行。

sed编辑器的保持空间命令：

|命令|描述|
|:---:|:---:|
|h|将模式空间复制到保持空间|
|H|将模式空间附加到保持空间|
|g|将保持空间复制到模式空间|
|G|将保持空间附加到模式空间|
|x|交换模式空间和保持空间的内容|

```
sed -n '/first/{h ; n ; p; g; p}' data.txt	#将含有first的行和后面一行交换顺序输出
```

#### 排除命令

使用感叹号(!)来排除命令，即让原本会起作用的命令不起作用。

```
sed -n '{1!G ; h ; $p}' data.txt
```

#### 改变流

分支命令`b`，格式如下：
[address]b [label]

```
sed '{2,3b; s/This is/Is this/; s/line./test?/}' data.txt	#在第2、3行跳过替换
```

测试命令`t`，根据替换命令的结果跳转到某个标签，而不是根据地址进行跳转，格式如下：
[address]t [label]

#### 模式替代

使用`&`符号用来代表替换命令中的匹配的模式

```
$ echo "The cat sleeps in his hat." | sed 's/.at/"&"/g'
```

sed使用圆括号来定义替换模式中的子模式。

```
$ echo "The System Administrator manual" | sed 's/\(System\) Administrator
/\1 User/g'
```

### 实例

#### 删除行

```
sed '1d' filename              #删除第一行 
sed '$d' filename              #删除最后一行
sed '1,2d' filename            #删除第一行到第二行
sed '2,$d' filename            #删除第二行到最后一行
sed '/1/,/3/d' filename        #在遇到数据行中有1时开始删除，遇到有3的行停止删除
```

#### 显示行

```
sed -n '1p' filename           #显示第一行 
sed -n '$p' filename           #显示最后一行
sed -n '1,2p' filename         #显示第一行到第二行
sed -n '2,$p' filename         #显示第二行到最后一行
```

#### 查询

```
sed -n '/centos/p' filename    #查询包括关键字centos所在所有行
sed -n '/\$/p' filename        #查询包括关键字$所在所有行，使用反斜线\对特殊字符进行转义
```

#### 增加一行或多行字符串

```
sed '1a drink tea' filename     #第一行后增加字符串"drink tea"
sed '1,3a drink tea' filename   #第一行到第三行后增加字符串"drink tea"
sed '1a drink\ncoffee' filename #第一行后增加多行，使用换行符\n
```

#### 替换一行或多行

```
sed '1c Hi' filename          #第一行代替为Hi
sed '1,2c Hi' filename        #第一行到第二行代替为Hi
```

#### 替换一行中的某部分

> 格式
  sed 's/要替换的字符串/新的字符串/g'   (要替换的字符串可以用正则表达式)

```
sed -n '/centos/p' filename | sed 's/centos/redhat/g'    #替换centos为redhat
sed -n '/centos/p' filename | sed 's/centos//g'          #删除centos
sed 's/test/trial/2' data.txt			         #只替换每行第二次出现的test
```

#### 插入

```
sed -i '$a bye' filename   #在文件filenamede 最后一行直接输入"bye"
sed -n '2,$p' filename     #显示第二行到最后一行
```

#### 转换

```
sed 'y/123/789/' test.txt                   将1替换为7,2替换为8,3替换为9
```

### 习题

1. 从ip addr中找到当前主机的ip

  ```
  ip addr|sed -nr 's/.*inet (.*)\/24.*$/\1/gp'
  192.168.17.131
  ```

2. 将/etc/passwd第一项和最后一项互换

  ```
  cat /etc/passwd | sed -nr 's/(.*)(:x.*:)(.*$)/\3\2\1/gp'
  ```

3. 加倍行间距

  ```
  $ sed '$!G' data.txt
  #如果文本中本身就有空白行
  $ sed '/^$/d; $!G' data.txt
  ```

4. 给文件中的行编号

  ```
  $ sed "=" data.txt | sed 'N; s/\n/ /g'
  ```

5. 打印文件末尾的若干行

  ```
  $ sed '{
    :start
    $q; N ; 11,$D
    b start
    }' data.txt
  ```

6. 删除连续的空白行

  ```
  $ sed '/./,/^$/!d' data.txt
  ```

7. 删除开头的空白行

  ```
  $ sed '/./,$!d' data.txt
  ```

8. 删除结尾的空白行

  ```
  $ sed '{
    :start
    /^\n*$/{$d ; N ; b start}
    }' data.txt
  ```

9. 删除HTML标签

  ```
  $ sed 's/<[^>]*>//g ; /^$/d' data.txt
  ```
