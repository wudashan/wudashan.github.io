---
layout:     post
title:      "如何快速过滤出一次请求的所有日志？"
subtitle:   "使用SLF4J日志框架的MDC工具，将请求标识植入一次请求的生命周期。"
date:       2018-02-15 14:30:00
author:     "Wudashan"
header-img: "img/post-bg-log-requset-in-mutithread.jpg"
catalog: true
tags:
    - 日志
    - Log4j
    - 请求标识
---

> 示例源码地址：`https://github.com/wudashan/slf4j-mdc-muti-thread`

# 前言

在现网出现故障时，我们经常需要获取一次请求流程里的所有日志进行定位。如果请求只在一个线程里处理，则我们可以通过线程ID来过滤日志，但如果请求包含异步线程的处理，那么光靠线程ID就显得捉襟见肘了。

华为IoT平台，提供了接收设备上报数据的能力， 当数据到达平台后，平台会进行一些复杂的业务逻辑处理，如数据存储，规则引擎，数据推送，命令下发等等。由于这个逻辑之间没有强耦合的关系，所以通常是异步处理。如何将一次数据上报请求中包含的所有业务日志快速过滤出来，就是本文要讲解的。

---
