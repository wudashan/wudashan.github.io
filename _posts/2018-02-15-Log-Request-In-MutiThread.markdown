---
layout:     post
title:      "如何快速过滤出一次请求的所有日志？"
subtitle:   "使用SLF4J日志框架的MDC工具，将请求ID植入一次请求的生命周期。"
date:       2018-02-15 14:30:00
author:     "Wudashan"
header-img: "img/post-bg-log-requset-in-mutithread.jpg"
catalog: true
tags:
    - 日志
    - Log4j
    - SLF4J
---

> 示例源码地址：`https://github.com/wudashan/slf4j-mdc-muti-thread`

# 前言

在现网出现故障时，我们经常需要获取一次请求流程里的所有日志进行定位。如果请求只在一个线程里处理，则我们可以通过线程ID来过滤日志，但如果请求包含异步线程的处理，那么光靠线程ID就显得捉襟见肘了。

华为IoT平台，提供了接收设备上报数据的能力， 当数据到达平台后，平台会进行一些复杂的业务逻辑处理，如数据存储，规则引擎，数据推送，命令下发等等。由于这个逻辑之间没有强耦合的关系，所以通常是异步处理。如何将一次数据上报请求中包含的所有业务日志快速过滤出来，就是本文要介绍的。

# 正文

SLF4J日志框架提供了一个MDC(Mapped Diagnostic Contexts)工具类，谷歌翻译为**映射的诊断上下文**，从字面上很难理解，我们可以先实战一把。

```
public class Main {

    private static final String KEY = "requestId";
    private static final Logger logger = LoggerFactory.getLogger(Main.class);
    
    public static void main(String[] args) {

        // 入口传入请求ID
        MDC.put(KEY, UUID.randomUUID().toString());
        
        // 打印日志
        logger.debug("log in main thread 1");
        logger.debug("log in main thread 2");
        logger.debug("log in main thread 3");

        // 出口移除请求ID
        MDC.remove(KEY);

    }

}

```

我们在main函数的入口调用`MDC.put()`方法传入请求ID，在出口调用`MDC.remove()`方法移除请求ID。配置好**log4j2.xml**文件后，运行main函数，可以在控制台看到以下日志输出：

```
2018-02-17 13:19:52.606 {requestId=f97ea0fb-2a43-40f4-a3e8-711f776857d0} [main] DEBUG cn.wudashan.Main - log in main thread 1
2018-02-17 13:19:52.609 {requestId=f97ea0fb-2a43-40f4-a3e8-711f776857d0} [main] DEBUG cn.wudashan.Main - log in main thread 2
2018-02-17 13:19:52.609 {requestId=f97ea0fb-2a43-40f4-a3e8-711f776857d0} [main] DEBUG cn.wudashan.Main - log in main thread 3
```

从日志中可以明显地看到花括号中包含了**（映射的）**请求ID(requestId)，这其实就是我们定位**（诊断）**问题的关键字**（上下文）**。有了MDC工具，只要在接口或切面植入`put()`和`remove()`代码，在现网定位问题时，我们就可以通过`grep requestId=xxx *.log`快速的过滤出某次请求的所有日志。

# 进阶

然而，MDC工具真的有我们所想的这么方便吗？回到我们开头，一次请求可能涉及多线程异步处理，那么在多线程异步的场景下，它是否还能正常运作呢？Talk is cheap, show me the code。

```
public class Main {

    private static final String KEY = "requestId";
    private static final Logger logger = LoggerFactory.getLogger(Main.class);

    public static void main(String[] args) {

        // 入口传入请求ID
        MDC.put(KEY, UUID.randomUUID().toString());

        // 主线程打印日志
        logger.debug("log in main thread");

        // 异步线程打印日志
        new Thread(new Runnable() {
            @Override
            public void run() {
                logger.debug("log in other thread");
            }
        }).start();

        // 出口移除请求ID
        MDC.remove(KEY);

    }

}
```

代码里我们新起了一个异步线程，并在匿名对象Runnable的run()方法打印日志。运行main函数，可以在控制台看到以下日志输出：

```
2018-02-17 14:05:43.487 {requestId=e6099c85-72be-4986-8a28-de6bb2e52b01} [main] DEBUG cn.wudashan.Main - log in main thread
2018-02-17 14:05:43.490 {} [Thread-1] DEBUG cn.wudashan.Main - log in other thread
```

不幸的是，请求ID在异步线程里不打印了。这是怎么回事呢？要解决这个问题，我们就得知道MDC的实现原理。由于篇幅有限，这里就暂不详细介绍，MDC之所以在异步线程中不生效是因为底层采用**ThreadLocal**作为数据结构，我们调用`MDC.put()`方法传入的请求ID只在当前线程有效。感兴趣的小伙伴可以自己深入一下代码细节。

---
