---
layout: post
title: ZooKeeper的安装使用及集群的搭建
date: 2016-12-31 12:03:15
tags: [ZooKeeper]
categories:  
baseline:
---

### 搭建ZooKeeper集群

#### 硬件准备
规划使用三台虚机
- 192.168.8.101
- 192.168.8.102
- 192.168.8.103

#### 软件准备
- jdk-8u91-linux-x64.tar.gz
- zookeeper-3.4.6.tar.gz [百度网盘下载](https://pan.baidu.com/s/1i44i2GX "zookeeper-3.4.6.tar.gz")

#### 软件安装
```
tar zxvf zookeeper-3.4.8.tar.gz -C /app
cd /app
cp -r zookeeper-3.4.8 zk-cluster-101
mkdir -p /app/zk-cluster-101/{logs,data}
cd zk-cluster-101
cp conf/zoo_sample.cfg conf/zoo.cfg
```
#### 软件配置
##### 修改 conf/zoo.cfg 文件 如下
```
tickTime=2000
initLimit=10
syncLimit=5
dataLogDir=/app/zk-cluster-101/logs
dataDir=/app/zk-cluster-101/data
clientPort=2181
autopurge.snapRetainCount=500
autopurge.purgeInterval=24
server.101= 192.168.8.101:2888:3888
server.102= 192.168.8.102:2888:3888
server.103= 192.168.8.103:2888:3888
```
##### 创建Server标识
```
echo "101" > /app/zk-cluster-101/data/myid
```
#### 软件部署
```
cd /app
cp -r zk-cluster-101 zk-cluster-102
```
##### 修改配置文件zoo.cfg
```
dataLogDir=/app/zk-cluster-102/logs
dataDir=/app/zk-cluster-102/data
```
##### 修改Server标识
```
echo "102" > /app/zk-cluster-101/data/myid
```
```
cp -r zk-cluster-101 zk-cluster-103
```
##### 修改配置文件zoo.cfg
```
dataLogDir=/app/zk-cluster-103/logs
dataDir=/app/zk-cluster-103/data
```
##### 修改Server标识
```
echo "103" > /app/zk-cluster-101/data/myid
```


##### 将/app/zk-cluster-101 传到 192.168.8.101 执行 /app/zk-cluster-101/bin/zkServer.sh start
##### 将/app/zk-cluster-102 传到 192.168.8.102 执行 /app/zk-cluster-102/bin/zkServer.sh start
##### 将/app/zk-cluster-103 传到 192.168.8.103 执行 /app/zk-cluster-103/bin/zkServer.sh start

#### 结束
##### 接下来可以使用 zkCli.sh 进行测试

### Question

1. 集群节点数为什么是奇数个?

> https://blog.csdn.net/u010476994/article/details/79806041

ZooKeeper的选举规则: Leader的选举，要求 `可用节点数>总节点数量/2` 注意是 > 不是大于等于



#### 参考
> http://www.cnblogs.com/linuxprobe/p/5851699.html
> https://devopscube.com/how-to-setup-a-zookeeper-cluster/
> ...
