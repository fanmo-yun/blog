---
title: "环境搭建:HBase分布式列式数据库搭建"
abbrlink: c60385f
date: 2024-06-27 10:31:04
categories:
tags:
---

# Hbase 环境搭建

## 复制和安装 hbase

1. 将本地`hbase-2.2.3-bin.tar.gz`复制进`master`容器中`root`目录中

```bash
docker cp /path/to/hbase-2.2.3-bin.tar.gz master:/root
```

2. 将`hbase`解压进`/opt/module`目录下

```bash
tar zxvf /root/hbase-2.2.3-bin.tar.gz -C /opt/module/
```

3. 将解压后的目录改名为`hbase`

```bash
mv /opt/module/hbase-2.2.3-bin /opt/module/hbase
```

## 配置 hbase

1. `hbase`的配置文件都存放在`${HBASE_HOME}/conf`中

```bash
cd /opt/module/hbase/conf
```

2. 配置 `hbase-env.sh`

- 添加以下环境变量

```bash
export JAVA_HOME=/opt/module/jdk
export HBASE_MANAGES_ZK=false
```

- `export HBASE_MANAGES_ZK=false`: 关闭自带的 zookeeper，hbase 自带 zookeeper 单机版，此处搭建的是分布式，因此将此关闭

3. 配置 `hbase-site.xml`

```xml
<configuration>
<property>
    <name>hbase.cluster.distributed</name>
    <value>true</value>
</property>
<property>
    <name>hbase.tmp.dir</name>
    <value>/opt/module/hbase/tmp</value>
</property>
<property>
    <name>hbase.unsafe.stream.capability.enforce</name>
    <value>false</value>
</property>
<property>
    <name>hbase.rootdir</name>
    <value>hdfs://master:9000/HBase</value>
</property>
<property>
    <name>hbase.zookeeper.quorum</name>
    <value>master:2181,slave1:2181,slave2:2181</value>
</property>
<property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/opt/module/zookeeper/data</value>
</property>
</configuration>
```

- `hbase.cluster.distributed`:是否为集群模式
- `hbase.tmp.dir`:指定 HBase 使用的临时文件目录
- `hbase.unsafe.stream.capability.enforce`:HBase 对流能力的某些检查
- `hbase.rootdir`:hbase 存储 hdfs 的路径
- `hbase.zookeeper.quorum`:指定 HBase 使用的 ZooKeeper 集群的地址列表
- `hbase.zookeeper.property.dataDir`:指定 ZooKeeper 存储数据的目录

4. 配置 `regionservers`

```bash
master
slave1
slave2
```

## 分发 `hbase`

```bash
scp -r /opt/module/hbase slave1:/opt/module
scp -r /opt/module/hbase slave2:/opt/module
```

## 启动群集

1. 在 `master` 容器中运行

```bash
/opt/module/hbase/bin/start-hbase.sh
```

## 验证群集

1. 输入 `jps`

```bash
jps
```

2. 进程中显示以下两个, 代表开启成功

```bash
9410 HMaster
9574 HRegionServer
```

3.  进入 `hbase` 列出所有表

- 输入指令

```bash
/opt/module/hbase/bin/hbase shell
```

- 查看命名空间

```bash
list_namespace
```

# 参考文章

[【HBase】Hbase-2.2.3 真分布式搭建](https://www.jianshu.com/p/0bb15c3a2945)
