---
layout:     post
title:      "调用树，性能问题排查神器"
subtitle:   "遇到性能瓶颈如何排查？借助调用树帮你事半功倍"
date:       2019-03-06 21:54:00
author:     "Wudashan"
header-img: "img/post-bg-the-profiler-tree.jpg"
catalog: true
tags:
    - 性能
    - 开源框架
---

> 调用树源码：https://github.com/wudashan/profiler

# 序言

性能这个话题，经常令人谈虎色变。因为我们经常会对关键业务进行性能压测，但是业务代码里可能会涉及各种数据库增删改查，第三方系统RPC调用，消息发送等操作，当发现性能瓶颈的时候我们无法很快的定位到底是哪个具体的操作耗时高。为了解决这个头疼的问题，**调用树**这个性能问题排查神器就可以排上用场。

# 调用树Profiler

所谓调用树，就是可以把一段业务代码，按照树的层级，将涉及到的操作层层展示出来，效果如下：

```
总耗时:175ms 自身耗时:0ms 在总时间里所占时间比:100% 内容:Main主流程
+---耗时:87ms 自身耗时:87ms 在父节点里所占时间比:50% 在总时间里所占时间比:50% 内容:调用RPC接口
+---耗时:34ms 自身耗时:34ms 在父节点里所占时间比:19% 在总时间里所占时间比:19% 内容:保存数据到数据库
`---耗时:54ms 自身耗时:54ms 在父节点里所占时间比:31% 在总时间里所占时间比:31% 内容:发送消息
```

从上面这个信息可以看到，Main主流程里分别进行了调用RPC接口、保存数据到数据库、发送消息操作，并且每个操作的耗时时间可以看得非常清晰，在排查性能瓶颈的时候，我们就可以知道Main主流程总共耗时175ms，其中调用RPC接口耗时87ms，占了总流程的一半时间，那么我们就可以针对这个RPC调用进行优化。如何才能使用调用树呢，心急的同学可以直接看[ProfilerTest](https://github.com/wudashan/profiler/blob/master/src/main/java/profiler/ProfilerTest.java)类。下面，给大家详细讲解调用树提供的API接口和使用方式。

## API接口

调用树Profiler，一共会有以下几个方法供开发者使用：

```
* reset()：重置调用树，清除可能残留的历史数据。

* init()：初始化调用树，在业务代码的入口处调用。

* enter()：进入埋点方法，在需要监控的操作的入口处调用。

* exit()：退出埋点方法，在需要监控的操作的出口处调用。

* dump()：输出调用树结果。
```

## 使用方式

开发者使用方式如下：

```java
public static void main(String[] args) {

    // 重置调用树
    Profiler.reset();
    // 初始化调用树
    Profiler.init("Main主流程");

    try {
        // 业务代码
        doBusiness();
    } finally {
        // 结束调用树
        Profiler.exit();
        // 打印调用树结果
        System.out.println(Profiler.dump());
        // 重置调用树
        Profiler.reset();
    }

}
```

在业务代码里，我们可以对想监控的方法进行埋点，在方法的入口处调用`Profiler.enter()`方法，在方法的出口处调用`Profiler.exit()`方法。示例代码如下：

```java
public static void doBusiness() {

    // 入口埋点
    Profiler.enter();
    // 数据库查询操作
    repository.query();
    // 出口埋点
    Profiler.exit();

    // 入口埋点
    Profiler.enter();
    // 数据库保存操作
    repository.save();
    // 出口埋点
    Profiler.exit();

}
```

`doBusiness()`方法里我们对数据库操作进行了埋点，这样我们就可以知道，查询操作和保存操作的耗时情况。当然如果我们对每个操作的入口出口进行硬编码`enter()`和`exit()`，那么整个项目的代码将非常的难看，稍有不慎可能只调用了`enter()`忘记调用`exit()`，并且每个开发同学还需要显式地感知要埋点，非常的不友好。

## 切面拦截

这个时候，我们就需要借助AOP（面向切面编程）能力，对操作进行拦截，并植入埋点代码。Spring提供了一个很方便的方法拦截器[MethodInterceptor](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/aopalliance/intercept/MethodInterceptor.html)类，拦截实现如下：

```java
public class Interceptor implements MethodInterceptor {
    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        Class clazz = invocation.getMethod().getDeclaringClass();
        String method = invocation.getMethod().getName();
        String mark = clazz.getCanonicalName() + "#" + method;
        // 调用被拦截的方法前，植入入口埋点
        Profiler.enter(mark);
        try {
            // 拦截器调用被拦截的方法
            return invocation.proceed();
        } finally {
            // 调用被拦截的方法后，植入出口埋点
            Profiler.exit();
        }
    }
}
```

同时，我们还需要在配置文件里声明我们需要拦截的方法，XML配置文件如下：

```xml
<bean class="org.springframework.aop.framework.autoproxy.BeanNameAutoProxyCreator">
    <property name="beanNames">
        <list>
            <!-- 对所有以repository结尾命名的类，进行方法拦截 -->
            <value>*repository</value>
        </list>
    </property>
    <property name="interceptorNames">
        <list>
            <!-- 声明我们的拦截器 -->
            <value>Interceptor</value>
        </list>
    </property>
</bean>  
```

通过对数据库层代码、RPC调用代码、消息发送代码进行AOP拦截，使我们不需要硬编码，对开发者来说开发代码的时候也无需特意关注这些点，便能详细明了地打印出整个业务代码中涉及的各个操作及其耗时情况。特别是对于那些有一定历史的项目代码，且代码写的不是很好，代码层层调用与嵌套，有非常显著的效果。

## 实现原理

感兴趣的同学，可以将调用树运用到项目中，见证它无法抗拒的魅力。更感兴趣的同学，让我们来看看调用树底层是如何实现的。

首先，它是对一段代码进行监控，所以它应该是有一个根节点。接着，代码里面每个操作都可以看做一个节点，且每个操作里面可能还有其他操作，所以就有父子节点关系。最后，每个操作中的子操作数量是不确定的。至此，我们可以推导出，调用树底层的数据结构是一个**N叉树**。N叉树的实现主要是一个节点包含一个**父节点**和**子节点列表**。由于我们把每个操作抽象看成一个节点，为了记录操作的耗时时间，所以节点还需要有**开始时间**和**结束时间**。

数据结构确定后，还有一个最重要的一点，就是调用树如何在业务代码中获取和保存当前节点？想想看，业务代码在执行`Profiler.enter()`方法时，并不需要传任何的形参，那么我们如何感知当前节点需要挂在哪棵调用树下面呢？如果你看过一些类似框架代码，例如Spring的数据库连接和Session管理，你就可以马上联想到，是通过**ThreadLocal**线程变量实现。我们将调用树按线程进行隔离，在调用Profiler的API时，我们从当前线程取出对应的调用树进行操作。

总结来说，调用树就是通过**N叉树+ThreadLocal线程变量**实现，有能力的同学可以尝试自己实现Profiler的API：reset()、init()、enter()、exit()、dump()。实现完成之后，可以对照参考[阿里的Profiler](https://github.com/alipay/sofa-common-tools/blob/1e120ebbe10b15eed36e480aa858d7554a10db23/core/src/main/java/com/alipay/sofa/common/profile/diagnostic/Profiler.java)和[我写的Profiler](https://github.com/wudashan/profiler/blob/master/src/main/java/profiler/Profiler.java)代码实现，看看谁的实现更优雅更美好。