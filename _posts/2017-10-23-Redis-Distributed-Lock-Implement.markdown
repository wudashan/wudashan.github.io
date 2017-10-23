---
layout:     post
title:      "Redis分布式锁的正确实现方式（Java版）"
subtitle:   "不瞒你们说，网上80%的实现都是有缺陷的"
date:       2017-10-23 21:00:00
author:     "Wudashan"
header-img: "img/post-bg-redis-distributed-lock.jpg"
catalog: true
tags:
    - 分布式锁
    - Redis
    - Jedis
---

> 本博客使用Jedis包实现与Redis服务端通信，且只考虑Redis单机部署的场景。

# 前言

分布式锁一共有3种实现方式：1.数据库乐观锁；2.基于Redis的分布式锁；3.基于ZooKeeper的分布式锁。刚好最近开发的功能需要分布式锁，考虑到自己对Redis还比较熟悉，而且项目组也没有前辈实现过分布式锁，所以决定自己一个人完成这项伟大的工程。


未完待续...