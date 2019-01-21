---
layout: post
title: Hyperledger Fabric 网络搭建（二）
date: 2019-01-02 18:51:18
tags: [Hyperledger Fabric]
categories: [Hyperledger Fabric]
baseline:
---

# Hyperledger Fabric 网络搭建（二）

### 0x01 规划组织结构
- church.org
- Orderer orderer.church.org
- Buddhism buddhism.church.org 2*peer 1*user
- Taoism taoism.church.org 2*peer 1*user

### 0x01 生成组织结构和身份证书
- ChannelName churchchannel
```
$ ./bin/cryptogen generate --config=./crypto-config.yaml

$ mkdir channel-artifacts
$ ./bin/configtxgen -profile TwoOrgsOrdererGenesis -outputBlock ./channel-artifacts/genesis.block
$ ./bin/configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/channel.tx -channelID churchchannel
$ ./bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/BuddhismMSPanchors.tx -channelID churchchannel -asOrg BuddhismMSP
$ ./bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/TaoismMSPanchors.tx -channelID churchchannel -asOrg TaoismMSP

```
### 0x02 启动网络
```
export COMPOSE_PROJECT_NAME=church
docker-compose -p church -f docker-compose-cli.yaml up -d
```

### 进入容器
docker exec -it church_cli bash

### 0x03 创建通道
export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/church.org/orderers/orderer.church.org/msp/tlscacerts/tlsca.church.org-cert.pem

peer channel create -o orderer.church.org:7050 -c churchchannel -t 50s -f ./channel-artifacts/channel.tx --tls --cafile $ORDERER_CA

docker exec peer0.buddhism.church.org ls /var/hyperledger/production/chaincode

加入通道
peer channel join -b churchchannel.block
peer channel list

peer chaincode instantiate -o orderer.church.org:7050 --tls ${CORE_PEER_TLS_ENABLED} --cafile $ORDERER_CA -C churchchannel -n churchchannel -v 1.0 -c '{"Args":["init","a","100","b","200"]}' -P "OR('BuddhismMSP.member','TaoismMSP.member')"


peer chaincode install -n churchchannel -v 2.0  -p github.com/chaincode/chaincode_example02/go/

 peer chaincode instantiate -o orderer.church.org:7050 --tls ${CORE_PEER_TLS_ENABLED} --cafile $ORDERER_CA -C churchchannel -n churchchannel -v 2.0 -c '{"Args":["init","a","100","b","200"]}' -P "OR('BuddhismMSP.member','TaoismMSP.member')"

转账
peer chaincode invoke -o orderer.church.org:7050  --tls --cafile $ORDERER_CA -C $CHANNEL_NAME -n churchchannel -c '{"Args":["invoke","a","b","10"]}'


### 切换到 TaoismMSP 加入通道
export CORE_PEER_LOCALMSPID="TaoismMSP"

export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/taoism.church.org/peers/peer0.taoism.church.org/tls/ca.crt

export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/taoism.church.org/users/Admin@taoism.church.org/msp

export CORE_PEER_ADDRESS=peer0.taoism.church.org:7051

加入通道
peer channel join -b churchchannel.block
peer channel list
