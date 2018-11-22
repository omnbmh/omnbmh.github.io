---
layout: post
title: Linux mkdir 命令
date: 2016-01-04 16:16:16
tags: [linux,shell]
categories: linux-manual
baseline: 
---

mkdir命令用来创建新目录。

- syntax<br>
- options
  - -m 设定权限 比如 644 777等
  - -p 可以一次创建多层目录
  - -v 显示创建详情
  - --help 显示帮助信息
  - --version 显示版本信息
- samples
  - 创建一个目录<br>
    `mkdir dir1`
  - 创建多级目录<br>
    `mkdir -p dir1/dir2`
  - 创建权限是777的目录<br>
    `mkdir -m 777 dir3`
  - 创建一个目录结构<br>
    `mkdir -vp dir4/dir5/{logs/{bin/,lib/},bin/,lib/}`
