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

在[《Californium开源框架分析（入门篇）》](http://wudashan.cn/2017/05/07/Californium-Framework-Analysis)博客中，我们通过模拟Debug + 源码走读的方式，对Californium框架有了一个整体的认识。本篇博客，我们将按框架的目录结构，对框架的结构进行分解，后续系列文章对每个模块进行详细的分析和解读。Californium开源框架由`californium-core`和`element-connector`两个jar包组成。

---

# californium-core.jar

californium-core是框架的核心实现，包图如下：

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/californium-core%E5%8C%85%E5%9B%BE.png)

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

[Californium开源框架之源码分析（一）——模块划分](http://wudashan.cn/2017/05/21/Californium-Framework-Analysis-01/) <-- 当前位置

[Californium开源框架之源码分析（二）——coap模块](http://wudashan.cn/2017/06/01/Californium-Framework-Analysis-02/)

[Californium开源框架之源码分析（三）——observe模块](http://wudashan.cn/2017/06/05/Californium-Framework-Analysis-03/)

