---
layout: post
title: Linux pwd 命令
date: 2016-01-03 16:16:16
tags: [linux,shell]
categories:
baseline:
---

pwd命令用来查看用户的当前所在目录。

- syntax<br>
  `pwd [option]`
- options
  - -P 目录是链接时 显示出实际路径
  - -L 显示出链接的路径
- samples
  - 查看当前所在目录<br>
    `pwd`
  - 查看链接的真实路径<br>
    `pwd -P`
