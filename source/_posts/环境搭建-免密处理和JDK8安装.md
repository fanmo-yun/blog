---
title: "环境搭建:免密处理和JDK8安装"
categories:
  - 环境搭建
tags:
  - 环境搭建
  - 免密处理
  - JDK8
abbrlink: 6557c80f
date: 2024-06-26 07:47:39
---

# 免密处理

> !!!以下指令需要在三个容器中都运行!!!

## 安装 ssh 及网络工具

1. 输入指令来安装 ssh 和网络工具

```bash
yum install openssh-server openssh-clients net-tools
```

## 配置 ssh

2. 编辑`/etc/ssh/sshd_config`文件

```bash
vi /etc/ssh/sshd_config
```

3. 去掉`Port 22`此行最前面的"#"号, 开放`22`端口

```bash
# If you want to change the port on a SELinux system, you have to tell
# SELinux about this change.
# semanage port -a -t ssh_port_t -p tcp #PORTNUMBER
#
Port 22
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::
```

4. 开启 ssh

```bash
systemctl start sshd    # 开启ssh
systemctl enable sshd   # 开机自启
```

## 修改`root密码`

5. 输入`passwd`指令后修改`root`的密码

```bash
passwd
```

## 配置`/etc/hosts`文件

6. 将`master slave1 slave2`容器的`ip地址`添加进`/etc/hosts`文件

```bash
192.168.1.151   master
192.168.1.152   slave1
192.168.1.153   slave2
```

## 免密处理

7. 生成`ssh`密钥, 并应用在每容器中

```bash
ssh-keygen
```

- `ssh-keygen`指令用于生成公钥与私钥对

```bash
ssh-copy-id master; ssh-copy-id slave1; ssh-copy-id slave2
```

- `ssh-copy-id`指令用于将本地的公钥复制到远程机器的 authorized_keys 文件中

# 安装 JDK8

## 复制和安装 JDK8

1. 将本地的`jdk-8u202-linux-x64.tar.gz`复制进`master`容器中`root`目录中

```bash
docker cp /path/to/jdk-8u202-linux-x64.tar.gz master:/root
```

- `docker cp`指令用于将本地文件复制进容器指定目录中

2. 使用`tar`指令解压到`/opt/module`目录中(目录没有直接创建一个)

```bash
tar zxvf /root/jdk-8u202-linux-x64.tar.gz -C /opt/module/
```

3. 将`jdk`文件夹重新命名

```bash
mv /opt/module/jdk1.8.0_202/ /opt/module/jdk
```

4. 编辑`/etc/profile`环境

```bash
vi /etc/profile
```

## 配置环境

5. 添加`JAVA_HOME`和`PATH`环境变量

```bash
export JAVA_HOME=/opt/module/jdk
export PATH=$PATH:$JAVA_HOME/bin
```

6. 生效`profile`

```bash
source /etc/profile
```

- 建议生效后输入 java 的指令验证一下: `java` `javac`

## 分发文件和环境

8. 分发`/opt/module/jdk/` `/etc/profile`到容器`slave1` `slave2`, 并且生效容器`slave1` `slave2`中的`profile`

```bash
scp -r /opt/module/jdk/ slave1:/opt/module/; scp -r /opt/module/jdk/ slave2:/opt/module/

scp /etc/profile slave1:/etc/profile; scp /etc/profile slave2:/etc/profile
```

# 参考文章

[使用 ssh-keygen 和 ssh-copy-id 三步实现 SSH 无密码登录 和 ssh 常用命令](https://blog.csdn.net/alifrank/article/details/48241699)
