---
layout: post
title: 使用 mysqldump 备份MySQL表结构和数据
date: 2016-07-02 16:16:16
tags: [mysql,mysqldump]
categories:  
baseline:
---

数据库需要备份


### 备份
```
# 导出数据库表结构及数据
$ mysqldump -h 10.10.10.10 -P 3306 -u root -p123456 dbName > structure_data.sql


-d 参数只导出表结构

# 导出数据库表结构

```


### 恢复


Questions：

1. mysqldump: Got error: 1033: Incorrect information in file: './\*/\*.frm' when using LOCK TABLES

别琢磨了 从备份恢复吧
