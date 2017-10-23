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


**未完待续...**