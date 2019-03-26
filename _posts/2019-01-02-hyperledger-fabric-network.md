---
layout: post
title: Hyperledger Fabric 网络搭建（二）
date: 2019-01-02 18:51:18
tags: [Hyperledger Fabric]
categories: [Hyperledger Fabric]
baseline:
---

### 0x01 规划组织结构

现阶段Fabric网络对于动态添加组织并不友好建议提前将组织规划好，再创建网络。

- church.org
- Orderer orderer.church.org
- Buddhism buddhism.church.org 2*peer 1*user
- Taoism taoism.church.org 2*peer 1*user

### 0x02 生成组织结构和身份证书
- 指定默认系统通道名 ChannelName 为 churchchannel

```
# 生成系统需要的证书文件
$ ./bin/cryptogen generate --config=./crypto-config.yaml
```

### 0x03 生成必要的网络启动配置文件

```
# 创建创世块 配置文件目录
$ mkdir channel-artifacts
# 生成创世块配置文件
$ ./bin/configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block
# 生成默认通道文件
$ ./bin/configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID churchchannel
# 生成组织 BuddhismMSP 锚点文件
$ ./bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/BuddhismMSPanchors.tx -channelID churchchannel -asOrg BuddhismMSP
# 生成组织 TaoismMSP 锚点文件
$ ./bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/TaoismMSPanchors.tx -channelID churchchannel -asOrg TaoismMSP
```

### 0x04 启动网络

```
# 指定docker网络项目名 用于后面指定链码部署docker网络
$ export COMPOSE_PROJECT_NAME=church
$ docker-compose -p $COMPOSE_PROJECT_NAME -f docker-compose-cli.yaml up -d

# 清理
$ docker-compose -p church -f docker-compose-cli.yaml down --volumes --remove-orphans
```

### 0x05 进入容器 来开始我们的操作

```
$ docker exec -it church_cli bash
# 指定连接orderer节点的tls证书
$ export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/church.org/orderers/orderer.church.org/msp/tlscacerts/tlsca.church.org-cert.pem && export CHANNEL_NAME=churchchannel
# 创建通道
$ peer channel create -o orderer.church.org:7050 -c $CHANNEL_NAME -t 50s -f ./channel-artifacts/channel.tx --tls --cafile $ORDERER_CA

# 组织 BuddhismMSP 加入通道
$ peer channel join -b churchchannel.block

# 验证下当前组织加入的通道
$ peer channel list

# 安装链码
$ peer chaincode install -n church -v 1.0  -p github.com/chaincode/chaincode_example02/go/
# docker exec peer0.buddhism.church.org ls /var/hyperledger/production/chaincode
# 实例化链码 初始化 a 100 b 100 积分 并指定背书策略
$ peer chaincode instantiate -o orderer.church.org:7050 --tls ${CORE_PEER_TLS_ENABLED} --cafile $ORDERER_CA -C $CHANNEL_NAME -n church -v 1.0 -c '{"Args":["init","a","10000","b","20000"]}' -P "OR('BuddhismMSP.member','TaoismMSP.member')"

# 查询下 ab的账户积分
$ peer chaincode query -C $CHANNEL_NAME -n church -c '{"Args":["query","a"]}'
$ peer chaincode query -C $CHANNEL_NAME -n church -c '{"Args":["query","b"]}'

# 执行转账操作 a 给 b 10
$ peer chaincode invoke -o orderer.church.org:7050  --tls ${CORE_PEER_TLS_ENABLED} --cafile $ORDERER_CA -C $CHANNEL_NAME -n church -c '{"Args":["invoke","a","b","10"]}'

### 切换到组织 TaoismMSP
$ export CORE_PEER_LOCALMSPID="TaoismMSP"
$ export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/taoism.church.org/peers/peer0.taoism.church.org/tls/ca.crt
$ export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/taoism.church.org/users/Admin@taoism.church.org/msp
$ export CORE_PEER_ADDRESS=peer0.taoism.church.org:7051

# 组织 TaoismMSP 加入通道
peer channel join -b churchchannel.block
peer channel list

# 安装链码
$ peer chaincode install -n church -v 1.0  -p github.com/chaincode/chaincode_example02/go/

# 执行转账链码 会自动初始化
peer chaincode invoke -o orderer.church.org:7050  --tls ${CORE_PEER_TLS_ENABLED} --cafile $ORDERER_CA -C $CHANNEL_NAME -n church -c '{"Args":["invoke","a","b","10"]}'

```
