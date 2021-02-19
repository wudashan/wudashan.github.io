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

通过Java内置的ThreadPoolExecutor线程池实现多线程执行测试用例。并且支持suite/tests/classes/methods/instances五种维度的多线程场景：suite多线程实现在org.testng.TestNG#runSuitesLocally，tests多线程实现在org.testng.SuiteRunner#runInParallelTestMode，classes/methods/instances多线程实现都在org.testng.TestRunner#privateRun，后三者区别在于通过org.testng.TestRunner#createWorkers创建的Work数量不同：classes场景该类的所有被测方法都在一个Work里串行执行，methods场景每个被测方法自己单独一个Work，instances场景每个被测方法实例单独一个Work。

## 灵活的测试配置功能

**TestNG如何解析命令行参数？**

使用JCommander第三方框架，解析main入口函数里用户通过命令行传入的args参数，并转成CommandLineArgs对象。

**TestNG如何解析testng.xml配置文件？**

实现了一个[Parser文件处理器](http://wudashan.com/2020/09/13/testng-learning/#parser-%E6%96%87%E4%BB%B6%E8%A7%A3%E6%9E%90%E5%99%A8)，支持对xml和yaml格式的配置文件进行解析，通过类继承关系可以知道TestNG支持通过SAX和DOM两种方式解析xml文件。

## 支持数据驱动的测试功能

**TestNG如何支持参数化执行用例？**

通过找到@DataProvider注解的方法，执行该方法并返回List<Object[]>对象（外层List代表被测方法要执行的次数，内层Object[]代表每次执行被测方法时传入的形参），或testng.xml里的`<parameter>`参数，得到参数列表，并在反射调用用例时传入参数。

## 其他功能

**TestNG如何展示用例执行结果？**

定义了IReporter接口，在用例执行结束后，回调其generateReport方法，并将整个用例结果SuiteResult传给该方法。

**TestNG如何解决测试用例之间的依赖顺序？**

通过**图**这个数据结构，将A方法依赖B方法，转义成A->B的单向图，实现用例之间存在依赖时，调用的先后顺序。

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

## ThreadPoolExecutor 线程池

![](https://raw.githubusercontent.com/wudashan/blog-picture/master/testng-learning/b7.svg)

## DynamicGraph 图数据结构

![](https://raw.githubusercontent.com/wudashan/blog-picture/master/testng-learning/b8.svg)

## IReporter 执行结果

![](https://raw.githubusercontent.com/wudashan/blog-picture/master/testng-learning/b9.svg)

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

## 图数据结构找无依赖的方法

```java
// 图由节点和边组成
public class DynamicGraph<T> {
	
  // Set记录节点，这里区分节点3个状态，因为已完成的节点可认为不再依赖
  private Set<T> m_nodesReady = Sets.newLinkedHashSet();
  private Set<T> m_nodesRunning = Sets.newLinkedHashSet();
  private Set<T> m_nodesFinished = Sets.newLinkedHashSet();
  // Map记录边
  private ListMultiMap<T, T> m_dependedUpon = Maps.newListMultiMap();
  private ListMultiMap<T, T> m_dependingOn = Maps.newListMultiMap();
	
  // 往图中增加节点
  public void addNode(T node) {
    m_nodesReady.add(node);
  }
	
  // 往图中增加边
  public void addEdge(T from, T to) {
    addNode(from);
    addNode(to);
    m_dependingOn.put(to, from);
    m_dependedUpon.put(from, to);
  }
	
  // 获取没有依赖的节点
  public List<T> getFreeNodes() {
    List<T> result = Lists.newArrayList();
    for (T m : m_nodesReady) {
      // 一个节点如何是“自由的”，那应该它没有依赖任何节点，或者依赖的节点状态都是已完成
      List<T> du = m_dependedUpon.get(m);
      if (!m_dependedUpon.containsKey(m)) {
        result.add(m);
      } else if (getUnfinishedNodes(du).size() == 0) {
        result.add(m);
      }
    }
    return result;
  }
  
  // 获取未到达终态的节点列表
  private Collection<? extends T> getUnfinishedNodes(List<T> nodes) {
    Set<T> result = Sets.newHashSet();
    for (T node : nodes) {
      if (m_nodesReady.contains(node) || m_nodesRunning.contains(node)) {
        result.add(node);
      }
    }
    return result;
  }
  
  // 设置节点状态
  public void setStatus(T node, Status status) {
    // 先将节点从原集合Set中删除
    removeNode(node);
    // 再插入到对应状态的新集合里
    switch(status) {
      case READY: m_nodesReady.add(node); break;
      case RUNNING: m_nodesRunning.add(node); break;
      case FINISHED: m_nodesFinished.add(node); break;
      default: throw new IllegalArgumentException();
    }
  }

  // 删除节点
  private void removeNode(T node) {
    // 这种代码有点难理解，就是三个集合依次删除，成功就返回
    if (!m_nodesReady.remove(node)) {
      if (!m_nodesRunning.remove(node)) {
        m_nodesFinished.remove(node);
      }
    }
  }
	
}
```

# 参考链接
* [TestNG官方文档](https://testng.org/doc/documentation-main.html)
* [TestNG学习笔记（语雀版）](https://www.yuque.com/wudashan/olrmnh/gsmge6)
