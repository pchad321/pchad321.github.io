---
title: Hadoop配置文件详解
date: 2018-02-11 09:51:44
modified:
categories:
- 大数据
tags:
- Hadoop
- 大数据
---

### 配置文件的层级关系

### core-site.xml文件

|参数|属性值|解释|
|:---:|:---:|:---:|
|fs.defaultFS|NameNode URL|hdfs://host:port/|
|io.file.buffer.size|131072|SequenceFiles文件中.读写缓存size设定|
|hadoop.tmp.dir|/home/hadoop/tmp|hadoop文件系统依赖的基础配置|
|ha.zookeeper.quorum|hostname:port|zookeeper的地址，多个地址以逗号分隔|

```
<configuration>
  <property>
    <name>fs.default.name</name>
    <value>hdfs://hadoop1:9000</value>
    <description>hadoop1为服务器主机名，其实也可以使用IP地址</description>
  </property>
  <property>
    <name>hadoop.tmp.dir</name>
    <value>/home/hadoop/tmp</value>
  </property>
  <property>
    <name>io.file.buffer.size</name>
    <value>131072</value>
    <description>该属性值单位为KB，131072KB即为默认的64M</description>
  </property>
</configuration>
```

### hdfs-site.xml文件

|参数|属性值|解释|
|:---:|:---:|:---:|
|dfs.namenode.name.dir|本地文件系统所在的NameNode的存储空间和持续化处理日志的路径|可以是按逗号分隔的目录列表，fsimage文件会存储在全部目录，冗余安全|
|dfs.blocksize|268435456|大文件系统HDFS块大小为256M，默认值为64M|
|dfs.datanode.data.dir|datanode存放数据块文件的目录|如果这是一个以逗号分隔的目录列表，那么数据将被存储在所有命名的目录，通常在不同的设备|
|dfs.replication|hdfs保存数据的副本数量|副本数目不能大于datanode数目，伪分布式可以将其配置成1|
|dfs.permissions|true/false|是否对远程读写hdfs进行权限检查，可以设置为false，即不检查|
|dfs.nameservices|clustername|使用federation，启动两个HDFS集群，clustername为集群的别名，可以为任意，但不能重复|
|dfs.ha.namenodes.clustername|hostname|指定nameservice是clustername时的namenode有哪些，这里的值是逻辑名称，可以随意，但不能重复|
|dfs.namenode.rpc-address.clustername.hostname|hostname:9000|指定rpc地址|
|dfs.namenode.http-address.clustername.hostname|hostname:50070|指定http地址|
|dfs.namenode.shared.edits.dir|qjournal://hostname:port;hostname:port/clustername|指定clustername的NameNode共享edits文件目录时，使用的JournalNode集群信息|
|dfs.ha.automatic-failover.enabled.cluster1|true/false|cluster1是否启用自动故障恢复，即当NameNode出故障时，是否切换到另一台NameNode|
|dfs.client.failover.proxy.provider.cluster1|org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider|指定cluster1出故障时，哪个实现类负责执行故障切换|
|dfs.ha.fencing.methods|sshfence|NameNode需要进行切换时，使用ssh方式进行操作|
|dfs.ha.fencing.ssh.private-key-files|/home/hadoop/.ssh/id_rsa|如果使用ssh方式，指定密钥存储的位置|

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
    <description>need not permissions</description>
  </property>
</configuration>
```

### slaves文件

从节点的主机名，直接添加即可。

### mapred-site.xml文件

### yarn-site.xml文件
