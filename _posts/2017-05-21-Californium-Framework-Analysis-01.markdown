---
layout:     post
title:      "Californium开源框架之源码分析（一）"
subtitle:   "从整体到模块，从模块到细节，细节见卓越！"
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

# 引言

在[《Californium开源框架分析（入门篇）》](http://wudashan.cn/2017/05/07/Californium-Framework-Analysis)博客中，我们通过模拟Debug + 源码走读的方式，对Californium框架有了一个整体的认识。本篇博客，我们将按框架的目录结构，对框架的结构进行分解，后续系列文章对每个包进行详细的分析和解读。Californium开源框架由`californium-core.jar`和`element-connector.jar`两个jar包组成，分析的版本为`2.0.0-M1`。


---

# californium-core.jar

californium-core是框架的核心实现，包图如下：

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/californium-core%E5%8C%85%E5%9B%BE.png)

## coap包

该模块包含了CoAP协议中定义的常量和消息基本模型。模块一共有以下类：

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/coap%E5%8C%85%E7%B1%BB%E5%9B%BE.png)

## observe包

该模块为CoAP协议的订阅模块，是协议一大功能。模块一共有以下类：

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/observe%E5%8C%85_01.png)

## server包

该模块为Californium框架中的服务端，模块一共有以下类：

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/server%E5%8C%85_01.png)


## network包

该模块为框架中网络传输的核心部分，包图如下：

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/network%E5%8C%85.png)

---

# element-connector.jar

element-connector则是从框架中独立出来的网络传输模块，其类如下：

 - Connector接口
 - ConnectorBase类
 - ConnectorFactory接口
 - CorrelationContext接口
 - DtlsCorrelationContext类
 - MapBasedCorrelationContext类
 - MessageCallback接口
 - RawData类
 - RawDataChannel接口
 - UDPConnector类

---

# 系列文章

[Californium开源框架之源码分析（一）—— 整体认识](http://wudashan.cn/2017/05/21/Californium-Framework-Analysis-01/) <-- 当前位置

[Californium开源框架之源码分析（二）—— coap包](http://wudashan.cn/2017/06/01/Californium-Framework-Analysis-02/)

[Californium开源框架之源码分析（三）—— observe包](http://wudashan.cn/2017/06/05/Californium-Framework-Analysis-03/)

[Californium开源框架之源码分析（四）—— server包](http://wudashan.cn/2017/06/16/Californium-Framework-Analysis-04/)

[Californium开源框架之源码分析（五）—— network包](http://wudashan.cn/2017/07/02/Californium-Framework-Analysis-05/)

