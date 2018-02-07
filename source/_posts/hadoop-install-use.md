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

### Hadoop配置

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

  其中fs.default.name是HDFS的URI，hadoop.tmp.dir是namenode上本地的hadoop临时文件夹

  ```
  <configuration>
    <property>
      <name>fs.default.name</name>
      <value>hdfs://localhost:9000</value>
    </property>

    <property>
      <name>hadoop.tmp.dir</name>
      <value>/home/hadoop/tmp</value>
    </property>
  </configuration>
  ```

3. 配置hdfs-site.xml

  其中dfs.name.dir是namenode上存储hdfs名字空间元数据，dfs.data.dir是namenode上数据块的物理存储位置，dfs.replication是副本的个数，默认为3，一般小于datanode的机器数量。

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
  </configuration>
  ```

### Hadoop启动

#### 格式化namenode

```
$ bin/hdfs namenode –format
```

#### 启动NameNode和DataNode的守护进程

```
$ sbin/start-dfs.sh
```

#### 启动ResourceManager和NodeManager的守护进程

```
$ sbin/start-yarn.sh
```

#### 验证启动

执行jps命令
