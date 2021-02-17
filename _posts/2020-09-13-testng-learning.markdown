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

# 框架介绍

## 英文原版

```
TestNG is a testing framework inspired from JUnit and NUnit but introducing some new functionalities that make it more powerful and easier to use, such as:

* Annotations.
* Run your tests in arbitrarily big thread pools with various policies available (all methods in their own thread, one thread per test class, etc...).
* Test that your code is multithread safe.
* Flexible test configuration.
* Support for data-driven testing (with @DataProvider).
* Support for parameters.
* Powerful execution model (no more TestSuite).
* Supported by a variety of tools and plug-ins (Eclipse, IDEA, Maven, etc...).
* Embeds BeanShell for further flexibility.
* Default JDK functions for runtime and logging (no dependencies).
* Dependent methods for application server testing.

TestNG is designed to cover all categories of tests:  unit, functional, end-to-end, integration, etc...
```

## 中文翻译

TestNG是一个受JUnit和NUnit启发的测试框架，但引入了一些使其更强大且更易于使用的新功能，例如：

* 注解。
* 线程池中运行测试用例。
* 支持测试代码是否多线程安全。
* 灵活的测试配置。
* 支持数据驱动的测试（使用@DataProvider）。
* 以插件形式被各种工具（Eclipse，IDEA，Maven等）集成。

TestNG旨在涵盖所有类别的测试：单元，功能，端到端，集成等。


# 源码版本

```xml
<dependency>
    <groupId>org.testng</groupId>
    <artifactId>testng</artifactId>
    <version>6.8</version>
    <scope>test</scope>
</dependency>
```

# 带着问题去学习

## TestNG如何解析命令行参数？

## TestNG如何解析testng.xml配置文件？

## TestNG如何发现需要被测试的类/方法？

## TestNG如何支持用户感知运行时状态？

## TestNG如何支持多线程执行测试用例？

## TestNG如何支持参数化执行用例？

## TestNG如何展示用例执行结果？

## TestNG如何解决测试用例之间的依赖顺序？



# 执行时序图

## TestNG.main()

![](https://raw.githubusercontent.com/wudashan/blog-picture/master/testng-learning/1.svg)

## TestNG.runSuitesLocally()

![](https://raw.githubusercontent.com/wudashan/blog-picture/master/testng-learning/2.svg)

## TestRunner.createWorkersAndRun()

![](https://raw.githubusercontent.com/wudashan/blog-picture/master/testng-learning/3.svg)

# 关键类类图

## TestNG 主程序

![](https://raw.githubusercontent.com/wudashan/blog-picture/master/testng-learning/b1.svg)

## Parser 文件解析器

![](https://raw.githubusercontent.com/wudashan/blog-picture/master/testng-learning/b2.svg)

## XmlSuite 测试数据

![](https://raw.githubusercontent.com/wudashan/blog-picture/master/testng-learning/b3.svg)

## Listener 监听器

![](https://raw.githubusercontent.com/wudashan/blog-picture/master/testng-learning/b4.svg)

## IAnnotation 注解接口

![](https://raw.githubusercontent.com/wudashan/blog-picture/master/testng-learning/b5.svg)

## SuiteRunner 执行类

![](https://raw.githubusercontent.com/wudashan/blog-picture/master/testng-learning/b6.svg)

# 参考链接
* [TestNG官方文档](https://testng.org/doc/documentation-main.html)
* [TestNG学习笔记（语雀版）](https://www.yuque.com/wudashan/olrmnh/gsmge6)