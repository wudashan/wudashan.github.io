---
layout:     post
title:      "Californium开源框架之源码分析（六）"
subtitle:   "network模块（下），网络传输核心模块。"
date:       2017-07-07 22:00:00
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

# network包

network包目录下，是框架中网络传输的核心模块。

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/network%E5%8C%85.png)

## stack包

### Layer接口

该接口表示抽象的协议层，负责处理CoAP消息。当处理完后，会将消息往上层或下层传递，由它们继续处理。使用的是设计模式中的职责链模式。

当`CoapEndpoint`收到消息时，它会把消息传给最底层的协议层，并回调`receiveXXX()`方法，每个协议层在处理完消息后决定是否将消息传给上层协议层。最上层的协议层会将消息传给`MessageDeliverer`。

当`CoapEndpoint`发送消息时，它会把消息传给最上层的协议层，并回调`sendXXX()`方法，每个协议层在处理完消息后决定是否将消息传给下层协议层。最下层的协议层会将消息传给`Outbox`。

`Exchange`包含着一对请求与响应的所有信息，协议层可以并发地修改Exchange。虽然多数情况下是单线程地修改Exchange，但是如果是多线程的场景，需要对Exchange做同步修改的保护。

每一协议层可以有自己的线程池，处理自己的任务，比如重传等等。

该接口还提供了一个`TopDownBuilder`类，用来生成协议栈。