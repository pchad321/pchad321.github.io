---
title: Zookeeper的安装和使用
date:
modified:
categories:
- Zookeeper
tags:
- Zookeeper
- Linux
---

## jdk1.8的安装
> 由于采用最小化安装，所以centOS中并没有预安装的jdk，故不需要先卸载

```
rpm -qa|grep java
mkdir /usr/java/
mv jdk-8u161-linux-x64.tar.gz /usr/java/
tar zxvf jdk-8u161-linux-x64.tar.gz
mv jdk1.8.0_161/ jdk1.8
```

> 编辑配置文件，配置环境变量

```
vim /etc/profile

JAVA_HOME=/usr/java/jdk1.8
CLASSPATH=$JAVA_HOME/lib/
PATH=$PATH:$JAVA_HOME/bin
export PATH JAVA_HOME CLASSPATH

source /etc/profile
```

## zookeeper安装

```
mkdir /usr/zookeeper/
mv zookeeper-3.4.11.tar.gz /usr/zookeeper/
tar zxvf zookeeper-3.4.11.tar.gz
cd zookeeper-3.4.11
cd conf
cp zoo_sample.cfg zoo.cfg
```

> 新增zookeeper用户以及zookeeper组

```
groupadd zookeeper
useradd -g zookeeper zookeeper
```

> 修改文件夹用户和组

```
chown -R zookeeper zookeeper-3.4.11
chgrp -R zookeeper zookeeper-3.4.11
```

> 新增data和logs文件夹

```
cd /usr/zookeeper/zookeeper-3.4.11
mkdir data
mkdir logs
```

> 修改/usr/zookeeper/zookeeper-3.4.11/conf目录下的zoo.cfg文件

```
# The number of milliseconds of each tick
# zookeeper 定义的基准时间间隔，单位：毫秒
tickTime=2000
# The number of ticks that the initial
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just
# example sakes.
# dataDir=/tmp/zookeeper
 
# 数据文件夹
dataDir=/usr/zookeeper/zookeeper-3.4.11/data
 
# 日志文件夹
dataLogDir=/usr/zookeeper/zookeeper-3.4.11/logs
 
# the port at which the clients will connect
# 客户端访问 zookeeper 的端口号
clientPort=2181
 
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
```

> 修改系统配置文件，添加环境变量

```
vim /etc/profile

JAVA_HOME=/usr/java/jdk1.8
ZOOKEEPER_HOME=/usr/zookeeper/zookeeper-3.4.11
CLASSPATH=$JAVA_HOME/lib/
PATH=$PATH:$JAVA_HOME/bin:$ZOOKEEPER_HOME/bin
export PATH JAVA_HOME CLASSPATH

source /etc/profile
```

### zookeeper常用命令

```
zkServer.sh start
zkServer.sh stop
zkServer.sh status
zkServer.sh restart
```

### 以集群方式启动zookeeper

> 先备份配置文件，然后将配置文件中的注释行去除

```
mv zoo.cfg zoo.cfg.standalone
grep -v "^$" zoo.cfg.standalone | grep -v "^#" > zoo.cfg
```

> 修改配置文件，如下

```
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/usr/zookeeper/zookeeper-3.4.11/data
dataLogDir=/usr/zookeeper/zookeeper-3.4.11/logs
clientPort=2181
server.1=192.168.17.133:2888:3888
server.2=192.168.17.134:2888:3888
server.3=192.168.17.135:2888:3888
```

> 然后在配置的datadir目录下，创建一个名为myid的文件，在该文件的第一行写上一个数字，与配置文件中server.后的数字一直

```
echo 1 > data/myid
echo 2 > data/myid
echo 3 > data/myid
```

> 在启动zookeeper集群前，需要先关闭防火墙

```
systemctl stop firewalld.service
systemctl disable firewalld.service
firewall-cmd --state
```

### zookeeper的可执行脚本

脚本 | 说明
---|---
zkCleanup | 清理Zookeeper的历史数据，包括事务日志和快照数据文件
zkCli | ZooKeeper的一个建议客户端
zkEvn | 设置ZooKeeper的环境变量
zkServer | ZooKeeper服务器的启动、停止和重启脚本
