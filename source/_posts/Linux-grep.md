---
title: Linux之grep详解
date: 2018-02-03 17:41:42
modified:
categories:
- Linux
tags:
- Linux
- grep
---

grep（global search regular expression(RE) and print out the line）是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹配的行打印出来。

#### 选项

> -a 不要忽略二进制数据。
  -A<显示列数> 除了显示符合范本样式的那一行之外，并显示该行之后的内容。
  -b 在显示符合范本样式的那一行之外，并显示该行之前的内容。
  -c 计算符合范本样式的列数。
  -C<显示列数>或-<显示列数>  除了显示符合范本样式的那一列之外，并显示该列之前后的内容。
  -d<进行动作> 当指定要查找的是目录而非文件时，必须使用这项参数，否则grep命令将回报信息并停止动作。
  -e<范本样式> 指定字符串作为查找文件内容的范本样式。
  -E 将范本样式为延伸的普通表示法来使用，意味着使用能使用扩展正则表达式。
  -f<范本文件> 指定范本文件，其内容有一个或多个范本样式，让grep查找符合范本条件的文件内容，格式为每一列的范本样式。
  -F 将范本样式视为固定字符串的列表。
  -G 将范本样式视为普通的表示法来使用。
  -h 在显示符合范本样式的那一列之前，不标示该列所属的文件名称。
  -H 在显示符合范本样式的那一列之前，标示该列的文件名称。
  -i 忽略字符大小写的差别。
  -l 列出文件内容符合指定的范本样式的文件名称。
  -L 列出文件内容不符合指定的范本样式的文件名称。
  -n 在显示符合范本样式的那一列之前，标示出该列的编号。
  -q 不显示任何信息。
  -R/-r 此参数的效果和指定“-d recurse”参数相同。
  -s 不显示错误信息。
  -v 反转查找。
  -w 只显示全字符合的列。
  -x 只显示全列符合的列。
  -y 此参数效果跟“-i”相同。
  -o 只输出文件中匹配到的部分。

#### 常见用法

在文件中搜索一个单词，命令会返回一个包含“match_pattern”的文本行：

> grep match_pattern file_name
  grep "match_pattern" file_name

在多个文件中查找：

> grep "match_pattern" file_1 file_2 file_3 ...

输出除之外的所有行 -v 选项：

> grep -v "match_pattern" file_name

使用正则表达式 -E 选项：

> grep -E "[1-9]+"
  或
  egrep "[1-9]+"

只输出文件中匹配到的部分 -o 选项：

> echo this is a test line. | grep -o -E "[a-z]+\."
  line.
  echo this is a test line. | egrep -o "[a-z]+\."
  line.

统计文件或者文本中包含匹配字符串的行数 -c 选项：

> grep -c "text" file_name

输出包含匹配字符串的行数 -n 选项：

> grep "text" -n file_name
  或
  cat file_name | grep "text" -n
  #多个文件
  grep "text" -n file_1 file_2

搜索多个文件并查找匹配文本在哪些文件中：

> grep -l "text" file1 file2 file3...


#### 练习

1. 获取root用户的shell程序

  ```
  cat /etc/passwd|grep ^root.*"/root:" | cut -d : -f 7
  ```

2. 显示CentOS7的/etc/grub2.cfg文件中，至少以一个空白 字符开头的且后面存非空白字符的行
 
  ```
  cat /etc/grub2.cfg | egrep "^[[:space:]] [^[:space:]]"
  ```

3. 找出`netstat -tan`命令的结果中以`LISTEN`后跟任意多个空白字符结尾的行

  ```
  netstat -tan | grep ".*LISTEN.*$"
  ```
