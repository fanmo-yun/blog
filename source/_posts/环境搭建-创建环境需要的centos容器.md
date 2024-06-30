---
title: "环境搭建:创建环境需要的centos容器"
categories:
  - 环境搭建
tags:
  - 环境搭建
  - docker
abbrlink: 62d1488f
date: 2024-06-25 15:47:05
---

# 容器准备

## 拉取`centos`镜像

1. 使用`docker pull`指令拉取镜像

```bash
docker pull centos:centos7.9.2009
```

- `docker pull`命令从`docker hub`上拉取一个`centos7.9.2009`的镜像

## 建立容器待更新

2. 新建一个容器并设置名为`centos`，再进入这个容器

```bash
docker run -itd --name centos centos:centos7.9.2009 /bin/bash
docker exec -it centos /bin/bash
```

- `docker run`命令用于在 Docker 中运行一个容器
- `-itd` 是选项组合：
  - `-i`: 交互模式运行容器
  - `-t`: 为容器分配一个伪终端
  - `-d`: 后台运行该容器（分离模式）
- `--name centos`为容器建立一个名称为`centos`
- `centos:centos7.9.2009`镜像名称和标签
- `/bin/bash`启动容器后启动/bin/bash
- `docker exec`在运行的容器中执行命令

## 更新容器

3. 备份`source.list`到`home`目录

```bash
cp /etc/yum.d/source.list ~/source.list
```

- 将`source.list`文件备份到主目录

4. 添加`ustc`镜像源

```bash
sed -e 's|^mirrorlist=|#mirrorlist=|g' \
         -e 's|^#baseurl=http://mirror.centos.org/centos|baseurl=https://mirrors.ustc.edu.cn/centos|g' \
         -i.bak \
         /etc/yum.repos.d/CentOS-Base.repo
```

- 修改容器中的`yum`源为中国科学技术大学的镜像源

5. 更新

```bash
yum update
```

- 更新容器中的软件包

## 保存容器为镜像

6. 提交保存镜像

```bash
docker commit -m "centos:update" centos centos:v1
```

- `docker commit`把有修改的容器提交成新的镜像
- `-m`添加注释
- `centos`有修改的容器名称
- `centos:v1`新的镜像名称和标签名

## 创建网络待容器使用

7. 创建网络

```bash
docker network create --subnet=192.168.1.0/24 mynetwork
```

- `docker network create`创建自定义网络的 Docker 命令
- `--subnet=192.168.1.0/24`创建一个网络并指定新网络的子网参数
  - `192.168.1.0/24`: 创建一个子网掩码为 255.255.255.0 的子网，范围从 192.168.1.1 到 192.168.1.254。/24 表示子网掩码有 24 位（255.255.255.0）
- `mynetwork`: 自定义网络的名称

## 建立环境容器

8. 输入以下指令启动三个容器, 模拟分布式

```bash
docker run -itd --name master -v /sys/fs/cgroup:/sys/fs/cgroup --network mynetwork --ip 192.168.1.151 --privileged=true --hostname master centos:v1 /sbin/init
docker run -itd --name slave1 --network mynetwork --ip 192.168.1.152 --privileged=true --hostname slave1 centos:v1 /sbin/init
docker run -itd --name slave2 --network mynetwork --ip 192.168.1.153 --privileged=true --hostname slave2 centos:v1 /sbin/init
```

- `-v /sys/fs/cgroup:/sys/fs/cgroup`: 将宿主机的`/sys/fs/cgroup`目录挂载到容器的`/sys/fs/cgroup`目录。这样做是为了让容器能够访问宿主机的`cgroup`文件系统，为后面安装`mysql`打下基础
- `--network mynetwork`: 将容器连接到名为 mynetwork 的 Docker 网络
- `--ip 192.168.1.151`: 指定容器的 IP 地址为 192.168.1.151
- `--privileged=true`: 启用特权模式
- `/sbin/init`: 在容器启动时运行`/sbin/init`

9. 输入以下指令进入此容器

```bash
docker exec -it master /bin/bash
docker exec -it slave1 /bin/bash
docker exec -it slave2 /bin/bash
```

- 分别进入`master`、`slave1`和`slave2`容器的Bash终端

# 参考文章

[Docker “pull“命令获取镜像，讲道理你真的会吗？](https://blog.csdn.net/zhuzicc/article/details/118066353)

[深入理解 Docker Run 命令：从入门到精通](https://cloud.tencent.com/developer/article/2390830)

[Docker network 命令](https://blog.csdn.net/gezhonglei2007/article/details/51627821)

[docker 简明教程](https://www.bookstack.cn/read/dockerguide/)
