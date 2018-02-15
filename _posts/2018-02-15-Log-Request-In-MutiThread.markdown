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



---
