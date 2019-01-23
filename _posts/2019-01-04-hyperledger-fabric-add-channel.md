---
layout: post
title: Hyperledger Fabric 动态添加通道（四）
date: 2018-01-04 18:51:18
tags: [Hyperledger Fabric]
categories: [Hyperledger Fabric]
baseline:
---

export CHANNEL_NAME=carchannel
./bin/configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/carchannel.tx -channelID $CHANNEL_NAME
$ ./bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/BuddhismMSPanchorsCar.tx -channelID $CHANNEL_NAME -asOrg BuddhismMSP
$ ./bin/configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate ./channel-artifacts/TaoismMSPanchorsCar.tx -channelID $CHANNEL_NAME -asOrg TaoismMSP

docker exec -it church_cli bash

export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/church.org/orderers/orderer.church.org/msp/tlscacerts/tlsca.church.org-cert.pem

peer channel create -o orderer.church.org:7050 -c $CHANNEL_NAME -t 50s -f ./channel-artifacts/carchannel.tx --tls --cafile $ORDERER_CA

 peer channel join -b carchannel.block


#### 安装链码
Buddhism Taoism 都安装了链码  在 peer0
peer chaincode install -n carcc -v 1.0 -p github.com/chaincode/fabcar/go/

peer chaincode instantiate -o orderer.church.org:7050 --tls ${CORE_PEER_TLS_ENABLED} --cafile $ORDERER_CA -C carchannel -n carcc -v 1.0 -c '{"Args":["init"]}' -P "OR('BuddhismMSP.member','TaoismMSP.member')"

peer chaincode invoke -o orderer.church.org:7050 -C carchannel -n carcc -c '{"function":"initLedger","Args":[""]}'
