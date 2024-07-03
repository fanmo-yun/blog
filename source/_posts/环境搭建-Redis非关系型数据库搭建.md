---
title: '环境搭建:Redis非关系型数据库搭建'
abbrlink: 150d2b8b
date: 2024-06-27 10:31:56
categories:
  - 环境搭建
tags:
  - 环境搭建
  - Redis
---

# Redis 非关系型数据库搭建

## 复制和安装 redis

1. 将本地`redis`复制进`master`容器中`root`目录中

```bash
docker cp /path/to/redis-6.2.6.tar.gz master:/root
```

2. 将`redis`解压进`/opt/module`目录下

```bash
tar zxvf /root/redis-6.2.6.tar.gz -C /opt/module/
```

3. 进入 Redis 源码目录并安装 Redis

```bash
cd /opt/module/redis-6.2.6
```

```bash
make MALLOC=libc && make install
```

## 配置 redis

4. 修改 Redis 配置文件，将 Redis 设置为后台运行。在 Redis 的配置文件中（默认在解压目录下）

```bash
vim /opt/module/redis-6.2.6/redis.conf
```

5. 找到第 257 行，将`daemonize no`改为`daemonize yes`

```conf
daemonize yes
```

## 开启 redis

6. 使用修改后的配置文件启动 Redis

```bash
redis-server /opt/module/redis-6.2.6/redis.conf
```

## 验证 redis

7. 使用 Redis 命令行客户端连接到 Redis 实例

```bash
redis-cli
```

8. 如果连接成功，会出现以下提示符

```bash
127.0.0.1:6379>
```

9. 输入 Redis 命令进行操作

```bash
127.0.0.1:6379> ping
PONG
```
