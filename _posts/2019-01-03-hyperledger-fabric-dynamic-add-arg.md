---
layout: post
title: Hyperledger Fabric 动态添加组织（三）
date: 2018-01-03 18:51:18
tags: [Hyperledger Fabric]
categories: [Hyperledger Fabric]
baseline:
---

# Hyperledger Fabric 动态添加组织（三）

### 0x01 添加配置文件

### 0x02 生成证书
```
$ cd lamaism-org/
$ ../bin/cryptogen generate --config=./crypto-config-lamaism.yaml
```
### 0x03 通道交易
读取 configtx.yaml 生成 lamaism.json
../bin/configtxgen -printOrg LamaismMSP > ../channel-artifacts/lamaism.json

复制orderer ca 到生成目录
cp -r crypto-config/ordererOrganizations lamaism-org/crypto-config/

### 0x04 进入 cli 开始配置
docker exec -it church_cli bash

export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/church.org/orderers/orderer.church.org/msp/tlscacerts/tlsca.church.org-cert.pem  && export CHANNEL_NAME=churchchannel

命令将二进制 protobuf 通道配置块保存到 `config_block.pb`
$ peer channel fetch config config_block.pb -o orderer.church.org:7050 -c $CHANNEL_NAME --tls --cafile $ORDERER_CA

将配置块 转换为 `config.json`
$ configtxlator proto_decode --input config_block.pb --type common.Block | jq .data.data[0].payload.data.config > config.json

添加 LamaismMSP 加密资料 添加到通道的应用程序组字段 `modified_config.json`
jq -s '.[0] * {"channel_group":{"groups":{"Application":{"groups": {"LamaismMSP":.[1]}}}}}' config.json ./channel-artifacts/lamaism.json > modified_config.json

现在 在cli里 我们得到了两个json文件 `config.json` `modified_config.json`
将 config.json 转换回 config.pb protobuf文件
configtxlator proto_encode --input config.json --type common.Config --output config.pb

将 modified_config.json 转换回 modified_config.pb
configtxlator proto_encode --input modified_config.json --type common.Config --output modified_config.pb

计算两个protobuf之间的增量 输出 lamaism_update.pb
configtxlator compute_update --channel_id $CHANNEL_NAME --original config.pb --updated modified_config.pb --output lamaism_update.pb

将 lamaism_update.pb 转换为 `lamaism_update.json`
configtxlator proto_decode --input lamaism_update.pb --type common.ConfigUpdate | jq . > lamaism_update.json

将标题加上 `lamaism_update_in_envelope.json`
echo '{"payload":{"header":{"channel_header":{"channel_id":"churchchannel", "type":2}},"data":{"config_update":'$(cat lamaism_update.json)'}}}' | jq . > lamaism_update_in_envelope.json

转换成 protobuf `lamaism_update_in_envelope.pb`
configtxlator proto_encode --input lamaism_update_in_envelope.json --type common.Envelope --output lamaism_update_in_envelope.pb

签名 并 提交配置更新 先让 BuddhismMSP 签名
peer channel signconfigtx -f lamaism_update_in_envelope.pb

切换到 TaoismMSP
export CORE_PEER_LOCALMSPID="TaoismMSP"

export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/taoism.church.org/peers/peer0.taoism.church.org/tls/ca.crt

export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/taoism.church.org/users/Admin@taoism.church.org/msp

export CORE_PEER_ADDRESS=peer0.taoism.church.org:7051

执行 update TaoismMSP将自动签名
peer channel update -f lamaism_update_in_envelope.pb -c $CHANNEL_NAME -o orderer.church.org:7050 --tls --cafile $ORDERER_CA


#### 启动 peer
export COMPOSE_PROJECT_NAME=church
docker-compose -p church -f docker-compose-cli-lamaism.yaml up -d

进入 lamaism_cli
docker exec -it lamaism_cli bash

export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/church.org/orderers/orderer.church.org/msp/tlscacerts/tlsca.church.org-cert.pem  && export CHANNEL_NAME=churchchannel

echo $ORDERER_CA && echo $CHANNEL_NAME

peer channel fetch 0 churchchannel.block -o orderer.church.org:7050 -c $CHANNEL_NAME --tls --cafile $ORDERER_CA

LamaismMSP 加入创世区块
peer channel join -b churchchannel.block

成功 升级 并 调用 Chaincode
peer chaincode install -n churchchannel -v 2.0 -p github.com/chaincode/chaincode_example02/go/

#### 回到 church_cli
peer chaincode install -n churchchannel -v 2.0 -p github.com/chaincode/chaincode_example02/go/


切换到 BuddhismMSP
export CORE_PEER_LOCALMSPID="BuddhismMSP"

export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/buddhism.church.org/peers/peer0.buddhism.church.org/tls/ca.crt

export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/buddhism.church.org/users/Admin@buddhism.church.org/msp

export CORE_PEER_ADDRESS=peer0.buddhism.church.org:7051

peer chaincode install -n churchchannel -v 2.0 -p github.com/chaincode/chaincode_example02/go/

升级链码
peer chaincode upgrade -o orderer.church.org:7050 --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA -C $CHANNEL_NAME -n churchchannel -v 2.0 -c '{"Args":["init","a","90","b","210"]}' -P "OR ('BuddhismMSP.peer','TaoismMSP.peer','LamaismMSP.peer')"

peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
peer chaincode invoke -o orderer.church.org:7050  --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA -C $CHANNEL_NAME -n churchchannel -c '{"Args":["invoke","a","b","10"]}'

peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
