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

性能这个话题，经常令人谈虎色变。因为我们经常会对关键业务流程进行性能压测，但是流程的代码里可能会涉及各种数据库增删改查，第三方系统RPC调用，消息发送等操作，当发现性能瓶颈的时候我们无法很快的定位到底是哪个具体的操作耗时高。为了解决这个头疼的问题，**调用树**这个性能问题排查神器就可以排上用场。

# 调用树

所谓调用树，就是可以把一段流程的代码，按照树的层级，将涉及到的操作层层展示出来，效果如下：

```
总耗时:175ms 自身耗时:0ms 在总时间里所占时间比:100% 内容:Main主流程
+---耗时:87ms 自身耗时:87ms 在父节点里所占时间比:50% 在总时间里所占时间比:50% 内容:调用RPC接口
+---耗时:34ms 自身耗时:34ms 在父节点里所占时间比:19% 在总时间里所占时间比:19% 内容:保存数据到数据库
`---耗时:54ms 自身耗时:54ms 在父节点里所占时间比:31% 在总时间里所占时间比:31% 内容:发送消息
```

从上面这个信息可以看到，Main主流程里分别进行了RPC调用、数据库操作、消息发送，并且每个操作的耗时时间可以看得非常清晰，在排查性能瓶颈的时候，我们就可以知道流程总共耗时175ms，其中调用RPC接口耗时87ms，占了总流程的一半时间，那么我们就可以针对这个RPC调用进行优化。如何才能使用调用树呢，心急的同学可以直接看[ProfilerTest](https://github.com/wudashan/profiler/blob/master/src/main/java/profiler/ProfilerTest.java)类。

调用树，一共会有以下几个方法供开发者使用：

```
* reset()：重置调用树，清除可能残留的历史数据。

* init()：初始化调用树，在关键流程的入口处调用。

* enter()：进入埋点方法，在需要监控的操作的入口处调用。

* exit()：退出埋点方法，在需要监控的操作的出口处调用。

* dump()：输出调用树结果。
```

开发者调用方式如下：

```
public static void main(String[] args) {

    // 重置调用树
    Profiler.reset();
    // 初始化调用树
    Profiler.init("Main主流程");

    try {
        // 关键流程代码
        ...
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

在关键流程代码，我们可以对想监控的方法进行埋点，在方法的入口处调用`Profiler.enter()`方法，在方法的出口处调用`Profiler.exit()`方法。

未完待续，持续更新ing。。。

