---
title: "环境搭建:Flume环境搭建"
categories:
  - 环境搭建
tags:
  - 环境搭建
  - Flume
abbrlink: 77a30426
date: 2024-06-26 16:58:21
---

# Flume 环境搭建

## 复制和安装 flume

1. 将本地`flume`复制进`master`容器中`root`目录中

```bash
docker cp /path/to/apache-flume-1.9.0-bin.tar.gz master:/root
```

2. 将`flume`解压进`/opt/module`目录下

```bash
tar zxvf /root/apache-flume-1.9.0-bin.tar.gz -C /opt/module/
```

3. 将解压后的目录改名为`flume`

```bash
mv /opt/module/apache-flume-1.9.0-bin /opt/module/flume
```

## 配置 flume

4. `flume`的配置文件都存放在`${FLUME_HOME}/conf`中

```bash
cd /opt/module/flume/conf
```

5. 将`配置文件模板`复制成`配置文件`

```bash
cp flume-env.sh.template flume-env.sh
```

6. 配置`flume-env.sh`

- 添加 java 环境变量

```bash
export JAVA_HOME=/opt/module/jdk
```

## 启动 flume

- 启动 Flume agent 并加载配置文件

```bash
/opt/module/flume/bin/flume-ng agent -c conf -f /opt/module/flume/conf/flume-conf.properties -n a1 -Dflume.root.logger=INFO,console
```

- 若能够正常启动代表搭建成功

## 若要与 hadoop 进行交互（可选）

1. 从 hadoop 中复制必要的`jar`包

```bash
cp $HADOOP_HOME/share/hadoop/common/hadoop-common-3.1.3.jar  /opt/module/flume/lib
cp $HADOOP_HOME/share/hadoop/common/lib/hadoop-auth-3.1.3.jar /opt/module/flume/lib
cp $HADOOP_HOME/share/hadoop/common/lib/commons-configuration2-2.1.1.jar /opt/module/flume/lib
```

2. 将 hadoop 的配置文件复制到 flume 的配置文件

```bash
cp $HADOOP_HOME/etc/hadoop/core-site.xml /opt/module/flume/conf
cp $HADOOP_HOME/etc/hadoop/hdfs-site.xml /opt/module/flume/conf
```

3. 删除冲突的 `jar` 包

```bash
rm /opt/module/flume/lib/guava-11.0.2.jar
```

# 参考文章

[flume环境配置-传输Hadoop日志（namenode或datanode日志）](https://blog.csdn.net/dafsq/article/details/131229101)
