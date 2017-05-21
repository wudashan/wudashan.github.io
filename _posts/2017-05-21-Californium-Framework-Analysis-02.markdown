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

**CoAP类**

该类定义了CoAP报文的一些常量，以及按协议要求定义了报文中各字段的格式。包括：

 - 消息类型：CON，NON，ACK，RST
 - 请求码：GET，POST，PUT，DELETE
 - 响应码：成功2.XX，客户端错误4.XX，服务端错误5.XX

**OptionNumberRegistry类**

根据RFC 7252的第12.2章节，该类定义了CoAP报文中Option字段支持的值。其值如下表格：

```
             +--------+------------------+-----------+ 
             | Number | Name             | Reference | 
             +--------+------------------+-----------+ 
             |      0 | (Reserved)       | [RFC7252] | 
             |      1 | If-Match         | [RFC7252] |
             |      3 | Uri-Host         | [RFC7252] | 
             |      4 | ETag             | [RFC7252] |  
             |      5 | If-None-Match    | [RFC7252] |  
             |      6 | Observe          |           |  
             |      7 | Uri-Port         | [RFC7252] |  
             |      8 | Location-Path    | [RFC7252] | 
             |     11 | Uri-Path         | [RFC7252] |
             |     12 | Content-Format   | [RFC7252] |
             |     14 | Max-Age          | [RFC7252] | 
             |     15 | Uri-Query        | [RFC7252] |  
             |     17 | Accept           | [RFC7252] |   
             |     20 | Location-Query   | [RFC7252] |  
             |     23 | Block2           |           |    
             |     27 | Block1           |           | 
             |     28 | Size2            |           | 
             |     35 | Proxy-Uri        | [RFC7252] |  
             |     39 | Proxy-Scheme     | [RFC7252] | 
             |     60 | Size1            | [RFC7252] |
             |    128 | (Reserved)       | [RFC7252] | 
             |    132 | (Reserved)       | [RFC7252] |  
             |    136 | (Reserved)       | [RFC7252] |   
             |    140 | (Reserved)       | [RFC7252] |    
             +--------+------------------+-----------+ 
```



