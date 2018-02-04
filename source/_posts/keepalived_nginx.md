---
title: Nginx+Keepalived的安装和使用
date: 2018-01-31
modified: 
categories:
- Linux
tags:
- Nginx
- Keepalived
- Linux
---

## vim安装

```
yum install vim -y
```

## 修改ip
修改`/etc/sysconfig/network-scripts/ifcfg-eth0`的内容(如果该文件不存在，先新建)

```
DEVICE=eth0 #网卡对应的设备别名
BOOTPROTO=static #设置为静态IP，static
ONBOOT=yes
DNS1=114.114.114.114
HWADDR=00:07:E9:05:E8:B4 #对应的网卡物理地址，可以不用配置
IPADDR=192.168.17.133 #ip地址静态指定
NETMASK=255.255.255.0 
GATEWAY=192.168.17.2
```
然后在虚拟机->设置->网络适配器中选择自定义VMnet8

> 可以将原有的`ifcfg-enoxxxxxxx`文件删除，然后修改`grub`文件
vim /etc/default/grub
然后在`GRUB_CMDLINE_LINUX`原有的参数后面加上`"net.ifnames=0 biosdevname=0"`，保存退出后，执行
`grub2-mkconfig -o /boot/grub2/grub.cfg`
然后重启计算机

## 添加用户和权限

```
$ useradd apache

$ passwd apache

$ visudo 

$ /root

$ yyp
```

## 安装apache
```
yum install httpd httpd-devel

#开启apache服务
systemctl start httpd.service

#关闭apache服务
systemctl stop httpd.service

#设置为开机启动
systemctl enable httpd.service

#关闭防火墙
systemctl stop firewalld.service

#开机不启动防火墙
systemctl disable firewalld.service

#apache默认使用/var/www/html文件夹里的资源，故可以创建一个index.html文件
echo apache133 > index.html
```

## 安装Nginx

### 使用压缩包安装nginx

1. 切换为root用户

2. 进入/usr/local/src目录下

3. 下载Nginx、openssl、pcre以及zlib

  ```
  wget http://nginx.org/download/nginx-1.12.2.tar.gz
  wget https://www.openssl.org/source/openssl-fips-2.0.16.tar.gz
  wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.40.tar.gz
  wget http://www.zlib.net/zlib-1.2.11.tar.gz
  ```

4. 可能需要先安装c++编译环境

  ```
  yum install gcc-c++
  ```

5. 安装openssl

  ```
  tar zxvf openssl-fips-2.0.16.tar.gz 
  cd openssl-fips-2.0.16/
  ./config
  make && make install
  ```

6. 安装pcre

  ```
  tar zxvf pcre-8.40.tar.gz 
  cd pcre-8.40/
  ./configure
  make && make install
  ```

7. 安装zlib

  ```
  tar zxvf zlib-1.2.11.tar.gz 
  cd zlib-1.2.11/
  ./configure
  make && make install
  ```

8. 安装Nginx

  ```
  tar zxvf nginx-1.12.2.tar.gz 
  cd nginx-1.12.2/
  ./configure
  make && make install
  ```

9. 启动Nginx

  ```
  /usr/local/nginx/sbin/nginx
  ```

10. Nginx基本操作

  ```
  #启动
  [root@localhost ~]# /usr/local/nginx/sbin/nginx
  #停止/重启
  [root@localhost ~]# /usr/local/nginx/sbin/nginx -s stop(quit、reload)
  #命令帮助
  [root@localhost ~]# /usr/local/nginx/sbin/nginx -h
  #验证配置文件
  [root@localhost ~]# /usr/local/nginx/sbin/nginx -t
  #配置文件
  [root@localhost ~]# vim /usr/local/nginx/conf/nginx.conf
  ```

11. 获取Nginx的安装位置

  ```
  whereis nginx
  ```

12. 修改Nginx的配置文件

  ```
  [root@localhost conf]# cd /usr/local/nginx/conf/
  [root@localhost conf]# vim nginx.conf
  ```

13. 在http内添加

  ```
  upstream test.com{
    server 192.168.17.133:80 weight=5;
    server 192.168.17.134:80 weight=1;
  }
  ```

14. 在http->server中的location中添加

  ```
  location / {
    proxy_pass  http://test.com;
    proxy_set_header Host       $host;
    proxy_set_header X-Real-IP  $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    root   html;
    index  index.html index.htm;
  }
  ```

15. 安装rz和sz

  ```
  yum install lrzsz
  ```

### 使用yum安装nginx(推荐)

> 本地yum源中没有我们想要的nginx，那么就需要创建一个`/etc/yum.repos.d/nginx.repo`的文件，新增一个yum源。

```
vim /etc/yum.repos.d/nginx.repo
```

> 然后加入如下内容

```
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1
```

> 查询nginx的相关信息并安装

```
yum list | grep nginx
yum install -y nginx
```


## 安装Keepalived
### 安装Keepalived
```
wget http://www.keepalived.org/software/keepalived-1.2.24.tar.gz
tar -zxvf keepalived-1.2.24.tar.gz
cd keepalived-1.2.24
./configure --prefix=/usr/local/keepalived
make && make install
```

> 在安装时可能会出现如下错误\
!!! OpenSSL is not properly installed on your system. !!!\
!!! Can not include OpenSSL headers files.\
解决办法：yum -y install openssl-devel

> 也可以直接使用yum安装

```
yum install keepalived
```

### 配置Keepalived
keepalived启动时会从/etc/keepalived目录下查找keepalived.conf配置文件，如果没有找到则使用默认的配置。/etc/keepalived目录安装时默认是没有安装的，需要手动创建。

> 先备份配置文件

```
cp keepalived.conf keepalived.conf.bak
```

> `在启动keepalived前先关闭防火墙和selinux`

```
systemctl stop firewalld.service
systemctl disable firewalld.service

getenforce
/usr/sbin/sestatus -v
#临时关闭
setenforce 0
#永久关闭
vim /etc/selinux/config
#将SELINUX=enforcing改为SELINUX=disabled，必须重启才能生效
```
