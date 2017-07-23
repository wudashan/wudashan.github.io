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

---

# 小试牛刀

## 引入依赖

首先我们需要引入对`Spring Batch`的依赖，在`pom.xml`文件加入下面的代码：

```
<dependency>
    <groupId>org.springframework.batch</groupId>
    <artifactId>spring-batch-core</artifactId>
    <version>3.0.8.RELEASE</version>
</dependency>
```

## 装载Bean

其次，我们需要在resources目录下，创建`applicationContext.xml`文件，用于自动注入我们需要的类：

```
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
           http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">


    <!-- 事务管理器 -->
    <bean id="transactionManager" class="org.springframework.batch.support.transaction.ResourcelessTransactionManager"/>

    <!-- 任务仓库 -->
    <bean id="jobRepository" class="org.springframework.batch.core.repository.support.MapJobRepositoryFactoryBean">
        <property name="transactionManager" ref="transactionManager"/>
    </bean>

    <!-- 任务加载器 -->
    <bean id="jobLauncher" class="org.springframework.batch.core.launch.support.SimpleJobLauncher">
        <property name="jobRepository" ref="jobRepository"/>
    </bean>

</beans>
```

有了上面声明的transactionManager、jobRepository、jobLauncher，我们就可以执行批量任务啦！不过，我们还需要创建一个任务。在Spring Batch框架中，一个任务Job由一个或者多个步骤Step，而步骤又由读操作Reader、处理操作Processor、写操作Writer组成，下面我们分别创建它们。
