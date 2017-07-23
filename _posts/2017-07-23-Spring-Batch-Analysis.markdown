---
layout:     post
title:      "Spring Batch批处理框架介绍（小白入门）"
subtitle:   "一款轻量的、全面的批处理框架，用于开发强大的批处理应用程序。"
date:       2017-07-23 22:30:00
author:     "Wudashan"
header-img: "img/post-bg-spring-batch-analysis.jpg"
catalog: true
tags:
    - Spring
    - Batch
    - 批处理
    - 开源框架
---

> 本篇博客基于Spring Batch的3.0.8版本。

# 前言

在大型的企业应用中，或多或少都会存在大量的任务需要处理，如邮件批量通知所有将要过期的会员等等。而在批量处理任务的过程中，又需要注意很多细节，如任务异常、性能瓶颈等等。那么，使用一款优秀的框架总比我们自己重复地造轮子要好得多一些。

我所在的物联网云平台部门就有这么一个需求，需要实现批量下发命令给百万设备。为了防止枯燥乏味，下面就让我们先通过Spring Batch框架简单地实现一下这个功能，再来详细地介绍这款框架！

