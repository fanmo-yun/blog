---
title: "环境搭建:Hive数据仓库搭建"
abbrlink: "677e0722"
date: 2024-06-27 10:30:03
categories:
  - 环境搭建
tags:
  - 环境搭建
  - Hive
---

# Hive 数据仓库搭建

## 复制和安装 hive

1. 将本地`apache-hive-3.1.2-bin.tar.gz`和`MySQL驱动压缩包`复制进`master`容器中`root`目录中

```bash
docker cp /path/to/apache-hive-3.1.2-bin.tar.gz master:/root
docker cp /path/to/mysql-connector-java-5.1.27.zip master:/root
```

2. 将`hive`解压进`/opt/module`目录下并且改名, 将`jdbc`压缩包解压到hive的lib目录下

```bash
tar zxvf /root/apache-hive-3.1.2-bin.tar.gz -C /opt/module/
```

```bash
mv /opt/module/apache-hive-3.1.2-bin /opt/module/hive
```

```bash
unzip /root/mysql-connector-java-5.1.27.zip -d /opt/module/hive/lib
cp /opt/module/hive/lib/mysql-connector-java-5.1.27/mysql-connector-java-5.1.27-bin.jar /opt/module/hive/lib/
```

## 配置 hive

3. `hive`的配置文件都存放在`${HIVE_HOME}/conf`中

```bash
cd /opt/module/hive/conf
```

4. 将`配置文件模板`复制成`配置文件`

```bash
cp hive-env.sh.template hive-env.sh
```

5. 配置`hive-env.sh`

- 添加`hadoop目录`和`hive配置文件目录`的环境变量

```bash
export HADOOP_HOME=/opt/module/hadoop
export HIVE_CONF_DIR=/opt/module/hive/conf
```

6. **创建**`hive-site.xml`并配置

```xml
<configuration>
<property>
   <name>hive.cli.print.header</name>
   <value>true</value>
</property>
<property>
   <name>hive.cli.print.current.db</name>
   <value>true</value>
</property>
<property>
   <name>javax.jdo.option.ConnectionURL</name>
   <value>jdbc:mysql://localhost:3306/metastore?createDatabaseIfNotExist=true</value>
</property>
<property>
   <name>javax.jdo.option.ConnectionDriverName</name>
   <value>com.mysql.jdbc.Driver</value>
</property>
<property>
   <name>hive.metastore.warehouse.dir</name>
   <value>/user/hive/warehouse</value>
</property>
<property>
   <name>javax.jdo.option.ConnectionUserName</name>
   <value>root</value>
</property>
<property>
   <name>javax.jdo.option.ConnectionPassword</name>
   <value>1234</value>
</property>
</configuration>
```

- `hive.cli.print.header`:是否在 cli 输出中显示表的列名
- `hive.cli.print.current.db`:是否在 cli 提示符中显示当前使用的数据库名称
- `javax.jdo.option.ConnectionURL`:用于连接元数据数据库的 JDBC URL
- `javax.jdo.option.ConnectionDriverName`:数据库连接的驱动名称
- `hive.metastore.warehouse.dir`:hive 在 hdfs 的默认数据仓库目录
- `javax.jdo.option.ConnectionUserName`:连接元数据数据库时使用的用户名
- `javax.jdo.option.ConnectionPassword`:连接元数据数据库时使用的密码

7. 删除旧版的`guava.jar`文件，从hadoop中复制新版的`guava.jar`，这样做为了防止两个版本之间发生冲突

```bash
rm -rf /opt/module/hive/lib/guava-19.0.jar
cp /opt/module/hadoop/share/hadoop/common/lib/guava-27.0-jre.jar /opt/module/hive/lib/
```

## 初始化 hive

```bash
/opt/module/hive/bin/schematool -dbType mysql -initSchema
```

## 验证 hive

8. 进入 Hive 命令行界面

```bash
/opt/module/hive/bin/hive
```

9. 测试 Hive 是否正常工作

```sql
hive> show databases;
```

10. 如果显示如下信息，则表示 Hive 配置成功：

```sql
OK
default
Time taken: 0.976 seconds, Fetched: 1 row(s)
```

# 参考文章

[Hive 3.1.2 启动报错 guava 版本冲突问题解决](https://blog.csdn.net/shockang/article/details/118052667)
[Hive详解(02) - Hive 3.1.2安装](https://www.cnblogs.com/meanshift/p/15802914.html)
