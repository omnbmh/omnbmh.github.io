---
layout: post
title: Linux less 命令
date: 2016-01-13 16:16:16
tags: [linux,shell]
categories: [linux-manual]
baseline:
---

- Description<br>
Less 类似于more命令，它可以向前和向后查看文件文件。另外，less 在查看文件之前不需要预加载文本文件，因此它在处理大文件的时候比起vi等文本编辑器启动时比较快的。
- Usage<br>
`less [option] file`
- Options
  - -e 当文件显示完后 自动离开
  - -m 类似more命令的百分比
  - -N 输出行号
  - -ppattern or --pattern=pattern 正泽查询 和 +/pattern 相同
 - -+ +号后面跟着Commands的命令 直接执行命令
- Commands
  - 回车 向下滚动一行
  - y 向上滚动一行
  - d 向下滚动半屏
  - u 向上滚动半屏
  - p n% 在整个文件的10%处开始显示
  - /pattern 在文件中搜索 pattern 关键字
  - n 向下查找
  - N 向上查找
  - g 跳到第一行
  - G 跳到最后一行
  - 退出命令是 q
- sample
  1. 查询文件
  less text.txt
  2. 查询历史命令 并分页显示
  history | less
  3. 查看多个文件
  less text1.txt text2.txt text3.txt
  4. 查询关键字 `error` 日志<br>
  使用 `less logfile` 打开日志文件,输入`/error`然后回车 搜索到后会高亮显示
  5. 文件比较大的时候 查询关键字 `error` 日志<br>
    - 需要查询最近的<br>
    使用 `less logfile` 打开日志文件,输入`G`然后回车 这会会定位到文件的尾部，然后 `/error` 开始查询，肯定是找不到，输入 `N` 从下往上开始查找 搜索到后会高亮显示
    - 要查询的日志 在文件中间附近
    使用 `less logfile` 打开日志文件,输入`p 30%`然后回车 这会会定位到文件30%处，然后 `/error` 开始查询，肯定是找不到，输入 `N` 从下往上开始查找 搜索到后会高亮显示
