---
title: Linux常用命令
date: 2018-02-01 20:17:26
modified:
categories:
- Linux
tags:
- Linux
---

### 系统信息

#### 显示机器的处理器架构
> arch
  uname -m

#### 显示正在使用的内核版本
> uname -r

#### 显示CPU的信息
> cat /proc/cpuinfo

#### 显示内存使用
> cat /proc/meminfo

#### 显示内核的版本
> cat /proc/version

#### 显示系统日期
> date

#### 同步时间
> ntpdate time.ntp.org 

### 关机
#### 关闭系统
> shutdown -h now
  init 0

#### 重启
> shutdown -r now
  reboot

#### 注销
> logout

### 文件和目录
#### cd
> /home 进入/home目录
  .. 返回上一级目录
  \- 返回上次所在的目录

#### pwd
显示工作路径

#### ls
> -F 查看目录中的文件 
  -l 显示文件和目录的详细资料
  -a 显示隐藏文件

#### mkdir
创建目录
> -Z：设置安全上下文，当使用SELinux时有效；
  -m<目标属性>或--mode<目标属性>建立目录的同时设置目录的权限；
  -p或--parents 若所要建立目录的上层目录目前尚未建立，则会一并建立上层目录；
  --version 显示版本信息。 

#### rm
> rm -f file1 删除一个叫做 'file1' 的文件' 
  rm dir dir1 删除一个叫做 'dir1' 的目录' 
  rm -rf dir1 删除一个叫做 'dir1' 的目录并同时删除其内容 
  rm -rf dir1 dir2 同时删除两个目录及它们的内容

#### mv
> mv dir1 new_dir 重命名/移动 一个目录 

#### cp
> cp file1 file2 复制一个文件 
  cp dir/* . 复制一个目录下的所有文件到当前工作目录 
  cp -a /tmp/dir1 . 复制一个目录到当前工作目录 
  cp -a dir1 dir2 复制一个目录

#### ln

### 文件搜索
#### find
> find / -name file1 从 '/' 开始进入根文件系统搜索文件和目录 
  find / -user user1 搜索属于用户 'user1' 的文件和目录 
  find /home/user1 -name \*.bin 在目录 '/ home/user1' 中搜索带有'.bin' 结尾的文件 
  find /usr/bin -type f -atime +100 搜索在过去100天内未被使用过的执行文件 
  find /usr/bin -type f -mtime -10 搜索在10天内被创建或者修改过的文件 
  find / -name \*.rpm -exec chmod 755 '{}' \; 搜索以 '.rpm' 结尾的文件并定义其权限 
  find / -xdev -name \*.rpm 搜索以 '.rpm' 结尾的文件，忽略光驱、捷盘等可移动设备

#### locate
> locate \*.ps 寻找以 '.ps' 结尾的文件 - 先运行 'updatedb' 命令 

#### whereis
> whereis halt 显示一个二进制文件、源码或man的位置

#### which
> which halt 显示一个二进制文件或可执行文件的完整路径

### 挂载文件系统
#### mount

#### unmount


### 磁盘空间
#### df
> -a或--all：包含全部的文件系统；
  --block-size=<区块大小>：以指定的区块大小来显示区块数目；
  -h或--human-readable：以可读性较高的方式来显示信息；
  -H或--si：与-h参数相同，但在计算时是以1000 Bytes为换算单位而非1024 Bytes；
  -i或--inodes：显示inode的信息；
  -k或--kilobytes：指定区块大小为1024字节；
  -l或--local：仅显示本地端的文件系统；
  -m或--megabytes：指定区块大小为1048576字节；
  --no-sync：在取得磁盘使用信息前，不要执行sync指令，此为预设值；
  -P或--portability：使用POSIX的输出格式；
  --sync：在取得磁盘使用信息前，先执行sync指令；
  -t<文件系统类型>或--type=<文件系统类型>：仅显示指定文件系统类型的磁盘信息；
  -T或--print-type：显示文件系统的类型；
  -x<文件系统类型>或--exclude-type=<文件系统类型>：不要显示指定文件系统类型的磁盘信息；
  --help：显示帮助；
  --version：显示版本信息。

#### du
> -a或-all 显示目录中个别文件的大小。
  -b或-bytes 显示目录或文件大小时，以byte为单位。
  -c或--total 除了显示个别目录或文件的大小外，同时也显示所有目录或文件的总和。
  -k或--kilobytes 以KB(1024bytes)为单位输出。
  -m或--megabytes 以MB为单位输出。
  -s或--summarize 仅显示总计，只列出最后加总的值。
  -h或--human-readable 以K，M，G为单位，提高信息的可读性。
  -x或--one-file-xystem 以一开始处理时的文件系统为准，若遇上其它不同的文件系统目录则略过。
  -L<符号链接>或--dereference<符号链接> 显示选项中所指定符号链接的源文件大小。
  -S或--separate-dirs 显示个别目录的大小时，并不含其子目录的大小。
  -X<文件>或--exclude-from=<文件> 在<文件>指定目录或文件。
  --exclude=<目录或文件> 略过指定的目录或文件。
  -D或--dereference-args 显示指定符号链接的源文件大小。
  -H或--si 与-h参数相同，但是K，M，G是以1000为换算单位。
  -l或--count-links 重复计算硬件链接的文件。

### 用户和组
#### useradd
创建的新的系统用户。使用useradd指令所建立的帐号，实际上是保存在`/etc/passwd`文本文件中。
> -c<备注>：加上备注文字。备注文字会保存在passwd的备注栏位中；
  -d<登入目录>：指定用户登入时的启始目录；
  -D：变更预设值；
  -e<有效期限>：指定帐号的有效期限；
  -f<缓冲天数>：指定在密码过期后多少天即关闭该帐号；
  -g<群组>：指定用户所属的群组；
  -G<群组>：指定用户所属的附加群组；
  -m：自动建立用户的登入目录；
  -M：不要自动建立用户的登入目录；
  -n：取消建立以用户名称为名的群组；
  -r：建立系统帐号；
  -s<shell>：指定用户登入后所使用的shell；
  -u<uid>：指定用户id。
  
#### userdel
删除给定的用户，以及与用户相关的文件。若不加选项，则仅删除用户帐号，而不删除相关文件。
> -f：强制删除用户，即使用户当前已登录；
  -r：删除用户的同时，删除与用户相关的所有文件。

#### passwd
设置用户的认证信息，包括用户密码、密码过期时间等。
系统管理者则能用它管理系统用户的密码。只有管理者可以指定用户名称，一般用户只能变更自己的密码。
> -d：删除密码，仅有系统管理者才能使用；
  -f：强制执行；
  -k：设置只有在密码过期失效后，方能更新；
  -l：锁住密码；
  -s：列出密码的相关信息，仅有系统管理者才能使用；
  -u：解开已上锁的帐号。

#### groupadd
创建一个新的工作组，新工作组的信息将被添加到系统文件中。
> -g：指定新建工作组的id；
  -r：创建系统工作组，系统工作组的组ID小于500；
  -K：覆盖配置文件“/ect/login.defs”；
  -o：允许添加组ID号不唯一的工作组。

#### groupdel
删除指定的工作组，本命令要修改的系统文件包括/ect/group和/ect/gshadow。若该群组中仍包括某些用户，则必须先删除这些用户后，方能删除群组。

### 文件权限
#### chmod
用来变更文件或目录的权限。在UNIX系统家族里，文件或目录权限的控制分别以读取、写入、执行3种一般权限来区分，另有3种特殊权限可供运用。用户可以使用chmod指令去变更文件与目录的权限，设置方式采用文字或数字代号皆可。符号连接的权限无法变更，如果用户对符号连接修改权限，其改变会作用在被连接的原始文件。

权限范围表示法如下：
`u` User，即文件或目录的拥有者；
`g` Group，即文件或目录的所属群组；
`o` Other，除了文件或目录拥有者或所属群组之外，其他用户皆属于这个范围；
`a` All，即全部的用户，包含拥有者，所属群组以及其他用户；
`r` 读取权限，数字代号为“4”;
`w` 写入权限，数字代号为“2”；
`x` 执行或切换权限，数字代号为“1”；
`-` 不具任何权限，数字代号为“0”；
`s` 特殊功能说明：变更文件或目录的权限。

> -c或——changes：效果类似“-v”参数，但仅回报更改的部分；
  -f或--quiet或——silent：不显示错误信息；
  -R或——recursive：递归处理，将指令目录下的所有文件及子目录一并处理；
  -v或——verbose：显示指令执行过程；
  --reference=<参考文件或目录>：把指定文件或目录的所属群组全部设成和参考文件或目录的所属群组相同；
  <权限范围>+<权限设置>：开启权限范围的文件或目录的该选项权限设置；
  <权限范围>-<权限设置>：关闭权限范围的文件或目录的该选项权限设置；
  <权限范围>=<权限设置>：指定权限范围的文件或目录的该选项权限设置；

#### chown

#### chgrp


### 打包和压缩
#### gzip

#### tar

#### zip

### yum软件包管理
> yum install package_name 下载并安装一个rpm包 
  yum localinstall package_name.rpm 将安装一个rpm包，使用你自己的软件仓库为你解决所有依赖关系 
  yum update package_name.rpm 更新当前系统中所有安装的rpm包 
  yum update package_name 更新一个rpm包 
  yum remove package_name 删除一个rpm包 
  yum list 列出当前系统中安装的所有包 
  yum search package_name 在rpm仓库中搜寻软件包 
  yum clean packages 清理rpm缓存删除下载的包 
  yum clean headers 删除所有头文件 
  yum clean all 删除所有缓存的包和头文件 
