---
title: hadoop安装和基本使用
date: 2018-02-07 19:05:43
categories:
- 大数据
tags:
- hadoop
---

### Hadoop安装

#### 创建hadoop用户

```
useradd -m hadoop -s /bin/bash
```

#### 安装java环境

1. 如果是系统自带的java，请先卸载：

  ```
  rpm -qa | grep jdk
  rpm -e --nodeps XXXX		#XXXX是上一条命令的查询结构
  ```

2. 到Oracle官网下载jdk，并安装(这里下载的是.tar.gz版)

  ```
  mkdir /usr/java/
  mv jdk-8u161-linux-x64.tar.gz /usr/java/
  tar zxvf jdk-8u161-linux-x64.tar.gz
  mv jdk1.8.0_161/ jdk1.8
  ```

3. 编辑配置文件，配置环境变量

  ```
  vim /etc/profile

  JAVA_HOME=/usr/java/jdk1.8
  CLASSPATH=$JAVA_HOME/lib/
  PATH=$PATH:$JAVA_HOME/bin
  export PATH JAVA_HOME CLASSPATH

  source /etc/profile
  ```

#### 安装Hadoop2

从hadoop的官网下载已经编译好的(binary)hadoop压缩包，在这里使用的是hadoop2.8.3版本。

```
cd /home/hadoop
tar -zxvf hadoop-2.8.3.tar.gz
mv hadoop-2.8.3 hadoop
```

修改配置文件，添加环境变量

```
su
vim /etc/profile

export HADOOP_HOME=/home/hadoop/hadoop
export PATH=.:$HADOOP_HOME/bin:$HADOOP_HOME/sbin:$PATH

source /etc/profile
```

### Hadoop目录结构说明

```
$ cd /home/hadoop/hadoop
$ ll
drwxr-xr-x. 2 hadoop hadoop  4096 Dec  5 12:28 bin
drwxr-xr-x. 3 hadoop hadoop    19 Dec  5 12:28 etc
drwxr-xr-x. 2 hadoop hadoop   101 Dec  5 12:28 include
drwxr-xr-x. 3 hadoop hadoop    19 Dec  5 12:28 lib
drwxr-xr-x. 2 hadoop hadoop  4096 Dec  5 12:28 libexec
-rw-r--r--. 1 hadoop hadoop 99253 Dec  5 12:28 LICENSE.txt
drwxrwxr-x. 3 hadoop hadoop  4096 Feb 10 11:20 logs
-rw-r--r--. 1 hadoop hadoop 15915 Dec  5 12:28 NOTICE.txt
-rw-r--r--. 1 hadoop hadoop  1366 Dec  5 12:28 README.txt
drwxr-xr-x. 2 hadoop hadoop  4096 Dec  5 12:28 sbin
drwxr-xr-x. 4 hadoop hadoop    29 Dec  5 12:28 share
```

1. bin：Hadoop最基本的管理脚本和使用脚本的目录，这些脚本是sbin目录下管理脚本的基础实现，用户可以直接使用这些脚本管理和使用Hadoop；

2. etc：Hadoop配置文件所在的目录，包括core-site,xml、hdfs-site.xml、mapred-site.xml等从Hadoop1.0继承而来的配置文件和yarn-site.xml等Hadoop2.0新增的配置文件；

3. include：对外提供的编程库头文件（具体动态库和静态库在lib目录中），这些头文件均是用C++定义的，通常用于C++程序访问HDFS或者编写MapReduce程序；

4. lib：该目录包含了Hadoop对外提供的编程动态库和静态库，与include目录中的头文件结合使用；

5. libexec：各个服务对用的shell配置文件所在的目录，可用于配置日志输出、启动参数（比如JVM参数）等基本信息；

6. sbin：Hadoop管理脚本所在的目录，主要包含HDFS和YARN中各类服务的启动/关闭脚本；

7. share：Hadoop各个模块编译后的jar包所在的目录。 

### Hadoop伪集群环境安装

#### 在/home/hadoop/目录下，建立tmp、hdfs/name、hdfs/data目录

```
$ cd /home/hadoop
$ mkdir tmp
$ mkdir hdfs
$ cd hdfs/
$ mkdir data
$ mkdir name
```

#### 修改hadoop的配置文件

1. 配置hadoop-env.sh和yarn-env.sh

  ```
  $ cd /home/hadoop/hadoop/etc/hadoop
  
  $ vim hadoop-env.sh
  #修改export JAVA_HOME
  export JAVA_HOME=/usr/java/jdk1.8
  
  $ vim yarn-env.sh
  #修改export JAVA_HOME
  export JAVA_HOME=/usr/java/jdk1.8 
  ```

2. 配置core-site.xml

  其中fs.defaultFS是NameNode的master节点的URI，hadoop.tmp.dir是namenode上本地的hadoop临时文件夹

  ```
  <configuration>
    <property>
      <name>fs.defaultFS</name>
      <value>hdfs://hadoop1:9000</value>
    </property>

    <property>
      <name>hadoop.tmp.dir</name>
      <value>/home/hadoop/tmp</value>
    </property>
  </configuration>
  ```

3. 配置hdfs-site.xml

  其中dfs.name.dir是namenode上存储hdfs名字空间元数据，dfs.data.dir是namenode上数据块的物理存储位置，dfs.replication是副本的个数，默认为3，一般小于datanode的机器数量，在伪集群配置中设置为1。

  ```
  <configuration>
    <property>
      <name>dfs.name.dir</name>
      <value>/home/hadoop/hdfs/name</value>
    </property>

    <property>
      <name>dfs.data.dir</name>
      <value>/home/hadoop/hdfs/data</value>
    </property>

    <property>
      <name>dfs.replication</name>
      <value>1</value>
    </property>
  </configuration>
  ```

4. 配置mapred-site.xml

  > 注意，默认文件夹中并没有这个文件，而是有一个mapred-site.xml.template，可以将该文件复制并重命名

  ```
  <configuration>
    <property>
      <name>mapreduce.framework.name</name>
      <value>yarn</value>
    </property>
  </configuration>
  ```

5. 配置yarn-site.xml

  ```
  <configuration>
    <property>
      <name>yarn.nodemanager.aux-services</name>
      <value>mapreduce_shuffle</value>
    </property>
    <property>
      <name>yarn.resourcemanager.hostname</name>
      <value>hadoop1</value>
    </property>
  </configuration>
  ```

### Hadoop启动

#### 格式化namenode

```
$ ./bin/hdfs namenode -format
```

#### 启动NameNode和DataNode的守护进程

```
$ ./sbin/start-dfs.sh
```

#### 启动ResourceManager和NodeManager的守护进程

```
$ ./sbin/start-yarn.sh
```

#### 验证启动

执行jps命令

```
$ jps
3994 SecondaryNameNode
3803 DataNode
4156 ResourceManager
4333 Jps
3695 NameNode
4255 NodeManager
```

> SecondaryNameNode:它不是namenode的冗余守护进程，而是提供周期检查点和清理任务。
DataNode:它负责管理连接到节点的存储(一个集群中可以有多个节点)。每个存储数据的节点运行一个datanode守护进程。
ResourceManager:接收客户端任务请求，接收和监控NodeManager(NM)的资源情况汇报，负责资源的分配与调度，启动和监控ApplicationMaster(AM)。
Jps：JDK提供查看当前java进程的小工具。
NameNode:它是Hadoop中的主服务器，管理文件系统名称空间和对集群中存储的文件的访问。
NodeManager:NodeManager(NM)是YARN中每个节点上的代理，它管理Hadoop集群中单个计算节点，包括与ResourceManger保持通信，监督Container的生命周期管理，监控每个Container的资源使用(内存、CPU等)情况，追踪节点健康状况，管理日志和不同应用程序用到的附属服务(auxiliaryservice)。

#### 免密登陆

每次在启动或者停止hadoop是都需要输入密码进行验证，此时可以设置免密登陆：

1. 安装ssh服务

  ```
  $ yum install -y openssh-server openssh-clients 
  ```

2. 进入用户目录，生成密钥

  ```
  $ cd ~
  $ cd .ssh/
  $ ssh-keygen -t rsa (然后一路回车)
  $ cp id_rsa.pub authorized_keys

  $ ssh localhost  #如果此时不提示任何错误，则表明设置成功
  ```

### Hadoop集群安装

#### 安装3台机器

  安装3个虚拟机，主机名分别为hadoop1，hadoop2和hadoop3，对应的ip分别为192.168.17.133,192.168.17.134以及192.168.17.135。

#### 修改机器名

```
hostnamectl set-hostname hadoop1
hostnamectl set-hostname hadoop2
hostnamectl set-hostname hadoop3
```

#### 修改/etc/hosts文件

```
192.168.17.133 hadoop1
192.168.17.134 hadoop2
192.168.17.135 hadoop3
```

#### 免密登陆

```
$ cd ~
$ cd .ssh/
$ ssh-keygen -t rsa (然后一路回车)
$ cp id_rsa.pub authorized_keys 

然后修改authorized_keys文件，将三台机器的文件内容合并，然后复制到每台机器中
```

#### 配置Hadoop

在hadoop1机器中执行以下命令：

```
mkdir /home/hadoop/tmp  
mkdir /home/hadoop/var  
mkdir /home/hadoop/hdfs  
mkdir /home/hadoop/hdfs/name  
mkdir /home/hadoop/hdfs/data
```

1. 修改core-site.xml文件：

  ```
  <configuration>
    <property>
      <name>hadoop.tmp.dir</name>
      <value>/home/hadoop/tmp</value>
      <description>Abase for other temporary directories.</description>
    </property>
    <property>
      <name>fs.default.name</name>
      <value>hdfs://hadoop1:9000</value>
    </property>
  </configuration>
  ```

2. 修改hadoop-env.sh文件：
 
  略

3. 修改hdfs-site.xml文件：

  ```
  <configuration>
    <property>
      <name>dfs.name.dir</name>
      <value>/home/hadoop/hdfs/name</value>
    </property>
    <property>
      <name>dfs.data.dir</name>
      <value>/home/hadoop/hdfs/data</value>
    </property>
    <property>
      <name>dfs.replication</name>
      <value>2</value>
    </property>
    <property>
      <name>dfs.permissions</name>
      <value>false</value>
    </property>
  </configuration>
  ```

4. 修改mapred-site.xml文件：

  ```
  <configuration>
    <property>
      <name>mapred.job.tracker</name>
      <value>hadoop1:49001</value>
    </property>
    <property>
      <name>mapred.local.dir</name>
      <value>/home/hadoop/var</value>
    </property>
    <property>
      <name>mapreduce.framework.name</name>
      <value>yarn</value>
    </property>
  </configuration>
  ```

5. 修改slaves文件：

  > hadoop2
    hadoop3
 
6. 修改yarn-site.xml文件：

  ```
  <configuration>
    <property>
      <name>yarn.resourcemanager.hostname</name>
      <value>hadoop1</value>
    </property>
    <property>
      <description>The address of the applications manager interface in the RM.</description>
      <name>yarn.resourcemanager.address</name>
      <value>${yarn.resourcemanager.hostname}:8032</value>
    </property>
    <property>
      <description>The address of the scheduler interface.</description>
      <name>yarn.resourcemanager.scheduler.address</name>
      <value>${yarn.resourcemanager.hostname}:8030</value>
    </property>
    <property>
      <description>The http address of the RM web application.</description>
      <name>yarn.resourcemanager.webapp.address</name>
      <value>${yarn.resourcemanager.hostname}:8088</value>
    </property>
    <property>
      <description>The https adddress of the RM web application.</description>
      <name>yarn.resourcemanager.webapp.https.address</name>
      <value>${yarn.resourcemanager.hostname}:8090</value>
    </property>
    <property>
      <name>yarn.resourcemanager.resource-tracker.address</name>
      <value>${yarn.resourcemanager.hostname}:8031</value>
    </property>
    <property>
      <description>The address of the RM admin interface.</description>
      <name>yarn.resourcemanager.admin.address</name>
      <value>${yarn.resourcemanager.hostname}:8033</value>
    </property>
    <property>
      <name>yarn.nodemanager.aux-services</name>
      <value>mapreduce_shuffle</value>
    </property>
    <property>
      <name>yarn.scheduler.maximum-allocation-mb</name>
      <value>2048</value>
      <discription>每个节点可用内存,单位MB,默认8182MB</discription>
    </property>
    <property>
      <name>yarn.nodemanager.vmem-pmem-ratio</name>
      <value>2.1</value>
    </property>
    <property>
      <name>yarn.nodemanager.resource.memory-mb</name>
      <value>2048</value>
    </property>
    <property>
      <name>yarn.nodemanager.vmem-check-enabled</name>
      <value>false</value>
    </property>
  </configuration>
  ```

#### 启动hadoop

略

#### 测试hadoop

在浏览器访问  http://192.168.17.133:50070 以及 http://192.168.17.133:8088/

