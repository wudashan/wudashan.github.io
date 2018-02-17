---
layout:     post
title:      "如何快速过滤出一次请求的所有日志？"
subtitle:   "使用SLF4J日志框架的MDC工具，将请求标识植入一次请求的生命周期。"
date:       2018-02-15 14:30:00
author:     "Wudashan"
header-img: "img/post-bg-log-requset-in-mutithread.jpg"
catalog: true
tags:
    - 日志
    - Log4j
    - 请求标识
---

> 示例源码地址：`https://github.com/wudashan/slf4j-mdc-muti-thread`

# 前言

在现网出现故障时，我们经常需要获取一次请求流程里的所有日志进行定位。如果请求只在一个线程里处理，则我们可以通过线程ID来过滤日志，但如果请求包含异步线程的处理，那么光靠线程ID就显得捉襟见肘了。

华为IoT平台，提供了接收设备上报数据的能力， 当数据到达平台后，平台会进行一些复杂的业务逻辑处理，如数据存储，规则引擎，数据推送，命令下发等等。由于这个逻辑之间没有强耦合的关系，所以通常是异步处理。如何将一次数据上报请求中包含的所有业务日志快速过滤出来，就是本文要介绍的。

# 正文

SLF4J日志框架提供了一个MDC(Mapped Diagnostic Contexts)工具类，谷歌翻译为映射的诊断上下文，从字面上很难理解，我们可以先实战一把。

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

从日志中可以明显地看到花括号中包含了（映射的）请求ID(requestId)，这其实就是我们定位（诊断）问题的关键字（上下文）。在现网定位问题时，我们可以通过`grep requestId=xxx *.log`快速的过滤出某次请求的所有日志。

---
