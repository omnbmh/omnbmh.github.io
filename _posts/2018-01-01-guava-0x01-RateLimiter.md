---
layout: post
title: 【Guava系列】 0x01. RateLimiter速率限制器的使用
date: 2016-07-01 16:16:16
tags: [Guava, RateLimiter]
categories: [Guava系列]
baseline:
---

RateLimiter速率限制器使用的是一种叫令牌桶的流控算法，RateLimiter会按照一定的频率忘小桶里面扔令牌，当前线程只有拿到令牌才能继续执行。
