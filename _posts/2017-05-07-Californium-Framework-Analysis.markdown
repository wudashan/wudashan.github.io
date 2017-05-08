---
layout:     post
title:      "Coap协议与Californium开源框架分析"
subtitle:   "一个受限制的应用协议，一个基于Java实现的Coap技术框架。"
date:       2017-05-07 22:00:00
author:     "Wudashan"
header-img: "img/post-bg-californium-framework-analysis.jpg"
catalog: true
tags:
    - Californium
    - 开源框架
    - 源码分析
---

> 项目源码地址：[https://github.com/eclipse/californium](https://github.com/eclipse/californium)

## 引言

物联网时代，所有设备都可以接入我们的互联网。想想看只要有一台智能手机，就可以操控所有的设备，也可以获取到所有设备采集的信息。不过，并不是所有设备都支持HTTP协议的，而且让设备支持HTTP协议也不现实，因为对于设备来说，这个协议太重了，会消耗大量的带宽和电量。于是CoAP协议也就运应而生了，我们可以把它看为超简化版的HTTP协议。而Californium框架，就是对CoAP协议的Java实现。

## 未完待续

## 参考阅读

[Californium 框架设计分析](http://www.cnblogs.com/littleatp/p/6417567.html)

[CoAP协议学习——CoAP基础](http://blog.csdn.net/xukai871105/article/details/17734163)

[RFC 7252 - The Constrained Application Protocol (CoAP)](https://tools.ietf.org/html/rfc7252)