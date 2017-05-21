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

在[Californium开源框架分析（上）](http://wudashan.cn/2017/05/07/Californium-Framework-Analysis-01)博客中，我们通过模拟Debug + 源码走读的方式，对Californium框架有了一个整体的认识。本篇博客，我们将按框架的目录结构，对所有的类进行详细的分析和解读。

当我们在pom.xml文件里引入Californium开源框架的依赖之后，将会出现`californium-core`和`element-connector`两个jar包。其中`californium-core`是框架的核心实现，而`element-connector`则是从框架中独立出来的网络传输模块。话不多说，就让我们赶紧看看它们的内部结构吧！

## californium-core

未完待续。
