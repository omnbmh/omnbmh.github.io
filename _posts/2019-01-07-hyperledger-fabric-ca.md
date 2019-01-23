---
layout: post
title: Hyperledger Fabric CA（七）
date: 2019-01-07 18:51:18
tags: [Hyperledger Fabric]
categories: [Hyperledger Fabric]
baseline:
---

Hyperledger Fabric CA 是 Hyperledger Fabric 的证书颁发机构，是一个可选的MemberService组件，对网络中各个实体的身份证书进行管理。

Fabric CA 可以以多层次的结构部署。

#### 环境要求
-

#### 常用命令


```
# 生成 church.org
./bin/fabric-ca-client enroll -M ./crypto-config/ordererOrganizations/church.org/msp -u http://admin:adminpw@10.143.143.197:10054 --home ca_church_org

# 添加组织
./bin/fabric-ca-client affiliation list -M ./crypto-config/ordererOrganizations/church.org/msp -u http://admin:adminpw@10.143.143.197:10054 --home ca_church_org

affiliation: .
   affiliation: org1
      affiliation: org1.department1
      affiliation: org1.department2
   affiliation: org2
      affiliation: org2.department1

./bin/fabric-ca-client affiliation remove org1 --force -M ./crypto-config/ordererOrganizations/church.org/msp -u http://admin:adminpw@10.143.143.197:10054 --home ca_church_org

./bin/fabric-ca-client affiliation remove org2 --force -M ./crypto-config/ordererOrganizations/church.org/msp -u http://admin:adminpw@10.143.143.197:10054 --home ca_church_org

./bin/fabric-ca-client affiliation add org -M ./crypto-config/ordererOrganizations/church.org/msp -u http://admin:adminpw@10.143.143.197:10054 --home ca_church_org

./bin/fabric-ca-client affiliation add org.church -M ./crypto-config/ordererOrganizations/church.org/msp -u http://admin:adminpw@10.143.143.197:10054 --home ca_church_org


# 生成 Admin@church.org 密码123456
注册
./bin/fabric-ca-client register --id.name Admin@church.org --id.type client --id.affiliation "org.church" --id.attrs '"hf.Registrar.Roles=client,orderer,peer,user","hf.Registrar.DelegateRoles=client,orderer,peer,user",hf.Registrar.Attributes=*,hf.GenCRL=true,hf.Revoker=true,hf.AffiliationMgr=true,hf.IntermediateCA=true,role=admin:ecert' --id.secret=123456 --csr.cn=church.org --csr.hosts=['church.org'] -M ./crypto-config/ordererOrganizations/church.org/msp -u http://admin:adminpw@10.143.143.197:10054 --home ca_church_org

登记
./bin/fabric-ca-client enroll -u http://Admin@church.org:123456@10.143.143.197:10054 --csr.cn=church.org --csr.hosts=['church.org'] -M ./crypto-config/ordererOrganizations/church.org/users/Admin@church.org/msp --home ./ca_church_org


# 生成 msp
mkdir ca_church_org/crypto-config/ordererOrganizations/church.org/users/Admin@church.org/msp/admincerts

cp ca_church_org/crypto-config/ordererOrganizations/church.org/users/Admin@church.org/msp/signcerts/cert.pem ca_church_org/crypto-config/ordererOrganizations/church.org/users/Admin@church.org/msp/admincerts


mkdir ca_church_org/crypto-config/ordererOrganizations/church.org/msp/admincerts

cp  ca_church_org/crypto-config/ordererOrganizations/church.org/msp/signcerts/cert.pem ca_church_org/crypto-config/ordererOrganizations/church.org/msp/admincerts

# 生成orderer.church.org的msp
注册
./bin/fabric-ca-client register --id.name orderer.church.org --id.type orderer --id.affiliation "org.church" --id.attrs '"role=orderer",ecert=true' --id.secret=123456 --csr.cn=orderer.church.org --csr.hosts=['orderer.church.org'] -u http://Admin@church.org:123456@10.143.143.197:10054 --home ca_church_org

登记
./bin/fabric-ca-client enroll -u http://Admin@church.org:123456@10.143.143.197:10054 --csr.cn=orderer.church.org --csr.hosts=['orderer.church.org'] -M ./crypto-config/ordererOrganizations/church.org/orderers/orderer.church.org/msp --home ca_church_org

生成msp
mkdir ca_church_org/crypto-config/ordererOrganizations/church.org/orderers/orderer.church.org/msp/admincerts
cp ca_church_org/crypto-config/ordererOrganizations/church.org/orderers/orderer.church.org/msp/signcerts/cert.pem ca_church_org/crypto-config/ordererOrganizations/church.org/orderers/orderer.church.org/msp/admincerts


生成 TLS 证书
生成 church.org 的TLS
登记
./bin/fabric-ca-client enroll -M ./crypto-config/ordererOrganizations/church.org/tls -u http://admin:adminpw@10.143.143.197:20054 --home ca_church_org
添加联盟成员 同上
./bin/fabric-ca-client affiliation add org -M ./crypto-config/ordererOrganizations/church.org/tls -u http://admin:adminpw@10.143.143.197:20054 --home ca_church_org

./bin/fabric-ca-client affiliation add org.church -M ./crypto-config/ordererOrganizations/church.org/tls -u http://admin:adminpw@10.143.143.197:20054 --home ca_church_org


生成 Admin@church.org 的 TLS
./bin/fabric-ca-client register --id.name Admin@church.org --id.type client --id.affiliation "org.church" --id.attrs '"hf.Registrar.Roles=client,orderer,peer,user","hf.Registrar.DelegateRoles=client,orderer,peer,user",hf.Registrar.Attributes=*,hf.GenCRL=true,hf.Revoker=true,hf.AffiliationMgr=true,hf.IntermediateCA=true,role=admin:ecert' --id.secret=123456 --csr.cn=church.org --csr.hosts=['church.org'] -M ./crypto-config/ordererOrganizations/church.org/tls -u http://admin:adminpw@10.143.143.197:20054 --home ca_church_org

登记
./bin/fabric-ca-client enroll -d --enrollment.profile tls -u http://Admin@church.org:123456@10.143.143.197:20054 --csr.cn=church.org --csr.hosts=['church.org'] -M ./crypto-config/ordererOrganizations/church.org/users/Admin@church.org/tls --home ca_church_org
生成 TLS
cp ca_church_org/crypto-config/ordererOrganizations/church.org/users/Admin@church.org/tls/tlsintermediatecerts/tls-10-143-143-197-20054.pem ca_church_org/crypto-config/ordererOrganizations/church.org/users/Admin@church.org/tls/ca.crt

cp ca_church_org/crypto-config/ordererOrganizations/church.org/users/Admin@church.org/tls/signcerts/cert.pem ca_church_org/crypto-config/ordererOrganizations/church.org/users/Admin@church.org/tls/client.crt

cp ca_church_org/crypto-config/ordererOrganizations/church.org/users/Admin@church.org/tls/keystore/624d6502f7b7492ebc4ae197262493528c9f4408ff7ab63f5d9441a9d64f2751_sk ca_church_org/crypto-config/ordererOrganizations/church.org/users/Admin@church.org/tls/client.key

生成 orderer.church.org 的 TLS
注册
./bin/fabric-ca-client register --id.name orderer.church.org --id.type orderer --id.affiliation "org.church" --id.attrs '"role=orderer",ecert=true' --id.secret=123456 --csr.cn=orderer.church.org --csr.hosts=['orderer.church.org'] -M ./crypto-config/ordererOrganizations/church.org/tls -u http://admin:adminpw@10.143.143.197:20054 --home ca_church_org

登记
./bin/fabric-ca-client enroll -d --enrollment.profile tls -u http://orderer.church.org:123456@10.143.143.197:20054 --csr.cn=orderer.church.org --csr.hosts=['orderer.church.org'] -M ./crypto-config/ordererOrganizations/church.org/orderers/orderer.church.org/tls --home ca_church_org

生成 TLS
$ cp ca_church_org/crypto-config/ordererOrganizations/church.org/orderers/orderer.church.org/tls/tlsintermediatecerts/tls-10-143-143-197-20054.pem ca_church_org/crypto-config/ordererOrganizations/church.org/orderers/orderer.church.org/tls/ca.crt

$ cp ca_church_org/crypto-config/ordererOrganizations/church.org/orderers/orderer.church.org/tls/signcerts/cert.pem ca_church_org/crypto-config/ordererOrganizations/church.org/orderers/orderer.church.org/tls/server.crt

$ cp ca_church_org/crypto-config/ordererOrganizations/church.org/orderers/orderer.church.org/tls/keystore/4691b264587f6915f13755283b7dc7553249f936f9e3d8f58e8cdcc42a1177c8_sk ca_church_org/crypto-config/ordererOrganizations/church.org/orderers/orderer.church.org/tls/server.key

# 开始生成Peer节点 证书
```


### 使用 Fabric CA 生成 组织 christianity.church.org 的证书
启动 CA Server 节点
$ sudo docker-compose -p fabric -f docker-compose-christianity-ca.yaml up -d

登记 christianity.church.org
$ ./bin/fabric-ca-client enroll --csr.cn=christianity.church.org --csr.hosts=['christianity.church.org'] -M ./crypto-config/peerOrganizations/christianity.church.org/msp -u http://admin:adminpw@10.143.143.197:50054 --home christianity-org

添加联盟成员
查看下
$ ./bin/fabric-ca-client affiliation list -M ./crypto-config/peerOrganizations/christianity.church.org/msp -u http://admin:adminpw@10.143.143.197:50054 --home christianity-org

$ ./bin/fabric-ca-client affiliation add org -M ./crypto-config/peerOrganizations/christianity.church.org/msp -u http://admin:adminpw@10.143.143.197:50054 --home christianity-org

$ ./bin/fabric-ca-client affiliation add org.church -M ./crypto-config/peerOrganizations/christianity.church.org/msp -u http://admin:adminpw@10.143.143.197:50054 --home christianity-org

$ ./bin/fabric-ca-client affiliation add org.church.christianity -M ./crypto-config/peerOrganizations/christianity.church.org/msp -u http://admin:adminpw@10.143.143.197:50054 --home christianity-org

### 生成 Admin@christianity.church.org 的MSP
#### 注册 Admin@christianity.church.org

```
$ ./bin/fabric-ca-client register --id.name Admin@christianity.church.org --id.type client --id.affiliation "org.church.christianity" --id.attrs '"hf.Registrar.Roles=client,orderer,peer,user","hf.Registrar.DelegateRoles=client,orderer,peer,user",hf.Registrar.Attributes=*,hf.GenCRL=true,hf.Revoker=true,hf.AffiliationMgr=true,hf.IntermediateCA=true,role=admin:ecert' --id.secret=123456 --csr.cn=christianity.church.org --csr.hosts=['christianity.church.org'] -M ./crypto-config/peerOrganizations/christianity.church.org/msp -u http://admin:adminpw@10.143.143.197:50054 --home christianity-org
```

#### 登记 Admin@christianity.church.org

```
$ ./bin/fabric-ca-client enroll -u http://Admin@christianity.church.org:123456@10.143.143.197:50054 --csr.cn=christianity.church.org --csr.hosts=['christianity.church.org'] -M ./crypto-config/peerOrganizations/christianity.church.org/users/Admin@christianity.church.org/msp --home christianity-org
```

#### 生成
```
$ mkdir christianity-org/crypto-config/peerOrganizations/christianity.church.org/users/Admin@christianity.church.org/msp/admincerts

$ cp christianity-org/crypto-config/peerOrganizations/christianity.church.org/users/Admin@christianity.church.org/msp/signcerts/cert.pem christianity-org/crypto-config/peerOrganizations/christianity.church.org/users/Admin@christianity.church.org/msp/admincerts

$ mkdir christianity-org/crypto-config/peerOrganizations/christianity.church.org/msp/admincerts

$ cp christianity-org/crypto-config/peerOrganizations/christianity.church.org/users/Admin@christianity.church.org/msp/signcerts/cert.pem christianity-org/crypto-config/peerOrganizations/christianity.church.org/msp/admincerts
```

### 生成 peer0.christianity.church.org 的MSP
#### 注册 peer0.christianity.church.org

```
$ ./bin/fabric-ca-client register --id.name peer0.christianity.church.org --id.type peer --id.affiliation "org.church.christianity" --id.attrs '"role=peer",ecert=true' --id.secret=123456 --csr.cn=peer0.christianity.church.org --csr.hosts=['christianity.church.org'] -M ./crypto-config/peerOrganizations/christianity.church.org/msp -u http://admin:adminpw@10.143.143.197:50054 --home christianity-org
```

#### 登记 peer0.christianity.church.org

```
$ ./bin/fabric-ca-client enroll -u http://peer0.christianity.church.org:123456@10.143.143.197:50054 --csr.cn=peer0.christianity.church.org --csr.hosts=['peer0.christianity.church.org'] -M ./crypto-config/peerOrganizations/christianity.church.org/peers/peer0.christianity.church.org/msp --home christianity-org
```

#### 生成
```
$ mkdir ./christianity-org/crypto-config/peerOrganizations/christianity.church.org/peers/peer0.christianity.church.org/msp/admincerts

$ cp christianity-org/crypto-config/peerOrganizations/christianity.church.org/users/Admin@christianity.church.org/msp/signcerts/cert.pem christianity-org/crypto-config/peerOrganizations/christianity.church.org/peers/peer0.christianity.church.org/msp/admincerts
```

### 使用 tlsca 的登记
$ ./bin/fabric-ca-client enroll --csr.cn=christianity.church.org --csr.hosts=['christianity.church.org'] -M ./crypto-config/peerOrganizations/christianity.church.org/tls -u http://admin:adminpw@10.143.143.197:51054 --home christianity-org

$ ./bin/fabric-ca-client affiliation add org -M ./crypto-config/peerOrganizations/christianity.church.org/tls -u http://admin:adminpw@10.143.143.197:51054 --home christianity-org

$ ./bin/fabric-ca-client affiliation add org.church -M ./crypto-config/peerOrganizations/christianity.church.org/tls -u http://admin:adminpw@10.143.143.197:51054 --home christianity-org

$ ./bin/fabric-ca-client affiliation add org.church.christianity -M ./crypto-config/peerOrganizations/christianity.church.org/tls -u http://admin:adminpw@10.143.143.197:51054 --home christianity-org

### 生成 Admin@christianity.church.org 的TLS
#### 注册 Admin@christianity.church.org

```
$ ./bin/fabric-ca-client register --id.name Admin@christianity.church.org --id.type client --id.affiliation "org.church.christianity" --id.attrs '"hf.Registrar.Roles=client,orderer,peer,user","hf.Registrar.DelegateRoles=client,orderer,peer,user",hf.Registrar.Attributes=*,hf.GenCRL=true,hf.Revoker=true,hf.AffiliationMgr=true,hf.IntermediateCA=true,role=admin:ecert' --id.secret=123456 --csr.cn=christianity.church.org --csr.hosts=['christianity.church.org'] -M ./crypto-config/peerOrganizations/christianity.church.org/tls -u http://admin:adminpw@10.143.143.197:51054 --home christianity-org
```

#### 登记 Admin@christianity.church.org

```
$ ./bin/fabric-ca-client enroll -d --enrollment.profile tls -u http://Admin@christianity.church.org:123456@10.143.143.197:51054 --csr.cn=christianity.church.org --csr.hosts=['christianity.church.org'] -M ./crypto-config/peerOrganizations/christianity.church.org/users/Admin@christianity.church.org/tls --home christianity-org
```

#### 生成

```
$ cp ./christianity-org/crypto-config/peerOrganizations/christianity.church.org/users/Admin@christianity.church.org/tls/tlsintermediatecerts/tls-10-143-143-197-51054.pem ./christianity-org/crypto-config/peerOrganizations/christianity.church.org/users/Admin@christianity.church.org/tls/ca.crt

$ cp ./christianity-org/crypto-config/peerOrganizations/christianity.church.org/users/Admin@christianity.church.org/tls/signcerts/cert.pem ./christianity-org/crypto-config/peerOrganizations/christianity.church.org/users/Admin@christianity.church.org/tls/client.crt

$ cp ./christianity-org/crypto-config/peerOrganizations/christianity.church.org/users/Admin@christianity.church.org/tls/keystore/7964e07ea7f1370d8b9a5ae5d370f7bf69aa3b2f01fb284928a4838a74cb72f3_sk ./christianity-org/crypto-config/peerOrganizations/christianity.church.org/users/Admin@christianity.church.org/tls/client.key

```

### 生成 peer0.christianity.church.org 的TLS
#### 注册 peer0.christianity.church.org

```
$ ./bin/fabric-ca-client register --id.name peer0.christianity.church.org --id.type peer --id.affiliation "org.church.christianity" --id.attrs '"role=peer",ecert=true' --id.secret=123456 --csr.cn=peer0.christianity.church.org --csr.hosts=['christianity.church.org'] -M ./crypto-config/peerOrganizations/christianity.church.org/tls -u http://admin:adminpw@10.143.143.197:51054 --home christianity-org
```

#### 登记 peer0.christianity.church.org

```
$ ./bin/fabric-ca-client enroll -d --enrollment.profile tls -u http://peer0.christianity.church.org:123456@10.143.143.197:51054 --csr.cn=peer0.christianity.church.org --csr.hosts=['peer0.christianity.church.org'] -M ./crypto-config/peerOrganizations/christianity.church.org/peers/peer0.christianity.church.org/tls --home christianity-org
```

#### 生成

```
$ cp ./christianity-org/crypto-config/peerOrganizations/christianity.church.org/peers/peer0.christianity.church.org/tls/tlsintermediatecerts/tls-10-143-143-197-51054.pem ./christianity-org/crypto-config/peerOrganizations/christianity.church.org/peers/peer0.christianity.church.org/tls/ca.crt

$ cp ./christianity-org/crypto-config/peerOrganizations/christianity.church.org/peers/peer0.christianity.church.org/tls/signcerts/cert.pem ./christianity-org/crypto-config/peerOrganizations/christianity.church.org/peers/peer0.christianity.church.org/tls/server.crt

$ cp ./christianity-org/crypto-config/peerOrganizations/christianity.church.org/peers/peer0.christianity.church.org/tls/keystore/7bf6147801728a52ca24ed84bcceaa7f3f46e65fb31806fe797cd070bec24261_sk ./christianity-org/crypto-config/peerOrganizations/christianity.church.org/peers/peer0.christianity.church.org/tls/server.key

```



初始化时 调用签约

mkdir fabric-ca-root
/app/go/bin/fabric-ca-server init -b admin:adminpw  --home fabric-ca-root
/app/go/bin/fabric-ca-server start -b admin:adminpw  --home fabric-ca-root

mkdir fabric-ca-org1
/app/go/bin/fabric-ca-server init -b org1:org1pwd -u http://admin:adminpw@127.0.0.1:7054 --home fabric-ca-org1
/app/go/bin/fabric-ca-server start -b org1:org1pwd -u http://admin:adminpw@127.0.0.1:7054 --home fabric-ca-org1

mkdir fabric-ca-org2
/app/go/bin/fabric-ca-server init -b org1:org2pwd -u http://admin:adminpw@127.0.0.1:7054 --home fabric-ca-org2
/app/go/bin/fabric-ca-server start -b org1:org2pwd -u http://admin:adminpw@127.0.0.1:7054 --home fabric-ca-org2

mkdir fabric-ca-org2-bj
/app/go/bin/fabric-ca-server init -b org2bj:org2bjpwd -u http://org1:org2pwd@127.0.0.1:9054 --home fabric-ca-org2-bj
/app/go/bin/fabric-ca-server start -b org2bj:org2bjpwd -u http://org1:org2pwd@127.0.0.1:9054 --home fabric-ca-org2-bj


mkdir fabric-ca-org3
/app/go/bin/fabric-ca-server init -b org3:org3pwd -u http://admin:adminpw@127.0.0.1:7054 --home fabric-ca-org3
/app/go/bin/fabric-ca-server start -p 10054 -b org3:org3pwd -u http://admin:adminpw@127.0.0.1:7054 --home fabric-ca-org3
