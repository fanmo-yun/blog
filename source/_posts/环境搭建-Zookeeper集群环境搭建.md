---
title: "环境搭建:Zookeeper集群环境搭建"
categories:
  - 环境搭建
tags:
  - 环境搭建
  - Zookeeper
abbrlink: 552d63d0
date: 2024-06-26 16:57:11
---

# Zookeeper 集群环境搭建

## 复制和安装 zookeeper

1. 将本地`apache-zookeeper-3.5.7-bin.tar.gz`复制进`master`容器中`root`目录中

```bash
docker cp /path/to/apache-zookeeper-3.5.7-bin.tar.gz master:/root
```

2. 将`zookeeper`解压进`/opt/module`目录下

```bash
tar zxvf /root/apache-zookeeper-3.5.7-bin.tar.gz -C /opt/module/
```

3. 将解压后的目录改名为 `zookeeper`

```bash
mv /opt/module/apache-zookeeper-3.5.7-bin /opt/module/zookeeper
```

## 配置 zookeeper

4. `zookeeper`的配置文件都存放在`${ZOOKEEPER_HOME}/conf`中

```bash
cd /opt/module/zookeeper/conf
```

5. 将`配置文件模板`复制成`配置文件`

```bash
cp zoo_sample.cfg zoo.cfg
```

6. 配置 `zoo.cfg`

- 修改 `dataDir`

```conf
initLimit=5
syncLimit=2
dataDir=/opt/module/zookeeper/data
```

- 在最底部添加

```conf
server.1=master:2888:3888
server.2=slave1:2888:3888
server.3=slave2:2888:3888
```

## 配置 `data` 文件夹

7. 在 `zookeeper` 目录下创建 `data` 文件夹

```bash
mkdir -p /opt/module/zookeeper/data
```

8. 在`data`文件夹中配置`myid`文件，并写入 `1` 表示 `master` 节点的 ID

```bash
echo "1" > myid
```

## 分发 `zookeeper`

```bash
scp -r /opt/module/zookeeper slave1:/opt/module
scp -r /opt/module/zookeeper slave2:/opt/module
```

9. 配置 `slave1` 中的 `myid` 文件，写入 `2` 表示 `slave1` 节点的 ID

```bash
echo "2" > /opt/module/zookeeper/data/myid
```

10. 配置 `slave2` 中的 `myid` 文件，写入 `3` 表示 `slave2` 节点的 ID

```bash
echo "3" > /opt/module/zookeeper/data/myid
```

## 启动集群

> !!!三个容器中都有执行!!!

```bash
/opt/module/zookeeper/bin/zkServer.sh start
```

## 验证集群

11. 使用`jps`命令查看`zookeeper`进程：

```bash
jps
```

12. 三个容器均出现进程`QuorumPeerMain`表示启动成功

```bash
7940 QuorumPeerMain
```

13. 输入指令进入`zkCli`

```bash
/opt/module/zookeeper/bin/zkCli.sh
```

14. 输入 `ls /`, 出现`[zookeeper]`即配置成功

```bash
[zk: localhost:2181(CONNECTED) 0] ls /
[zookeeper]
```

# 参考文章

[zookeeper官方文档](https://zookeeper.apache.org/doc/r3.5.7/index.html)
