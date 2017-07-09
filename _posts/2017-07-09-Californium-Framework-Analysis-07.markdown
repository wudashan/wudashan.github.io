---
layout:     post
title:      "Californium开源框架之源码分析（七）"
subtitle:   "core包，一些封装好的供开发者使用的类。"
date:       2017-07-09 21:00:00
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

# core包

core根目录下，封装好了一些供开发者使用的类。

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/core.png)

## 根目录

### CaliforniumFormatter类

该类继承了`java.util.logging.Formatter`类，并重写了`format()`方法。对日志输出进行了格式化处理，默认给每条日志添加线程id、日志级别、类名、详细信息、源码行号、方法名、线程名信息，方便定位问题。

### CaliforniumLogger类

当`java.util.logging`没有使用配置文件时，该类可以作为一个辅助类控制`org.eclipse.californium.core`和`org.eclipse.californium.elements`包下的日志输出格式和级别。日志输出格式使用的是CaliforniumFormatter类。

### CoapHandler接口

该类用于`CoapClient`异步发送消息，需要对消息的后续结果进行处理的场景。当客户端收到响应时，回调`onLoad()`方法；当客户端发送的请求超时或被服务端拒绝时，回调`onError()`方法。

### CoapObserveRelation类

该类用于`CoapClient`处理订阅关系，它表示一个客户端和服务端资源的订阅关系。该类提供了几个简单的public方法，如检测订阅关系是否建立或取消，重新注册以刷新订阅关系。

### CoapClient类

该类表示CoAP客户端，提供了很多方法供开发者使用，主要有`postXXX()`、`getXXX()`、`putXXX()`、`deleteXXX()`方法，同步和异步发送请求，发起订阅请求、发起ping请求和发起discover请求等等。

### CoapResource类

该类在[《Californium开源框架之源码分析（四）—— server包》](http://wudashan.cn/2017/06/16/Californium-Framework-Analysis-04/)已分析过。