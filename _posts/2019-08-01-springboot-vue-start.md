---
layout: post
title: SpringBoot+Vue实战（一）- 环境搭建、项目初始化
date: 2019-04-17 15:55:18
tags: [Spring Boot, Vue]
categories: [SpringBoot+Vue实战]
baseline: 
artileno: 20190801_01
---

## 环境搭建

### 项目开发环境
- JDK 1.8
- Mysql Redis
- Intellij Idea 
- Spring Boot
- Spring Security
- Vue

## 项目初始化

1. 创建项目工作目录

```
mkdir cobra
cd cobra
```

2. 使用 Vue 创建前端项目

```
vue init webpack cobra-vue
```

创建完成后，项目目录如下

![目录结构]({{site.url}}/assets/20190801_01_01.png)

3. 使用 Maven 创建 SpringBoot 项目

```
mvn archetype:generate -DgroupId=org.github.omnbmh.cobra -DartifactId=cobra-spring-boot -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

```

创建完成后，项目目录如下

![目录结构]({{site.url}}/assets/20190801_01_02.png)
