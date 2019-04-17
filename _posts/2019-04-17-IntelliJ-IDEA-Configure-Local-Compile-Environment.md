---
layout: post
title: IntellliJ IDEA 配置本底编译环境
date: 2019-04-17 15:55:18
tags: [linux]
categories:
baseline:
---

### Maven

### Gradle
#### 0x01 下载二进制包 https://services.gradle.org/distributions/gradle-5.4-bin.zip
#### 0x02 解压缩 `unzip gradle-5.4-bin.zip` 配置好环境变量
- 我的安装目录 `/Applications/Utilities/gradle-5.4`
```
# 设置环境变量
export GRADLE_HOME=/Applications/Utilities/gradle-5.4
export PATH=$GRADLE_HOME/bin:$PATH
```
#### 0x03 验证安装
```
$ gradle -v

Welcome to Gradle 5.4!
...
------------------------------------------------------------
Gradle 5.4
------------------------------------------------------------

Build time:   2019-04-16 02:44:16 UTC
Revision:     a4f3f91a30d4e36d82cc7592c4a0726df52aba0d
...
```

#### 0x04 配置 IntelliJ IDEA 的本地Gradle路径
1. 打开 Preferences...
2. 按照下图序号配置 点击 Apply Ok
3. ![示例图]({{ site.url }}/assets/1555490136174.jpg)
