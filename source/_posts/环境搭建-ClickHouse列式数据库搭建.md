---
title: "环境搭建:ClickHouse列式数据库搭建"
abbrlink: a4fb20bc
date: 2024-06-27 10:32:47
categories:
tags:
---

# ClickHouse 列式数据库搭建

## 复制和安装 clickhouse

1. 将本地 `clickhouse` 相关文件复制进 `master` 容器中 `root` 目录中

```bash
docker cp clickhouse-client-21.9.4.35.tgz master:/root
docker cp clickhouse-common-static-21.9.4.35.tgz master:/root
docker cp clickhouse-common-static-dbg-21.9.4.35.tgz master:/root
docker cp clickhouse-server-21.9.4.35.tgz master:/root
```

2. 将 `clickhouse` 文件解压到 `/opt/module` 目录中

```bash
tar zxvf clickhouse-client-21.9.4.35.tgz -C /opt/module
tar zxvf clickhouse-common-static-21.9.4.35.tgz -C /opt/module
tar zxvf clickhouse-common-static-dbg-21.9.4.35.tgz -C /opt/module
tar zxvf clickhouse-server-21.9.4.35.tgz -C /opt/module
```

3. 安装 `clickhouse`

> 若有输入用户名或密码的则跳过

- 执行每个文件路径下 `install` 目录下 `doinstall.sh`

```bash
/opt/module/clickhouse-common-static-21.9.4.35/install/doinst.sh
/opt/module/clickhouse-common-static-21.9.4.35/install/doinst.sh
/opt/module/clickhouse-common-static-dbg-21.9.4.35/install/doinst.sh
/opt/module/clickhouse-server-21.9.4.35/install/doinst.sh
```

## 配置 clickhouse

4. 修改 `/etc/clickhouse-server/config.xml`

- 将里面 `9000` 端口改为 `9001`

```bash
sed -i 's/9000/9001/g' config.xml
```

- 在文件约`159`行处，将注释的文本去掉注释

```xml
<!-- <listen_host>0.0.0.0</listen_host> --> 改为 <listen_host>0.0.0.0</listen_host>
```

5. 开启并验证 clickhouse

```bash
systemctl start clickhouse-server
```

```bash
systemctl status clickhouse-server
```
