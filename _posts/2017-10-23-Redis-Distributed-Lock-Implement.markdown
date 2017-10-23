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
    - Java
---

> 本博客使用Jedis实现Redis客户端，且只考虑Redis服务端单机部署的场景。

# 前言

分布式锁一共有3种实现方式：1. 数据库乐观锁；2. 基于Redis的分布式锁；3. 基于ZooKeeper的分布式锁。本篇博客将介绍第二种方式，基于Redis实现分布式锁。虽然网上已经有各种介绍Redis分布式锁实现的博客，然而他们的实现却有着各种各样的问题，为了避免误人子弟，本篇博客将详细介绍如何正确地实现Redis分布式锁。

---

# 可靠性

为了确保分布式锁的可靠性，我们至少要确保我们的实现同时满足以下三个条件：

1. 互斥性。在任一时刻，只有一个客户端能持有锁。
2. 不会发生死锁。即使有一个客户端在持有锁的期间崩溃而没有主动释放锁，也能保证后续其他客户端能获取锁。
3. 具有容错性。只要大部分的Redis节点正常运行，客户端就可以获取锁和释放锁。


**未完待续...**

---

# 参考阅读

[[1] Distributed locks with Redis](https://redis.io/topics/distlock)


