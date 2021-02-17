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

通过TestNG框架的官方介绍，我们知道了它主要提供了哪些功能，对应的我们需要通过几个问题去理解其如何实现（原理）？

## 注解功能

**TestNG如何发现需要被测试的方法？**

通过Java两大特性，注解+反射，找到被测方法。具体原理为先通过main函数入参或testng.xml配置文件获取需要扫描的类，再通过反射获取类信息，判断是否有@Test注解，如果有则表示该类的方法需要测试。

**TestNG如何支持用户感知框架运行时状态？**

通过开放各种Listener接口（父类为org.testng.ITestNGListener），如IExecutionListener、IConfigurationListener、IInvokedMethodListener等，并在运行时进行回调，使用户感知当前运行状态。

## 线程池中运行测试用例功能

**TestNG如何支持多线程执行测试用例？**

## 灵活的测试配置功能

**TestNG如何解析命令行参数？**

**TestNG如何解析testng.xml配置文件？**

## 支持数据驱动的测试功能

**TestNG如何支持参数化执行用例？**

## 其他功能

**TestNG如何展示用例执行结果？**

**TestNG如何解决测试用例之间的依赖顺序？**



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

# 经典代码

## 反射获取Class类

```java
// org.testng.internal.ClassHelper#forName
public static Class<?> forName(final String className) {
  // 获取类加载器集合
  Vector<ClassLoader> allClassLoaders = new Vector<ClassLoader>();
  ClassLoader contextClassLoader = Thread.currentThread().getContextClassLoader();
  if (contextClassLoader != null) {
    allClassLoaders.add(contextClassLoader);
  }
  if (m_classLoaders != null) {
    allClassLoaders.addAll(m_classLoaders);
  }
  
  // 遍历类加载器，看谁能加载成功类
  int count = 0;
  for (ClassLoader classLoader : allClassLoaders) {
    ++count;
    if (null == classLoader) {
      continue;
    }
    try {
      return classLoader.loadClass(className);
    }
    catch(ClassNotFoundException ex) {
      // With additional class loaders, it is legitimate to ignore ClassNotFoundException
      if (null == m_classLoaders || m_classLoaders.size() == 0) {
        logClassNotFoundError(className, ex);
      }
    }
  }
  // 问题1：Class.forName() 和 ClassLoader.loadClass() 有什么不同？
  // 答案：https://stackoverflow.com/questions/8100376/class-forname-vs-classloader-loadclass-which-to-use-for-dynamic-loading

  // 问题2：Class.forName() 使用哪个类加器进行加载？
  // 答案：默认会使用调用类的类加载器来进行类加载，顺便理解双亲委派机制，（双亲是哪双亲？）。
  try {
    return Class.forName(className);
  }
  catch(ClassNotFoundException cnfe) {
    logClassNotFoundError(className, cnfe);
    return null;
  }
}
```

## Java SPI获取Listener实现类

```java
// org.testng.TestNG#addServiceLoaderListeners
private void addServiceLoaderListeners() {
  Iterable<ITestNGListener> loader;
  try {
    if (m_serviceLoaderClassLoader != null) {
      // spi原理：加载META-INF/services/路径下的文件
      // 文件名是接口，文件内容每行是实现类，反射创建实现类实例，并强转成接口
      // 使用到了懒加载机制
      loader = ServiceLoader.load(ITestNGListener.class, m_serviceLoaderClassLoader);
    } else {
      loader = ServiceLoader.load(ITestNGListener.class);
    }
    for (ITestNGListener l : loader) {
      addListener(l);
      addServiceLoaderListener(l);
    }
  } catch (Exception ex) {
      // Ignore
  }
}
```

# 参考链接
* [TestNG官方文档](https://testng.org/doc/documentation-main.html)
* [TestNG学习笔记（语雀版）](https://www.yuque.com/wudashan/olrmnh/gsmge6)