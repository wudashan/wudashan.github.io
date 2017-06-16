---
layout:     post
title:      "Californium开源框架之源码分析（四）"
subtitle:   "server模块，服务端和资源。"
date:       2017-06-16 14:00:00
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

# server包

server包目录下，描述的是服务端和内嵌的资源。

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/server%E5%8C%85_01.png)

---

# 根目录

### ServerInterface接口

该接口描述了一个服务端需要提供什么功能：CoAP资源的运行环境。

资源是以N叉树的数据结构表示的，服务端只持有根节点。一个服务端可以绑定多个Endpoint，只要ip地址和端口号够用。

在设计ServerInterface的实现类的时候，应该允许资源可以动态地添加和删除到服务端。当调用服务端的`start()`方法之后，可以开始处理CoAP请求；当调用服务端的`stop()`方法之后，将停止处理新的请求。

根据以上描述的功能，该接口声明了对应的方法：

```
public interface ServerInterface {
    
    // 启动、停止、销毁服务端
    void start();
    void stop();
    void destroy();
    
    // 添加、删除资源
    ServerInterface add(Resource... resources);
    boolean remove(Resource resource);
    
    // 添加、获取端点
    void addEndpoint(Endpoint endpoint);
    Endpoint getEndpoint(...);

}

```

该接口的是实现类是`CoapServer类`，但是不在这个目录下，后续我们再详细了解。

### MessageDeliverer接口

该接口负责将接收到的CoAP消息分发给合适的对象去处理。该接口只有2个方法需要实现：

```
public interface MessageDeliverer {
    // 分发请求消息给对应的资源
    void deliverRequest(Exchange exchange);
  
    // 分发响应消息给对应的请求
    void deliverResponse(Exchange exchange, Response response);
}
```

实现类需要根据请求消息的URI来分发给对应的资源。如果找不到对应的资源，实现类应该返回`4.04 NOT_FOUND`响应码。如果实现类接收到的是响应消息，则应该分发给对应的请求消息。

### ServerMessageDeliverer类

该类实现MessageDeliverer接口，用于服务端分发CoAP消息。

当接收到请求消息时，该类根据URI找到对应的资源，并检查是否是订阅请求，若是订阅请求还需要保存订阅关系，这样在资源发生变化时，可以通过保存的订阅关系通知对应的客户端。找到资源后，将请求传递给资源，由资源处理请求。源码如下：

```
public void deliverRequest(final Exchange exchange) {

    // 从exchange里获取request
    Request request = exchange.getRequest();
    
    // 从request里获取请求路径
    List<String> path = request.getOptions().getUriPath();
    
    // 找出请求路径对应的Resource
    final Resource resource = findResource(path);
    
    // 检查是否是订阅请求，是的话还需要保存订阅关系
    checkForObserveOption(exchange, resource);
    
    // 由Resource来真正地处理请求
    resource.handleRequest(exchange);
    
    // 一些非关键操作
    ...
    
}
```

当接收到响应消息时，该类将匹配对应的请求消息。源码如下：

```
public void deliverResponse(Exchange exchange, Response response) {
    exchange.getRequest().setResponse(response);
}
```

## resources包

