---
layout: post
title: 本地搭建、部署JekyII
date: 2017-10-11 11:11:11
tags: [HowTo manual]
---

有很多东西需要记下来,搭建一个博客系统方便查阅

我们接下来在本地搭建一个JekyII环境


### 创建本地环境
``` shell
sudo gem install bundler jekyll
# Successfully installed bundler-2.0.2
# Successfully installed jekyll-3.8.6

jekyll new my-awesome-site
cd my-awesome-site
# 启动服务
bundle exec jekyll serve
```

```
Server address: http://127.0.0.1:4000/
Server running... press ctrl-c to stop.
```
