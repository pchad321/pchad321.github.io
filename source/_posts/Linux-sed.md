---
title: Linux之sed详解
date:
modified:
categories:
- Linux
tags:
- sed
- Linux
---

sed是一种流编辑器，它是文本处理中常营的工具，能够完美的配合正则表达式使用。处理时，把当前处理的行存储在临时缓冲区中，称为“模式空间”（pattern space），接着用sed命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。接着处理下一行，这样不断重复，直到文件末尾。文件内容并没有 改变，除非你使用重定向存储输出。Sed主要用来自动编辑一个或多个文件；简化对文件的反复操作；编写转换程序等。

#### 命令格式
sed [-nefri] ‘command’ 输入文本

#### 常用选项
-n∶使用安静(silent)模式。在一般 sed 的用法中，所有来自STDIN的资料一般都会被列出到屏幕上。但如果加上`-n`参数后，则只有经过sed特殊处理的那一行(或者动作)才会被列出来。
-e∶直接在指令列模式上进行sed的动作编辑；
-f∶直接将sed的动作写在一个档案内，`-f filename`则可以执行 filename 内的sed 动作；
-r∶sed的动作支援的是延伸型正规表示法的语法。(预设是基础正规表示法语法)
-i∶直接修改读取的档案内容，而不是由屏幕输出。 

#### 常用命令
a：新增，a的后面可以接字串，而这些字串会在新的一行出现(目前的下一行)
c：取代，c的后面可以接字串，这些字串可以取代n1,n2之间的行
d：删除，由于是删除，所以d后面通常不接任何符号
i：插入，i的后面可以接字串，而这些字串会在新的一行出现(目前的上一行)
p：打印，即将某个匹配的字符串打印。通常p会与参数 sed -n 一起使用
s：替换，直接将匹配的字符串进行替换。通常与正则表达式一起使用

#### 实例
##### 删除行

```
[root@zyj shell]# sed '1d' filename              #删除第一行 
[root@zyj shell]# sed '$d' filename              #删除最后一行
[root@zyj shell]# sed '1,2d' filename            #删除第一行到第二行
[root@zyj shell]# sed '2,$d' filename            #删除第二行到最后一行
```

##### 显示行

```
[root@zyj shell]# sed -n '1p' filename           #显示第一行 
[root@zyj shell]# sed -n '$p' filename           #显示最后一行
[root@zyj shell]# sed -n '1,2p' filename         #显示第一行到第二行
[root@zyj shell]# sed -n '2,$p' filename         #显示第二行到最后一行
```

##### 查询

```
[root@zyj shell]# sed -n '/centos/p' filename  #查询包括关键字centos所在所有行
[root@zyj shell]# sed -n '/\$/p' filename      #查询包括关键字$所在所有行，使用反斜线\对特殊字符进行转义
```

##### 增加一行或多行字符串

```
[root@zyj shell]# sed '1a drink tea' filename   #第一行后增加字符串"drink tea"
[root@zyj shell]# sed '1,3a drink tea' filename #第一行到第三行后增加字符串"drink tea"
[root@zyj shell]# sed '1a drink\ncoffee' filename #第一行后增加多行，使用换行符\n
```

##### 替换一行或多行

```
[root@zyj shell]# sed '1c Hi' filename          #第一行代替为Hi
[root@zyj shell]# sed '1,2c Hi' filename        #第一行到第二行代替为Hi
```

##### 替换一行中的某部分

> 格式
  sed 's/要替换的字符串/新的字符串/g'   (要替换的字符串可以用正则表达式)

```
[root@zyj shell]# sed -n '/centos/p' filename | sed 's/centos/redhat/g'    #替换centos为redhat
[root@zyj shell]# sed -n '/centos/p' filename | sed 's/centos//g'          #删除centos
```

##### 插入

```
[root@zyj shell]# sed -i '$a bye' filename   #在文件filenamede 最后一行直接输入"bye"
[root@localhost ruby] # sed -n '2,$p' ab        #显示第二行到最后一行
```

#### 习题

1. 从ip addr中找到当前主机的ip

  ```
  [root@zyj shell]# ip addr|sed -nr 's/.*inet (.*)\/24.*$/\1/gp'
  192.168.17.131
  ```

2. 将/etc/passwd第一项和最后一项互换

  ```
  cat /etc/passwd | sed -nr 's/(.*)(:x.*:)(.*$)/\3\2\1/gp'
  ``` 

