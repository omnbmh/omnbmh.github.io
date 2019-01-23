---
layout: post
title: Hyperledger Fabric 动态添加组织（三）
date: 2019-01-03 18:51:18
tags: [Hyperledger Fabric]
categories: [Hyperledger Fabric]
baseline:
---
### 手动添加组织 LamaismMSP 到网络
#### 0x01 生成组织证书 基于`crypto-config-lamaism.yaml`

```
$ mkdir lamaism-org && cd lamaism-org/
$ ../bin/cryptogen generate --config=./crypto-config-lamaism.yaml

# 复制 orderer ca 到证书生成目录
$ cp -r crypto-config/ordererOrganizations lamaism-org/crypto-config/
```

#### 0x02 生成通道交易文件 基于 `configtx.yaml`

```
# 读取 configtx.yaml 生成 lamaism.json
$ ../bin/configtxgen -printOrg LamaismMSP > ../channel-artifacts/lamaism.json
```

#### 0x03 进入 church_cli 开始配置

```
$ docker exec -it church_cli bash

# 导入orderer证书和通道
$ export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/church.org/orderers/orderer.church.org/msp/tlscacerts/tlsca.church.org-cert.pem  && export CHANNEL_NAME=churchchannel

# 将二进制 protobuf 通道配置块保存到 `config_block.pb`
$ peer channel fetch config config_block.pb -o orderer.church.org:7050 -c $CHANNEL_NAME --tls --cafile $ORDERER_CA

# 将配置块 转换为 `config.json`
$ configtxlator proto_decode --input config_block.pb --type common.Block | jq .data.data[0].payload.data.config > config.json

# 添加 LamaismMSP 加密资料 添加到通道的应用程序组字段 `modified_config.json`
$ jq -s '.[0] * {"channel_group":{"groups":{"Application":{"groups": {"LamaismMSP":.[1]}}}}}' config.json ./channel-artifacts/lamaism.json > modified_config.json

# 现在 在cli里 我们得到了两个json文件 `config.json` `modified_config.json`
# 将 config.json 转换回 config.pb protobuf文件
$ configtxlator proto_encode --input config.json --type common.Config --output config.pb

# 将 modified_config.json 转换回 modified_config.pb
$ configtxlator proto_encode --input modified_config.json --type common.Config --output modified_config.pb

# 计算两个protobuf之间的增量 输出 lamaism_update.pb
$ configtxlator compute_update --channel_id $CHANNEL_NAME --original config.pb --updated modified_config.pb --output lamaism_update.pb

# 将 lamaism_update.pb 转换为 `lamaism_update.json`
$ configtxlator proto_decode --input lamaism_update.pb --type common.ConfigUpdate | jq . > lamaism_update.json

# 将标题加上 `lamaism_update_in_envelope.json`
$ echo '{"payload":{"header":{"channel_header":{"channel_id":"churchchannel", "type":2}},"data":{"config_update":'$(cat lamaism_update.json)'}}}' | jq . > lamaism_update_in_envelope.json

# 转换成 protobuf `lamaism_update_in_envelope.pb`
$ configtxlator proto_encode --input lamaism_update_in_envelope.json --type common.Envelope --output lamaism_update_in_envelope.pb

# 签名 并 提交配置更新 先让 BuddhismMSP 签名
$ peer channel signconfigtx -f lamaism_update_in_envelope.pb

# 切换到 TaoismMSP
$ export CORE_PEER_LOCALMSPID="TaoismMSP"

$ export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/taoism.church.org/peers/peer0.taoism.church.org/tls/ca.crt

$ export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/taoism.church.org/users/Admin@taoism.church.org/msp

$ export CORE_PEER_ADDRESS=peer0.taoism.church.org:7051

# 执行 update TaoismMSP将自动签名
$ peer channel update -f lamaism_update_in_envelope.pb -c $CHANNEL_NAME -o orderer.church.org:7050 --tls --cafile $ORDERER_CA
```

#### 0x04 启动 peer 节点

```
$ export COMPOSE_PROJECT_NAME=church
$ docker-compose -p church -f docker-compose-cli-lamaism.yaml up -d
```

#### 0x05 进入 lamaism_cli 继续接下来的配置

```
$ docker exec -it lamaism_cli bash

$ export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/church.org/orderers/orderer.church.org/msp/tlscacerts/tlsca.church.org-cert.pem  && export CHANNEL_NAME=churchchannel

$ echo $ORDERER_CA && echo $CHANNEL_NAME

$ peer channel fetch 0 churchchannel.block -o orderer.church.org:7050 -c $CHANNEL_NAME --tls --cafile $ORDERER_CA

# LamaismMSP 加入创世区块
$ peer channel join -b churchchannel.block

# 成功 升级 并 调用 Chaincode
$ peer chaincode install -n churchchannel -v 2.0 -p github.com/chaincode/chaincode_example02/go/
```

## 回到 church_cli
```
$ docker exec -it church_cli bash
$ peer chaincode install -n churchchannel -v 2.0 -p github.com/chaincode/chaincode_example02/go/

# 切换到 BuddhismMSP
$ export CORE_PEER_LOCALMSPID="BuddhismMSP"

$ export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/buddhism.church.org/peers/peer0.buddhism.church.org/tls/ca.crt

$ export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/buddhism.church.org/users/Admin@buddhism.church.org/msp

$ export CORE_PEER_ADDRESS=peer0.buddhism.church.org:7051

$ peer chaincode install -n churchchannel -v 2.0 -p github.com/chaincode/chaincode_example02/go/

# 升级链码
$ peer chaincode upgrade -o orderer.church.org:7050 --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA -C $CHANNEL_NAME -n churchchannel -v 2.0 -c '{"Args":["init","a","90","b","210"]}' -P "OR ('BuddhismMSP.peer','TaoismMSP.peer','LamaismMSP.peer')"

$ peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'
$ peer chaincode invoke -o orderer.church.org:7050  --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA -C $CHANNEL_NAME -n churchchannel -c '{"Args":["invoke","a","b","10"]}'

$ peer chaincode query -C $CHANNEL_NAME -n mycc -c '{"Args":["query","a"]}'

```

#### 0x06 完结！！！下面的使用 Fabric CA 来生成组织证书

### 基于 Fabric CA 生成的证书添加一个组织 ChristianityOrg

- 证书目录 `/app/fabric-network/christianity-org/crypto-config`

```
# 将 orderer 的证书复制到 组织证书目录内
$ cp -r crypto-config/ordererOrganizations lamaism-org/crypto-config/
```

#### 0x01 添加配置文件 configtx.yaml
#### 0x02

```
../bin/configtxgen -printOrg ChristianityMSP > ../channel-artifacts/christianity.json
```

#### 0x03 进入 BuddhismMSP 客户端

```
$ docker exec -it church_cli bash
$ peer channel list

$ export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/church.org/orderers/orderer.church.org/msp/tlscacerts/tlsca.church.org-cert.pem  && export CHANNEL_NAME=churchchannel

peer channel fetch config config_block.pb -o orderer.church.org:7050 -c $CHANNEL_NAME --tls --cafile $ORDERER_CA

configtxlator proto_decode --input config_block.pb --type common.Block | jq .data.data[0].payload.data.config > config.json

# 添加 ChristianityMSP 加密资料 添加到通道的应用程序组字段 `modified_config.json`
$ jq -s '.[0] * {"channel_group":{"groups":{"Application":{"groups": {"ChristianityMSP":.[1]}}}}}' config.json ./channel-artifacts/christianity.json > modified_config.json

# 现在 在cli里 我们得到了两个json文件 `config.json` `modified_config.json`
# 将 config.json 转换回 config.pb protobuf文件
configtxlator proto_encode --input config.json --type common.Config --output config.pb

# 将 modified_config.json 转换回 modified_config.pb
$ configtxlator proto_encode --input modified_config.json --type common.Config --output modified_config.pb

# 计算两个protobuf之间的增量 输出 christianity_update.pb
$ configtxlator compute_update --channel_id $CHANNEL_NAME --original config.pb --updated modified_config.pb --output christianity_update.pb

# 将 christianity_update.pb 转换为 `christianity_update.json`
configtxlator proto_decode --input christianity_update.pb --type common.ConfigUpdate | jq . > christianity_update.json

# 将标题加上 `christianity_update_in_envelope.json`
echo '{"payload":{"header":{"channel_header":{"channel_id":"churchchannel", "type":2}},"data":{"config_update":'$(cat christianity_update.json)'}}}' | jq . > christianity_update_in_envelope.json

转换成 protobuf `christianity_update_in_envelope.pb`
configtxlator proto_encode --input christianity_update_in_envelope.json --type common.Envelope --output christianity_update_in_envelope.pb

签名 并 提交配置更新 先让 BuddhismMSP 签名
peer channel signconfigtx -f christianity_update_in_envelope.pb

切换到 TaoismMSP
export CORE_PEER_LOCALMSPID="TaoismMSP"

export CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/taoism.church.org/peers/peer0.taoism.church.org/tls/ca.crt

export CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/taoism.church.org/users/Admin@taoism.church.org/msp

export CORE_PEER_ADDRESS=peer0.taoism.church.org:7051

# 执行 update TaoismMSP将自动签名
$ peer channel update -f christianity_update_in_envelope.pb -c $CHANNEL_NAME -o orderer.church.org:7050 --tls --cafile $ORDERER_CA





```

#### 0x04 启动 ChristianityMSP 新的peer节点
```
$ export COMPOSE_PROJECT_NAME=church
$ docker-compose -p church -f docker-compose-cli-lamaism.yaml up -d
```

#### 0x05 安装链码

```
$ docker exec -it christianity_cli bash

export ORDERER_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/church.org/orderers/orderer.church.org/msp/tlscacerts/tlsca.church.org-cert.pem  && export CHANNEL_NAME=churchchannel

# 获取通道块信息
peer channel fetch 0 churchchannel.block -o orderer.church.org:7050 -c $CHANNEL_NAME --tls --cafile $ORDERER_CA

# 加入通道
peer channel join -b churchchannel.block

# 安装链码
peer chaincode install -n church -v 1.0 -p github.com/chaincode/chaincode_example02/go/

# 实例化链码 通道上链码 只需实例化一次
peer chaincode instantiate -o orderer.church.org:7050 --tls ${CORE_PEER_TLS_ENABLED} --cafile $ORDERER_CA -C $CHANNEL_NAME -n church -v 1.0 -c '{"Args":["init","a","100","b","200"]}' -P "OR('BuddhismMSP.member','TaoismMSP.member','LamaismMSP.member','ChristianityMSP.member')"

peer chaincode query -C churchchannel -n church -c '{"Args":["query","a"]}'

peer chaincode invoke -o orderer.church.org:7050  --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA -C $CHANNEL_NAME -n church -c '{"Args":["invoke","a","b","10"]}'

```
