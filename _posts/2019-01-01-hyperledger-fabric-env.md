---
layout: post
title: Hyperledger Fabric 环境准备（一）
date: 2019-01-01 18:51:18
tags: [Hyperledger Fabric]
categories: [Hyperledger Fabric]
baseline:
---

Hyperledger项目是区块链技术中第一个面向企业应用场景开源分布式账本平台。

官网： https://www.hyperledger.org

Github： https://github.com/hyperledger

### 0x00 Env
- Ubuntu 18.04
- Docker 18.06.1-ce Docker-Compose 1.21.2
- Hyperledger fabric 1.2.0

### 0x01 Go
- `apt install golang-go`
- 设置环境变量
```
export GOPATH=$HOME/go
export GOROOT=/usr/local/go
export PATH=$GOROOT/bin:$PATH
```
- 查看下版本
```
$ go version
go version go1.10.4 linux/amd64
```

### 0x02 Install Docker && Docker-Compose
- 执行下面的命令

```
$ sudo apt update
$ sudo apt install docker.io docker-compose
```

- 检测docker运行状态

```
$ sudo service docker status
-----
● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; disabled; vendor preset: enabled)
   Active: active (running) since Fri 2019-01-04 02:46:33 UTC; 30s ago
     Docs: https://docs.docker.com
 Main PID: 18056 (dockerd)
    Tasks: 23
   CGroup: /system.slice/docker.service
           ├─18056 /usr/bin/dockerd -H fd://
           └─18078 docker-containerd --config /var/run/docker/containerd/containerd.toml
-----
```

- 查看版本号

```
$ sudo docker version
-----
Client:
 Version:           18.06.1-ce
 API version:       1.38
 Go version:        go1.10.4
 Git commit:        e68fc7a
 Built:             Fri Oct 19 19:43:14 2018
 OS/Arch:           linux/amd64
 Experimental:      false

Server:
 Engine:
  Version:          18.06.1-ce
  API version:      1.38 (minimum version 1.12)
  Go version:       go1.10.4
  Git commit:       e68fc7a
  Built:            Thu Sep 27 02:39:50 2018
  OS/Arch:          linux/amd64
  Experimental:     false
-----

$ sudo docker-compose version
-----
docker-compose version 1.17.1, build unknown
docker-py version: 2.5.1
CPython version: 2.7.15rc1
OpenSSL version: OpenSSL 1.1.0g  2 Nov 2017
-----
```

### 0x03 Download Hyperledger Fabric 二进制文件
- 下载

```
$ mkdir -p $HOME/fabric-network
$ cd $HOME/fabric-network
$ curl https://nexus.hyperledger.org/content/repositories/releases/org/hyperledger/fabric/hyperledger-fabric/linux-amd64-1.2.0/hyperledger-fabric-linux-amd64-1.2.0.tar.gz -o hyperledger-fabric-linux-amd64-1.2.0.tar.gz

$ curl https://nexus.hyperledger.org/content/repositories/releases/org/hyperledger/fabric-ca/hyperledger-fabric-ca/linux-amd64-1.2.0/hyperledger-fabric-ca-linux-amd64-1.2.0.tar.gz -o hyperledger-fabric-ca-linux-amd64-1.2.0.tar.gz
```

- 解压

```
$ tar zxvf hyperledger-fabric-linux-amd64-1.2.0.tar.gz
$ tar zxvf hyperledger-fabric-ca-linux-amd64-1.2.0.tar.gz
$ ls
-----
bin  config  hyperledger-fabric-ca-linux-amd64-1.2.0.tar.gz  hyperledger-fabric-linux-amd64-1.2.0.tar.gz
-----
```
