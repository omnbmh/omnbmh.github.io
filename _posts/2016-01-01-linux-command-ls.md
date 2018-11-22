---
layout: post
title: Linux ls 命令
date: 2016-01-01 16:16:16
tags: [linux,shell]
categories: linux-manual
baseline:
---

ls命令用于查看文件夹和文件的信息（名称，权限等）

- syntax<br>
  `ls [option] file or directory`
- options
  - -a 列出目录下饿所有文件 包含隐藏文件 . 开头的
  - -h 好看的形式显示文件大小 （1k 2M 3G）
  - -i 显示文件的inode
  - -k 以k字节显示文件大小
  - -t 以文件修改顺序排序
  - -help 显示帮助信息
  - -version 显示版本信息
- samples
  - 列出`/etc/rc.d`文件夹下的所有文件和目录的详细信息<br>
    ```
    ls -lR /etc/rc.d
    ```
  - 列出`/etc`下以p开头的目录的详细信息<br>
    ```
    ls -l /etc/p*
    ```
  - 计算当前目录下的文件数和目录数
    - 文件数<br>
      ```
      ls -l * | grep '^-' | wc -l
      ```
    - 目录数<br>
      ```
      ls -l * | grep '^d' | wc -l
      ```
  - 列出文件的绝对路径
    ```
    ls | sed 's:^:\`pwd\`/:'
    ```
- more
  - 看到文件颜色五颜六色<br>
    ```
    蓝色 -> 目录
    绿色 -> 可执行文件
    红色 -> 压缩文件
    浅灰色 -> 链接文件
    灰色 -> 其他文件
    ```
