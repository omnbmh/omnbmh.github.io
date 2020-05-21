---
layout: post
title: 【MySQL系列】 0x01. MySQL二进制安装及配置
date: 2016-07-01 16:16:16
tags: [MySQL]
categories: [MySQL系列]
baseline:
---

### 准备二进制文件

- mysql-5.7.30-linux-glibc2.12-x86_64.tar.gz
  - 链接: [https://pan.baidu.com/s/1frQhhPkqVPitgkZeVkHgHw](https://pan.baidu.com/s/1frQhhPkqVPitgkZeVkHgHw) 提取码: hs8b
- 主机IP 10.10.10.10

### 安装

文章中大部分命令需要 root 权限

工作目录 `/app` tar.gz 包已经放置在该目录下

```
$ ls /app

--- output
mysql-5.7.30-linux-glibc2.12-x86_64.tar.gz
```

解压 tar.gz 包

```
$ cd /app
$ tar zxf mysql-5.7.30-linux-glibc2.12-x86_64.tar.gz
```

### 配置

二进制包安装主要是环境、权限、配置文件等的配置

#### 添加mysql用户

推荐添加一个 mysql 用户

```
$ groupadd mysql
$ useradd -s /bin/bash -m -g mysql mysql
```

#### 创建数据目录

```
$ mkdir -p /data/mysql_data
$ chown -R mysql:mysql /data/mysql_data
```

#### 初始化 MySQL

记住两个目录，后面会用到
- basedir MySQL 安装目录 `/app/mysql-5.7.30-linux-glibc2.12-x86_64`
- datadir MySQL 存放数据的目录 `/data/mysql_data` 默认是 `/var/lib/mysql`

5.7.6 之前 root 运行涉及到切换用户

```
$ cd /app/mysql-5.7.30-linux-glibc2.12-x86_64
$ ./scripts/mysql_install_db --user=mysql --basedir=/app/mysql-5.7.30-linux-glibc2.12-x86_64 --datadir=/data/mysql_data
```

5.7.6 之后

```
$ cd /app/mysql-5.7.30-linux-glibc2.12-x86_64
$ ./bin/mysqld --initialize --user=mysql --basedir=/app/mysql-5.7.30-linux-glibc2.12-x86_64 --datadir=/data/mysql_data
```

> 初始化过程中会打印出默认密码,记录下来

```
...
A temporary password is generated for root@localhost: y/Bgcwy,r9id
...
```

#### 配置文件 `vi /etc/my.cnf`

MySQL 配置文件配置项 非常多 参考官方文档:

- 最小化配置文件

```
[mysqld]
basedir=/app/mysql-5.7.30-linux-glibc2.12-x86_64
datadir=/data/mysql_data
socket=/tmp/mysql.sock
symbolic-links=0
user=mysql
port=3306
bind-address=10.10.10.10
lower_case_table_names=1

[mysqld_safe]
log-error=/var/log/mariadb/mariadb.log
pid-file=/var/run/mysql.pid

!includedir /etc/my.cnf.d
```

### 启动

```
$ cd /app/mysql-5.7.30-linux-glibc2.12-x86_64
$ cp support-files/mysql.server /etc/init.d/mysqld
$ /etc/init.d/mysqld start

--- output
```

设置开机启动

```
$ systemctl enable mysqld.service
```

### 启用远程 root 登录

默认mysql是禁止远程登录的

#### 使用 mysql 客户端登录 测试下

```
$ ./bin/mysql -uroot -S /data/mysql_data/mysql.sock -p

--- output
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 13
Server version: 5.7.30 MySQL Community Server (GPL)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

输入上面记录的密码，正常登录

> MYSQL 5.7.4 版本密码默认是过期的需要修改密码

```
mysql> show databases;

--- output
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
```

修改下默认的root密码

MySQL Version < 5.7.6

```
mysql> SET PASSWORD = PASSWORD('123456');
```

MySQL Version >= 5.7.6

```
mysql> ALTER USER USER() IDENTIFIED BY '123456';
```

授权 root 远程登录

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

### 远程登录，并执行一些SQL

```
$ mysql -h 10.10.10.10 -P 3306 -uroot -p

--- out
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 14
Server version: 5.7.30 MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.04 sec)

mysql>

```

###
