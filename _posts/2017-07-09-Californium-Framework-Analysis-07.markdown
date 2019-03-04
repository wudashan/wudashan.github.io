---
layout:     post
title:      "Californium开源框架之源码分析（七）"
subtitle:   "core包，一些封装好的供开发者使用的类。"
date:       2017-07-09 21:00:00
author:     "Wudashan"
header-img: "img/post-bg-californium-framework-analysis.jpg"
catalog: true
tags:
    - Californium
    - 开源框架
    - 源码分析
    - CoAP
---

> 项目源码地址：[https://github.com/eclipse/californium](https://github.com/eclipse/californium)

# core包

core根目录下，封装好了一些供开发者使用的类。

![](https://raw.githubusercontent.com/wudashan/blog-picture/master/californium-framework-analysis-07/core.png)

## 根目录

### CaliforniumFormatter类

该类继承了`java.util.logging.Formatter`类，并重写了`format()`方法。对日志输出进行了格式化处理，默认给每条日志添加线程id、日志级别、类名、详细信息、源码行号、方法名、线程名信息，方便定位问题。

### CaliforniumLogger类

当`java.util.logging`没有使用配置文件时，该类可以作为一个辅助类控制`org.eclipse.californium.core`和`org.eclipse.californium.elements`包下的日志输出格式和级别。日志输出格式使用的是CaliforniumFormatter类。

### CoapHandler接口

该类用于`CoapClient`异步发送消息，需要对消息的后续结果进行处理的场景。当客户端收到响应时，回调`onLoad()`方法；当客户端发送的请求超时或被服务端拒绝时，回调`onError()`方法。

### CoapObserveRelation类

该类用于`CoapClient`处理订阅关系，它表示一个客户端和服务端资源的订阅关系。该类提供了几个简单的public方法，如检测订阅关系是否建立或取消，重新注册以刷新订阅关系。

### CoapClient类

该类表示CoAP客户端，提供了很多方法供开发者使用，主要有`postXXX()`、`getXXX()`、`putXXX()`、`deleteXXX()`方法，同步和异步发送请求，发起订阅请求、发起ping请求和发起discover请求等等。

### CoapResource类

该类在[《Californium开源框架之源码分析（四）—— server包》](http://wudashan.cn/2017/06/16/Californium-Framework-Analysis-04/)已分析过。

### CoapResponse类

该类将`Response`类进行了包装，并只提供了几个简单的public方法，如果需要直接读取其成员变量`Response`，则需要调用`advanced()`方法。


### CoapServer类

该类实现了`ServerInterface`接口，代表着包含CoAP资源的服务端。其结构如下图：

```
+------------------------------------- CoapServer --------------------------------------+
|                                                                                       |
|                               +-----------------------+                               |
|                               |    MessageDeliverer   +--> (Resource Tree)            |
|                               +---------A-A-A---------+                               |
|                                         | | |                                         |
|                                         | | |                                         |
|                 .-------->>>------------' | '--------<<<------------.                 |
|                /                          |                          \                |
|               |                           |                           |               |
|             * A                         * A                         * A               |
| +-----------------------+   +-----------------------+   +-----------------------+     |
| |        Endpoint       |   |        Endpoint       |   |      Endpoint         |     |
| +-----------------------+   +-----------------------+   +-----------------------+     |
+------------v-A--------------------------v-A-------------------------v-A---------------+
             v A                          v A                         v A            
             v A                          v A                         v A         
          (Network)                    (Network)                   (Network)
```

服务端持有一个树形结构的资源Resource，还包括多个绑定了网络端口的端点Endpoint。服务端可以启动和停止。当服务端停止时，端点需要释放其占用的端口，但是线程池需要保持运行以确保还能继续启动。

通过如下代码，可以创建一个服务端，当客户端发起GET请求时，响应“hello world”：

```
CoapServer server = new CoapServer(port);
server.add(new CoapResource("hello-world") {
    public void handleGET(CoapExchange exchange) {
        exchange.respond(ResponseCode.CONTENT, "hello world");
    }
});
server.start();
```

### Utils类

该类是一个工具类，提供了字节数组转十六进制的方法，还有美化请求与响应的内容打印。

### WebLink类

该类表示一个服务端的Resource对应的uri和属性，当调用`CoapClient.discover()`方法，会返回一个包含WebLink的列表。

---

# 系列文章

[Californium开源框架之源码分析（一）—— 整体认识](http://wudashan.cn/2017/05/21/Californium-Framework-Analysis-01/) 

[Californium开源框架之源码分析（二）—— coap包](http://wudashan.cn/2017/06/01/Californium-Framework-Analysis-02/) 

[Californium开源框架之源码分析（三）—— observe包](http://wudashan.cn/2017/06/05/Californium-Framework-Analysis-03/) 

[Californium开源框架之源码分析（四）—— server包](http://wudashan.cn/2017/06/16/Californium-Framework-Analysis-04/) 

[Californium开源框架之源码分析（五）—— network包（上）](http://wudashan.cn/2017/07/02/Californium-Framework-Analysis-05/) 

[Californium开源框架之源码分析（六）—— network包（下）](http://wudashan.cn/2017/07/07/Californium-Framework-Analysis-06/) 

[Californium开源框架之源码分析（七）—— core包](http://wudashan.cn/2017/07/09/Californium-Framework-Analysis-07/) <-- 当前位置

[Californium开源框架之源码分析（八）—— element包](http://wudashan.cn/2017/07/12/Californium-Framework-Analysis-08/)