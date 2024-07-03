---
title: "环境搭建:MySQL数据库搭建"
abbrlink: cec3650d
date: 2024-06-27 10:29:58
categories:
  - 环境搭建
tags:
  - 环境搭建
  - MySQL
---

# MySQL 数据库搭建

## 复制和安装 mysql

1. 将本地`mysql-5.7.44-1.el7.x86_64.rpm-bundle.tar`复制进`master`容器中`root`目录中

```bash
docker cp /path/to/mysql-5.7.44-1.el7.x86_64.rpm-bundle.tar master:/root
```

2. 将`mysql`的 tar 包解压到当前目录中

```bash
tar xf mysql-5.7.44-1.el7.x86_64.rpm-bundle.tar
```

- 解压完毕后会得到以下 rpm 包

```
mysql-community-client-5.7.44-1.el7.x86_64.rpm
mysql-community-common-5.7.44-1.el7.x86_64.rpm
mysql-community-devel-5.7.44-1.el7.x86_64.rpm
mysql-community-embedded-5.7.44-1.el7.x86_64.rpm
mysql-community-embedded-compat-5.7.44-1.el7.x86_64.rpm
mysql-community-embedded-devel-5.7.44-1.el7.x86_64.rpm
mysql-community-libs-5.7.44-1.el7.x86_64.rpm
mysql-community-libs-compat-5.7.44-1.el7.x86_64.rpm
mysql-community-server-5.7.44-1.el7.x86_64.rpm
mysql-community-test-5.7.44-1.el7.x86_64.rpm
```

## 下载依赖并安装 rpm 包

3. 下载依赖

```bash
yum install perl net-tools libaio libnuma
```

4. 安装相应的 rpm 包

```bash
rpm -ivh mysql-community-common-5.7.44-1.el7.x86_64.rpm
rpm -ivh mysql-community-libs-5.7.44-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-5.7.44-1.el7.x86_64.rpm
rpm -ivh mysql-community-server-5.7.44-1.el7.x86_64.rpm
```

## 配置并运行 mysql

5. 初始化

```bash
mysqld --initialize
```

6. 查询 mysql 密码

```bash
tail /var/log/mysqld.log
```

- 某一行是这样的就是密码

```
2024-06-30T02:52:23.822524Z 1 [Note] A temporary password is generated for root@localhost: <l9Ml9CmtyjU
```

7. 修改数据库目录的所属用户及其所属组

```bash
chown mysql:mysql /var/lib/mysql -R
```

8. 启动 mysql 数据库

```bash
systemctl start mysqld
```

```bash
systemctl enable mysqld
```

## 进入 mysql 并修改密码和权限

9. 运行

```bash
mysql -u root -p'<l9Ml9CmtyjU'
```

10. 修改密码策略和密码长度

```sql
set global validate_password_policy=0;
set global validate_password_length=4;
```

11. 修改 root 密码

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY '1234';
```

12. 赋予 root 所有权限

```sql
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '1234' WITH GRANT OPTION;
```

13. 刷新权限

```sql
FLUSH PRIVILEGES;
```

13. 重启 MySQL

```bash
systemctl restart mysqld
```
