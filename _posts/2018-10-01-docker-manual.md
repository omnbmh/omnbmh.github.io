---
layout: post
title: Docker 使用手册
date: 2018-10-01 18:51:18
tags: [Docker]
categories: [Docker]
baseline:
---

#### 0x01 修改下载源

由于国内网络的原因，`pull` 镜像的时候太慢，修改下默认下载源

- 编辑文件 `/etc/default/docker`

``` shell
$ sudo vi /etc/default/docker
```

- 添加下面的内容

```
DOCKER_OPTS="--registry-mirror=https://registry.docker-cn.com"
```

- 重启 Docker 服务

```
$ sudo systemctl daemon-reload
$ sudo service docker restart
```

#### 0x02 导入、导出镜像

```
# 导出
$ docker save image_id > name_version.tar

# 导入
$ docker load < name_version.tar
```
