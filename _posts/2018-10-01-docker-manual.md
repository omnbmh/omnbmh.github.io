---
layout: post
title: Docker 使用手册
date: 2018-10-01 18:51:18
tags: [Docker 源 pull 镜像]
categories: [Docker]
baseline:
---

#### 修改下载源

由于国内网络的原因，`pull` 镜像的时候太慢，修改下默认下载源

编辑文件 `daemon.json` 如果没有就新建一个

``` shell
$ sudo vi /etc/docker/daemon.json
```

修改为下面的内容

```
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}
```

重启 Docker 服务

```
$ sudo service docker restart
```
