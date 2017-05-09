---
layout:     post
title:      "Californium开源框架分析"
subtitle:   "一个基于Java实现的CoAP技术框架。"
date:       2017-05-07 22:00:00
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

物联网时代，所有设备都可以接入我们的互联网。想想看只要有一台智能手机，就可以操控所有的设备，也可以获取到所有设备采集的信息。不过，并不是所有设备都支持HTTP协议的，而且让设备支持HTTP协议也不现实，因为对于设备来说，这个协议太重了，会消耗大量的带宽和电量。于是CoAP协议也就运应而生了，我们可以把它看为超简化版的HTTP协议。而Californium框架，就是对CoAP协议的Java实现。

## CoAP协议

在阅读Californium框架之前，我们需要对CoAP协议有个大致的了解，已经懂得了的同学可以直接跳过本章节。

#### CoAP报文

首先让我们看一下CoAP协议的报文是长啥样的：

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/coap%E6%95%B0%E6%8D%AE%E5%8C%85.png)

**Version (Ver)：**长度为2位，表示CoAP协议的版本号。当前版本为01（二进制表示形式）。

**Type (T)：**长度为2位，表示报文类型。其中各类型及二进制表示形式如下，Confirmable (00)、Non-confirmable (01)、Acknowledgement (10)、Reset (11)。在描述的时候为了简便，会将Confirmable缩写为CON，Non-confirmable缩写为NON，Acknowledgement缩写为ACK，Reset缩写为RST。比如一个报文的类型为Confirmable，我们就会简写为CON报文。

**Token Length (TKL)：**长度为4位，表示Token字段的长度。

**Code：**长度为8位，表示响应码。其中前3位代表一个数，后5位代表一个数。如`010 00000`，转为十进制就是`2.00`（表示时中间带一个点），其意思可以理解为HTTP中`200 OK`响应码。

**Message ID：**长度为16位，表示消息id。用来表示是否为同一个的报文（重发场景下，去重会用到），或者CON请求报文和ACK响应报文的匹配。

**Token：**长度由TKL字段决定，表示一次会话记录。用来关联请求和响应。有人可能有疑惑，Message ID不是可以将请求和响应关联吗？的确，CON类型的请求报文与ACK类型的响应报文可以用Message ID进行关联，但NON类型的报文由于没有要求是一对的，所有如果NON类型的报文想成对，那就只能通过相同的Token来匹配了。

**Options：**长度不确定，表示报文的选项。类似为HTTP的请求头，内容包括Uri-Host、Uri-Path、Uri-Port等等。

**1 1 1 1 1 1 1 1：**Payload Marker，用来隔离Options字段和Payload字段。

**Payload：**长度由数据包决定，表示应用层需要的数据。


#### 消息传输模型

CoAP协议是虽然是建立在UDP之上的，但是它有可靠和不可靠两种传输模型。

**可靠传输模型：**

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/%E5%8F%AF%E9%9D%A0%E6%B6%88%E6%81%AF%E4%BC%A0%E8%BE%93%E6%A8%A1%E5%9E%8B.png)

如上图，客户端通过发起一个CON报文（Message ID = 0x7d34），服务端在收到CON报文之后，需要回复一个ACK报文（Message ID = 0x7d34）。通过Message ID将CON报文和ACK报文对应起来。

确保可靠传输的方法有俩：其一，通过服务端回复ACK报文，客户端可以确认CON报文已被服务端接收；其二，超时重传机制。若客户端在一定时间内未收到ACK报文，则认为CON报文已经在链路上丢失，这时候就会重传CON报文，重传时间和次数可配置。

**不可靠传输模型：**

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/%E4%B8%8D%E5%8F%AF%E9%9D%A0%E6%B6%88%E6%81%AF%E4%BC%A0%E8%BE%93%E6%A8%A1%E5%9E%8B.png)

如上图，客户端发起一个NON报文（Message ID = 0x01a0）之后，服务端无需回复响应，客户端也不会重发。


## 未完待续

## 参考阅读

[Californium 框架设计分析](http://www.cnblogs.com/littleatp/p/6417567.html)

[CoAP协议学习——CoAP基础](http://blog.csdn.net/xukai871105/article/details/17734163)

[RFC 7252 - The Constrained Application Protocol (CoAP)](https://tools.ietf.org/html/rfc7252)