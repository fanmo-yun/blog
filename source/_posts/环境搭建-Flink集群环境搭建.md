---
title: "环境搭建:Flink集群环境搭建"
categories:
  - 环境搭建
tags:
  - 环境搭建
  - Flink
abbrlink: d6dfb64e
date: 2024-06-26 15:58:48
---

# Flink 集群环境搭建

## 复制和安装 flink

1. 将本地`flink`复制进`master`容器中`root`目录中

```bash
docker cp /path/to/flink-1.14.0-bin-scala_2.11.tgz master:/root
```

2. 在`/etc/profile`中添加如下环境变量

```bash
export HADOOP_CLASSPATH = `hadoop classpath`
```

3. 将 `flink` 解压进 `/opt/module` 目录下

```bash
tar zxvf /root/flink-1.14.0-bin-scala_2.11.tgz -C /opt/module/
```

4. 将解压后的目录改名为`flink`

```bash
mv /opt/module/flink-1.14.0-bin-scala_2.11 /opt/module/flink
```

## 配置 flink

5. `flink`的配置文件都存放在`${FLINK_HOME}/conf`中

```bash
cd /opt/module/flink/conf
```

6. 配置`flink-conf.yaml`

- 该文件是 Flink 的基本配置，用于配置`JobManager`、内存大小等

```yaml
jobmanager.rpc.address: master
taskmanager.numberOfTaskSlots: 2
```

- 指定`JobManager`为`master`
- 设置`TaskManager`的插槽数量为 2（根据你的需要进行调整）

7. 配置`workers`

- 该文件用于指定`TaskManager`节点

```conf
master
slave1
slave2
```

8. 配置 `masters`

- 该文件用于指定`JobManager`节点和端口

```conf
master:8081
```

9. 分发`flink`

```bash
scp -r /opt/module/flink/ slave1:/opt/module/
scp -r /opt/module/flink/ slave2:/opt/module/
```

## 启动群集

10. 在容器 `master` 中执行

```bash
/opt/module/flink/bin/start-cluster.sh
```

## 验证集群

```bash
jps
```

11. 使用`jps`命令查看`flink`进程：

```bash
8275 StandaloneSessionClusterEntrypoint
8622 TaskManagerRunner
```

12. 在 `slave1` `slave2` 中显示这个进程则开启成功

```bash
2943 TaskManagerRunner
```

# 参考文章

[快速入门 Flink(2)——Flink 集群环境搭建（3 台节点 建议收藏）](https://blog.csdn.net/qq_43791724/article/details/108030170)
[Flink 配置文件详解](https://blog.csdn.net/u013982921/article/details/96428258)
