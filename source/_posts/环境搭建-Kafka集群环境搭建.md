---
title: "环境搭建:Kafka集群环境搭建"
categories:
  - 环境搭建
tags:
  - 环境搭建
  - Kafka
abbrlink: a5092baa
date: 2024-06-26 16:58:36
---

# Kafka 集群环境搭建

## 复制和安装 kafka

1. 将本地`kafka_2.11-2.4.1.tgz`复制进`master`容器中`root`目录中

```bash
docker cp /path/to/kafka_2.11-2.4.1.tgz master:/root
```

2. 将`kafka`解压进`/opt/module`目录下

```bash
tar zxvf /root/kafka_2.11-2.4.1.tgz -C /opt/module/
```

3. 将解压后的目录改名为`kafka`

```bash
mv /opt/module/kafka_2.11-2.4.1 /opt/module/kafka
```

## 配置 kafka

4. `kafka`的配置文件都存放在`${KAFKA_HOME}/conf`中

```bash
cd /opt/module/kafka/config
```

5. 配置 `server.properties`

```conf
broker.id=0
listeners=PLAINTEXT://master:9092
log.dirs=/opt/module/kafka/data
zookeeper.connect=master:2181,slave1:2181,slave2:2181
```

- `broker.id` 用于标识 Kafka broker 的唯一 ID。每个 broker 在集群中必须有一个唯一的 ID
- `listeners=PLAINTEXT://:9092`：`listeners` 用于指定 broker 监听客户端连接的地址和端口、`PLAINTEXT` 表示使用明文传输（不加密）
- `log.dirs`：用于指定 Kafka 存储日志数据（包括消息）的目录。这个目录将包含 Kafka topic 的分区数据
- `zookeeper.connect=master:2181,slave1:2181,slave2:2181`：用于指定 Kafka 集群使用的 Zookeeper 集群的连接字符串、Zookeeper 是 Kafka 用来进行集群管理和协调的服务、连接字符串包括 Zookeeper 的主机名和端口号。多个 Zookeeper 实例用逗号分隔

6. 分发 `kafka`

```bash
scp -r /opt/module/kafka/ slave1:/opt/module/
scp -r /opt/module/kafka/ slave2:/opt/module/
```

7. 修改 `slave1` `slave2` 中的 `server.properties`

- slave1

```conf
broker.id=1
listeners=PLAINTEXT://slave1:9092
```

- slave2

```conf
broker.id=2
listeners=PLAINTEXT://slave2:9092
```

## 启动群集

> 在开启 kafka 之前确保 zookeeper 是开启状态

8. !!!三个容器中都有执行!!!

```bash
/opt/module/kafka/bin/kafka-server-start.sh -daemon /opt/module/kafka/config/server.properties
```

## 验证集群

9. 使用`jps`命令查看`zookeeper`进程：

```bash
jps
```

- 出现以下进程表示开启成功

```bash
4548 Kafka
```

# 参考文章

[Kafka 官方文档](https://kafka.apache.org/documentation/#gettingStarted)
