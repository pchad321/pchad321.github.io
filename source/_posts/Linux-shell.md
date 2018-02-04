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
#!/bin/bash

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
#!/bin/bash

input="user.csv"
while IFS="," read -r userid name
do
  echo "adding $userid"
  useradd -c "$name" -m "userid" 
done < "$input"
```
