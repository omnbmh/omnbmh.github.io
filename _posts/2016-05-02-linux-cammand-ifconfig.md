---
layout: post
title: Linux ifconfig 命令
date: 2016-05-02 16:16:16
tags: [linux,shell]
categories:
baseline:
---

ifconfig工具可以用来查看和修改网络配置信息。

- syntax<br>
  - ifconfig [网络设备] [参数]
- options
 - up 启动指定网络设备/网卡。
 - down 关闭指定网络设备/网卡。该参数可以有效地阻止通过指定接口的IP信息流，如果想永久地关闭一个接口，我们还需要从核心路由表中将该接口的路由信息全部删除。
 - arp 设置指定网卡是否支持ARP协议。
 - -allmulti 设置是否支持多播模式，如果选择此参数，网卡将接收网络中所有的多播数据包
 - -a 显示全部接口信息
 - -s 显示摘要信息（类似于 netstat -i）
 - add 给指定网卡配置IPv6地址
 - del 删除指定网卡的IPv6地址
 - <硬件地址> 配置网卡最大的传输单元
 - mtu<字节数> 设置网卡的最大传输单元 (bytes)
 - netmask<子网掩码> 设置网卡的子网掩码。掩码可以是有前缀0x的32位十六进制数，也可以是用点分开的4个十进制数。如果不打算将网络分成子网，可以不管这一选项；如果要使用子网，那么请记住，网络中每一个系统必须有相同子网掩码。
 - tunel 建立隧道
 - dstaddr 设定一个远端地址，建立点对点通信
 - -broadcast<地址> 为指定网卡设置广播协议
 - -pointtopoint<地址> 为网卡设置点对点通讯协议
 - multicast 为网卡设置组播标志
 - address 为网卡设置IPv4地址
 - txqueuelen<长度> 为网卡设置传输列队的长度
- samples
  - 显示网络设备信息（激活状态的<br>
    `ifconfig`
  - 启动关闭指定网卡<br>
    `ifconfig eth0 up`
    `ifconfig eth0 down`
