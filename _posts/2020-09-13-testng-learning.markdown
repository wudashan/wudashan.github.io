---
layout:     post
title:      "《TestNG》学习笔记"
subtitle:   ""
date:       2020-09-13 22:00:00
author:     "Wudashan"
header-img: "img/post-bg-testng-learning.jpg"
catalog: true
tags:
    - 开源框架
    - 源码分析
---

# 源码版本

```xml
<dependency>
    <groupId>org.testng</groupId>
    <artifactId>testng</artifactId>
    <version>6.8</version>
    <scope>test</scope>
</dependency>
```


# 执行时序图

![](https://raw.githubusercontent.com/wudashan/blog-picture/6dcdd3597aaf76082381770da124afcb9accef4f/testng-learning/1.svg)

![](https://raw.githubusercontent.com/wudashan/blog-picture/6dcdd3597aaf76082381770da124afcb9accef4f/testng-learning/2.svg)

![](https://raw.githubusercontent.com/wudashan/blog-picture/6dcdd3597aaf76082381770da124afcb9accef4f/testng-learning/3.svg)

# 关键类类图

## TestNG 主程序

![](https://raw.githubusercontent.com/wudashan/blog-picture/6dcdd3597aaf76082381770da124afcb9accef4f/testng-learning/b1.svg)

## Parser 文件解析器

![](https://raw.githubusercontent.com/wudashan/blog-picture/6dcdd3597aaf76082381770da124afcb9accef4f/testng-learning/b2.svg)

## XmlSuite 测试数据

![](https://raw.githubusercontent.com/wudashan/blog-picture/6dcdd3597aaf76082381770da124afcb9accef4f/testng-learning/b3.svg)

## Listener 监听器

![](https://raw.githubusercontent.com/wudashan/blog-picture/6dcdd3597aaf76082381770da124afcb9accef4f/testng-learning/b4.svg)

# 参考链接
* [TestNG官方文档](https://testng.org/doc/documentation-main.html)
* [TestNG学习笔记（语雀版）](https://www.yuque.com/wudashan/olrmnh/gsmge6)