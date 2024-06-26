---
title: "环境搭建:Hadoop集群环境搭建"
categories:
  - 环境搭建
tags:
  - 环境搭建
  - Hadoop
abbrlink: 7f9fb82d
date: 2024-06-26 08:51:52
---

# Hadoop 集群环境搭建

## 复制和安装 hadoop

1. 将本地`hadoop-3.3.6.tar.gz`复制进`master`容器中`root`目录中

```bash
docker cp /path/to/hadoop-3.3.6.tar.gz master:/root
```

2. 将`hadoop-3.3.6.tar.gz`解压进`/opt/module`目录下

```bash
tar zxvf /root/hadoop-3.3.6.tar.gz -C /opt/module/
```

3. 将`hadoop-3.3.6`改名为`hadoop`

```bash
mv /opt/module/hadoop-3.3.6 /opt/module/hadoop
```

4. 创建存储`tmp` `namenode` `datanode`数据的目录

```bash
mkdir -p /opt/module/hadoop/tmp /opt/module/hadoop/dfs/{name,data}
```

## 配置 hadoop

5. `hadoop`的配置文件都存放在`${HADOOP_HOME}/etc/hadoop`中

```bash
cd /opt/module/hadoop/etc/hadoop/
```

6. 在`hadoop-env.sh`第一行添加以下内容

- 该文件类似 bashrc、profile，用于配置 Hadoop 的环境变量的

```bash
export JAVA_HOME=/opt/module/jdk
export HDFS_NAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export YARN_NODEMANAGER_USER=root
export YARN_RESOURCEMANAGER_USER=root
```

- 指定`JAVA_HOME`路径
- 指定`HDFS_NAMENODE_USER`服务的用户为`root`
- 指定`HDFS_DATANODE_USER`服务的用户为`root`
- 指定`HDFS_SECONDARYNAMENODE_USER`服务的用户为`root`
- 指定`YARN_NODEMANAGER_USER`服务的用户为`root`
- 指定`YARN_RESOURCEMANAGER_USER`服务的用户为`root`

7. 配置 `core-site.xml`

- 该文件是 Hadoop 的核心配置文件，其目的是配置 HDFS 地址、端口号，以及临时文件目录

```xml
<configuration>
<property>
    <name>fs.defaultFS</name>
    <value>hdfs://master:9000</value>
</property>
<property>
    <name>hadoop.tmp.dir</name>
    <value>/opt/module/hadoop/tmp</value>
</property>
</configuration>
```

- `fs.defaultFS`: 文件系统的默认 URI 地址
- `hadoop.tmp.dir`: 确定临时文件的存储路径

8. 配置 `hdfs-site.xml`

- 该文件用于设置 HDFS 的 NameNode 和 DataNode 两大进程。

```xml
<configuration>
<property>
    <name>dfs.replication</name>
    <value>3</value>
</property>
<property>
    <name>dfs.namenode.name.dir</name>
    <value>/opt/module/hadoop/dfs/name</value>
</property>
<property>
    <name>dfs.datanode.data.dir</name>
    <value>/opt/module/hadoop/dfs/data</value>
</property>
</configuration>
```

- `dfs.replication`: 创建文件时指定实际的复制数
- `dfs.namenode.name.dir`: 确定 namenode 在本地文件系统上存储位置
- `dfs.datanode.data.dir`: 确定 datanode 在本地文件系统上存储位置

9. 配置 `mapred-site.xml`

- 该文件是 MapReduce 的核心配置文件，用于指定 MapReduce 运行时框架。

```xml
<configuration>
<property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
</property>
</configuration>
```

- `mapreduce.framework.name`: 用于执行 MapReduce 作业的运行时框架。可以是`local` `classic`或`yarn`。

10. 配置 `yarn-site.xml`

- 本文件是 Yarn 框架的核心配置文件，配置 ResourceManager 和 NodeManager。

```xml
<configuration>
<property>
    <name>yarn.resourcemanager.hostname</name>
    <value>master</value>
</property>
<property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>
</configuration>
```

- `yarn.resourcemanager.hostname`: resourcemanager 的主机名
- `yarn.nodemanager.aux-services`: NodeManager 上运行的附属服务。需配置成 mapreduce_shuffle，才可运行 MapReduce 程序

11. 配置 `workers`

- 本文件是配置所有工作节点的主机名或 IP 地址。

```
master
slave1
slave2
```

## 分发文件

12. 分发`hadoop`到`slave1` `slave2`

```bash
scp -r /opt/module/hadoop/ slave1:/opt/module/
scp -r /opt/module/hadoop/ slave2:/opt/module/
```

## 启动群集

- 在`master`容器中格式化`namenode`（仅需执行一次）

```bash
/opt/module/hadoop/bin/hdfs namenode -format
```

- 启动`HDFS`集群

```bash
/opt/module/hadoop/sbin/start-dfs.sh
```

- 启动`YARN`集群

```bash
/opt/module/hadoop/sbin/start-yarn.sh
```

## 测试集群

13. 查看`HDFS`的根目录

- 输入`hdfs dfs -ls /`指令来查看根目录，一般来说都是空的

```bash
/opt/module/hadoop/bin/hdfs dfs -ls /
```

14. 使用`jps`命令查看`Hadoop`进程：

```bash
jps
```

- master 容器中

```bash
[root@master ~]# jps
6257 SecondaryNameNode
6549 ResourceManager
5990 DataNode
7641 Jps
6718 NodeManager
5823 NameNode
```

- slave1 容器中

```bash
[root@slave1 ~]# jps
165 DataNode
441 Jps
271 NodeManager
```

- slave2 容器中

```bash
[root@slave2 ~]# jps
247 NodeManager
397 Jps
141 DataNode
```

# 参考文章

[Hadoop Cluster Setup](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/ClusterSetup.html)

[Hadoop YARN 配置参数剖析](https://blog.csdn.net/gyqjn/article/details/50846287)

[Hadoop 学习（二） Hadoop 配置文件参数详解](https://www.cnblogs.com/yinghun/p/6230436.html)

[【Hadoop】集群配置之主要配置文件（hadoop-env.sh、yarn-env.sh、core-site.xml、hdfs-site.xml、mapred-site.xml...）](https://blog.csdn.net/m0_60511809/article/details/135254570)
