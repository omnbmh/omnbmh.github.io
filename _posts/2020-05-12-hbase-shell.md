---
layout: post
title: 使用 Hbase Shell
date: 2020-05-12 15:15:15
categories: [Hbase]
tags: [Hbase,Shell]
---

下载 hbase-1.6.0-bin.tar.gz 链接:[https://pan.baidu.com/s/1kk4ZaICYNfCsVrl0mpn8XQ](https://pan.baidu.com/s/1kk4ZaICYNfCsVrl0mpn8XQ)  密码:e6dd

### 客户端配置

需要修改两个配置文件

- 修改配置文件 conf/hbase-env.sh
```shell
export JAVA_HOME=/path_to/jdk
```

- 修改配置文件 conf/hbase-site.xml
```xml
<configuration>
<property>  
  <name>hbase.zookeeper.property.clientPort</name>  
  <value>2181</value>  
</property>
<property>
  <name>hbase.zookeeper.quorum</name>
  <value>zk_host1,1zk_host2,zk_host3</value>
</property>
</configuration>
```

### 常用Hbase Shell操作
#### namespace

- 创建namespace
```
hbase(main):010:0> create_namespace 'testhbase'
0 row(s) in 1.6130 seconds
```

- 查看namespace列表
```
hbase(main):011:0> list_namespace
NAMESPACE
default
hbase
jiuji
testhbase                                                                                                                                                                            
4 row(s) in 0.5390 seconds
```

- 删除namespace
```
hbase(main):013:0> drop_namespace 'testhbase'
0 row(s) in 0.9450 seconds
```

#### table

- 在namespace tp 下 创建表 sys 包含两个列族
```
hbase(main):022:0> create 'tp:sys','col_user','col_limits'
0 row(s) in 1.9310 seconds

=> Hbase::Table - tp:sys
```

- 向表 `tp:sys` 添加一条记录 rowKey 是 10001
```
hbase(main):027:0> put 'tp:sys','10001','col_user:basic_info','{"name":"Sam","age":27}'
0 row(s) in 0.2560 seconds

hbase(main):028:0> put 'tp:sys','10001','col_user:bankcards','[{"card_no":"6225"},[{"card_no":"6221"}]'
0 row(s) in 0.0420 seconds
```

- 查看上面插入的数据 查询指定rowKey行的数据
```
hbase(main):037:0> get 'tp:sys','10001'
COLUMN                                         CELL                                                                                                                                  
 col_user:bankcards                            timestamp=1589269848710, value=[{"card_no":"6225"},[{"card_no":"6221"}]                                                               
 col_user:basic_info                           timestamp=1589269786766, value={"name":"Sam","age":27}                                                                                
1 row(s) in 0.1770 seconds
```

- 查看上面插入的数据 查询指定列数据

```
hbase(main):038:0> get 'tp:sys','10001','col_user'
COLUMN                                         CELL                                                                                                                                  
 col_user:bankcards                            timestamp=1589269848710, value=[{"card_no":"6225"},[{"card_no":"6221"}]                                                               
 col_user:basic_info                           timestamp=1589269786766, value={"name":"Sam","age":27}                                                                                
1 row(s) in 0.0230 seconds
```

- 查看Cell数据单元的数据
```
hbase(main):039:0> get 'tp:sys','10001','col_user:basic_info'
COLUMN                                         CELL                                                                                                                                  
 col_user:basic_info                           timestamp=1589269786766, value={"name":"Sam","age":27}                                                                                
1 row(s) in 0.0400 seconds
```

- count 表中的数据
```
hbase(main):042:0> count 'tp:sys'
2 row(s) in 0.0140 seconds

=> 2
```

- scan 模糊查询表中的数据

```
hbase(main):043:0> scan 'tp:sys'
ROW                                            COLUMN+CELL                                                                                                                           
 10001                                         column=col_user:bankcards, timestamp=1589269848710, value=[{"card_no":"6225"},[{"card_no":"6221"}]                                    
 10001                                         column=col_user:basic_info, timestamp=1589269786766, value={"name":"Sam","age":27}                                                    
 10002                                         column=col_user:bankcards, timestamp=1589270266409, value=[{"card_no":"6225"},[{"card_no":"6221"}]                                    
2 row(s) in 0.0410 seconds
```
