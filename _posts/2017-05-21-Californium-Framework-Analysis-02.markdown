---
layout:     post
title:      "Californium开源框架分析（下）"
subtitle:   "一个基于Java实现的CoAP技术框架。"
date:       2017-05-21 22:00:00
author:     "Wudashan"
header-img: "img/post-bg-californium-framework-analysis.jpg"
catalog: true
tags:
    - Californium
    - 开源框架
    - 源码分析
    - CoAP
---

> 项目源码地址：[https://github.com/eclipse/californium](https://github.com/eclipse/californium)

## 引言

在[上一篇](http://wudashan.cn/2017/05/07/Californium-Framework-Analysis-01)博客中，我们通过模拟Debug + 源码走读的方式，对Californium框架有了一个整体的认识。本篇博客，我们将按框架的目录结构，对所有的类进行详细的分析和解读。

Californium开源框架由`californium-core`和`element-connector`两个jar包组成。其中`californium-core`是框架的核心实现，而`element-connector`则是从框架中独立出来的网络传输模块。话不多说，就让我们赶紧看看它们的内部结构吧！

## californium-core

包图如下：

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/californium-core%E5%8C%85%E5%9B%BE.png)

#### coap包

coap包目录下，主要是CoAP协议中定义的常量和消息基本模型。

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/coap%E5%8C%85%E7%B1%BB%E5%9B%BE.png)

###### CoAP类

未完待续。

