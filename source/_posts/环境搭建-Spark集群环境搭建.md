---
title: "环境搭建:Spark集群环境搭建"
categories:
  - 环境搭建
tags:
  - 环境搭建
  - Spark
abbrlink: 16951bbc
date: 2024-06-26 15:06:48
---

# Spark 集群环境搭建

## 复制和安装 spark

1. 将本地`spark`复制进`master`容器中`root`目录中

```bash
docker cp /path/to/spark-3.1.1-bin-hadoop3.2.tgz master:/root
```

2. 将`spark`解压进`/opt/module`目录下

```bash
tar zxvf /root/spark-3.1.1-bin-hadoop3.2 -C /opt/module/
```

3. 将解压后的目录改名为`spark`

```bash
mv /opt/module/spark-3.1.1-bin-hadoop3.2 /opt/module/spark
```

## 配置 spark

1. `spark`的配置文件都存放在`${SPARK_HOME}/conf`中

```bash
cd /opt/module/spark/conf/
```

2. 将`配置文件模板`复制成`配置文件`

```bash
cp spark-defaults.conf.template spark-defaults.conf
cp spark-env.sh.template spark-env.sh
cp workers.template workers
```

3. 配置`spark-defaults.conf`

- 该文件存放着 spark 运行时的默认选项

- 将最下面`Example`中的一些注释去除:

```conf
# Example:
spark.master                     spark://master:7077
spark.eventLog.enabled           true
# spark.eventLog.dir               hdfs://namenode:8021/directory
spark.serializer                 org.apache.spark.serializer.KryoSerializer
spark.executor.memory            1g
# spark.executor.extraJavaOptions  -XX:+PrintGCDetails -Dkey=value -Dnumbers="one two three"
```

4. 配置`spark-env.sh`

- 该文件存放着 spark 的环境变量

- 最顶端添加如下的内容

```bash
export JAVA_HOME=/opt/module/jdk
export HADOOP_HOME=/opt/module/hadoop
export HADOOP_CONF_DIR=/opt/module/hadoop/etc/hadoop
export YARN_CONF_DIR=/opt/module/hadoop/etc/hadoop
export SPARK_MASTER_HOST=master
export SPARK_MASTER_IP=master
export SPARK_MASTER_PORT=7077
```

- 指定`JAVA_HOME`路径
- 指定`hadoop`安装所在路径
- 指定`hadoop`配置文件所在路径
- 指定`YARN`配置文件所在目录
- 指定`Spark Master`的主机名
- 指定`Spark Master`的 IP 地址
- 指定`Spark Master`的端口号

5. 配置`workers`

- 配置所有工作节点的主机名或 IP 地址

```conf
master
slave1
slave2
```

6. 分发 spark 到 slave1 和 slave2

```bash
scp -r /opt/module/spark slave1:/opt/module/
scp -r /opt/module/spark slave2:/opt/module/
```

## 启动群集

- 在`master`容器中运行以下命令

```bash
/opt/module/spark/sbin/start-master.sh
```

```bash
/opt/module/spark/sbin/start-workers.sh
```

## 测试群集

7. 运行 `spark` 中自带的计算 `π` 的案例

```bash
/opt/module/spark/bin/spark-submit --master yarn --class org.apache.spark.examples.SparkPi /opt/module/spark/examples/jars/spark-examples_2.12-3.1.1.jar
```

- 输出的是`π`的大概值(每个人运算出的结果都不一样)

```bash
23/10/23 06:55:45 INFO YarnScheduler: Removed TaskSet 0.0, whose tasks have all completed, from pool
23/10/23 06:55:45 INFO DAGScheduler: ResultStage 0 (reduce at SparkPi.scala:38) finished in 1.953 s
23/10/23 06:55:45 INFO DAGScheduler: Job 0 is finished. Cancelling potential speculative or zombie tasks for this job
23/10/23 06:55:45 INFO YarnScheduler: Killing all running tasks in stage 0: Stage finished
23/10/23 06:55:45 INFO DAGScheduler: Job 0 finished: reduce at SparkPi.scala:38, took 2.008845 s
--- Pi is roughly 3.143515717578588 ---
23/10/23 06:55:45 INFO SparkContext: SparkContext is stopping with exitCode 0.
23/10/23 06:55:45 INFO SparkUI: Stopped Spark web UI at http://master:4040
23/10/23 06:55:45 INFO YarnClientSchedulerBackend: Interrupting monitor thread
23/10/23 06:55:45 INFO YarnClientSchedulerBackend: Shutting down all executors
23/10/23 06:55:45 INFO YarnSchedulerBackend$YarnDriverEndpoint: Asking each executor to shut down
```

8. 使用`jps`命令查看`spark`进程：

```bash
jps
```

- master 容器中

```bash
9329 Jps
8935 Master
9127 Worker
```

- slave1 容器中

```bash
9359 Jps
8127 Worker
```

- slave2 容器中

```bash
2329 Jps
9457 Worker
```

# 参考文章

[Spark Configuration](https://spark.apache.org/docs/latest/configuration.html)
