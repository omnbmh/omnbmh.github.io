---
layout: post
title: Hyperledger Fabric 常用命令（五）
date: 2019-01-05 18:51:18
tags: [Hyperledger Fabric]
categories: [Hyperledger Fabric]
baseline:
---

# Hyperledger Fabric 常用命令（五）

#### 查询节点上链码列表
docker exec peer0.buddhism.church.org ls /var/hyperledger/production/chaincodes

#### 查询通道上的信息
peer channel getinfo -c churchchannel
