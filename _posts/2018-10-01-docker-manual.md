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
$ docker load -i name_version.tar
```

#### 0x03 更换 docker 储存位置

```
# 停服务
service docker stop

mv -f /var/lib/docker /opt/docker_data
ln -s /opt/docker_data /var/lib/docker

service docker start
```

### Docker Swarm

```
# 服务监控
docker run -d --name visualizer --rm -p 8990:8080 -v /var/run/docker.sock:/var/run/docker.sock dockersamples/visualizer
```
```
# docker config 共享文件
```

```
# 使用 compose 文件
```

#### 服务名是域名的时候 无法进行通信 测试一下

启动三个服务 分别使用 a.btc.io b.btc.io c.btc.io

```
# docker-compose.yaml
version: '3'
networks:
  btc_network:

services:
  a.btc.io:
    image: alpine:latest
    command: ping baidu.com
    networks:
      - btc_network
  b.btc.io:
    image: alpine:latest
    command: ping baidu.com
    networks:
      - btc_network
  c.btc.io:
    image: alpine:latest
    command: ping baidu.com
    networks:
      - btc_network
```

```
# 创建服务
docker stack deploy aaa -c docker-compose-test-domain.yaml
```

经过测试 真的不行  可以使用  aliases 相当于当前服务的别名
```
networks:
  btc_network:
    aliases:
      - c.btc.io
```
