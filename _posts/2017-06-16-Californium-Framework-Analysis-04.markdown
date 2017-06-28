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

## 根目录

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

当接收到请求消息时，该类根据URI找到对应的资源，并检查请求是否与订阅相关，如果是建立订阅则保存订阅关系，如果是取消订阅则删除订阅关系。找到资源后，将请求传递给资源，由资源处理请求。源码如下：

```
public void deliverRequest(final Exchange exchange) {

    // 从exchange里获取request
    Request request = exchange.getRequest();
    
    // 从request里获取请求路径
    List<String> path = request.getOptions().getUriPath();
    
    // 找出请求路径对应的Resource
    final Resource resource = findResource(path);
    
    // 检查是否是订阅相关的请求，是的话需要保存或删除订阅关系
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

### ResourceObserver接口

这个接口使用的是设计模式中的观察者模式。通过`Resource.addObserver()`添加观察者。当资源的事件触发时会回调ResourceObserver接口的对应方法，该接口支持如下事件回调：

 - `changedName()` // 当资源的名称发生变化
 - `changedPath()` // 当资源的路径发生变化
 - `addedChild()` // 当资源添加了新的子资源
 - `removedChild()` // 当资源的子资源被移除
 - `addedObserveRelation()` // 当资源添加了新的订阅关系
 - `removedObserveRelation()` // 当资源的订阅关系呗移除
 

### ResourceAttributes类

该类表示资源的属性，其数据结构如下图：

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/ResourceAttributes.png)

ResourceAttributes类包含CoAP协议定义的不同属性，如标题，资源类型或接口描述。这些属性也将包含在它们所属资源的链接描述中。例如，如果资源指定了标题，它的链接描述可能看起来像这样`</sensors>; title =“标题党”`。

### RequestProcessor接口

该接口只有一个`processRequest(Exchange exchange)`方法，但是并没有在框架中使用到，估计是发现与`Resource.handleRequest(Exchange exchange)`方法意思相同，所以就废弃了。

### CoapExchange类

该类包含一个Exchange类（表示一对请求和响应）和一个CoapResource类。CoapExchange类把它们封装了起来，并提供友好的API供开发者响应请求消息。

有多友好？该类一共提供了7个`respond(...)`响应请求消息的同名方法，就是为了方便开发者，可以直接调用所需要的方法。不过，其中6个都是进行简单地封装，然后调用第7个，所以我们可以直接看第7个响应方法：

```
public void respond(Response response) {
        
    // 检查入参
    ...
    
    // 若成员变量不为null，则赋值到response对象中
    if (locationPath != null) response.getOptions().setLocationPath(locationPath);
    if (locationQuery != null) response.getOptions().setLocationQuery(locationQuery);
    if (maxAge != 60) response.getOptions().setMaxAge(maxAge);
    if (eTag != null) {
        response.getOptions().clearETags();
        response.getOptions().addETag(eTag);
    }
        
    // 检查是否建立了订阅关系，是的话资源需要保存订阅关系
    resource.checkObserveRelation(exchange, response);
    
    // 发送响应
    exchange.sendResponse(response);
    
}
```

当然该类还提供了`accept()`方法和`reject()`方法响应请求。总之，对比Exchange类和CoapResource类，你会发现还是CoapExchange类简单易上手，因为它屏蔽了不必要的细节。


### Resource接口

资源是服务端中资源树的一个子节点，由于是资源是以N叉树的数据结构存在，所以一个资源可能也有自己的子节点。资源是服务端的重要组成部分（处理请求的类，能不重要吗=、=）。一个资源必须要有自己的名称，并且它的URI是根据它的父节点名称 + 自己的名称组成的。具体定义可以看下面这个例子：

```
若树结构表示如下：
  根
  |
  |-- foo
  |    `-- bar
  |         `-- bal
 
则对于资源bal来说：
  bal.getName() = "bal"
  bal.getPath() = "/foo/bar/"
  bal.getURI()  = "/foo/bar/bal"
```

资源用于响应CoAP请求。请求被封装在`Exchange`类，该类包含着一些请求的额外信息。尽管CoAP协议里允许分块传输请求，但是到达Resource接口的都是完整的请求体。

当一个请求到达服务端，`ServerMessageDeliverer`类会根据请求的URI搜索到对应的资源类，原理是通过调用`Resource.getChild(String)`获取子节点。当然，我们也可以覆盖这个方法来获取想要的子节点。比如请求的URI中包含通配符也可找到对应资源，或某些URI前缀都相同时交由同一个资源处理。

一个资源可以有它自己的线程池。如果一个资源拥有自己的线程池，则请求将在它的线程池中被处理；如果没有，则转交由父资源的线程池处理。如果所有的父类都没有线程池，则由主线程处理请求。

`CoapResource类`是`Resource接口`的实现类，奇怪的是，框架并没有把它放在同一个目录下。为了方便，下面一并介绍`CoapResource类`。


### CoapResource类

CoapResource是Resource的基本实现类，我们可以继承它来编写自己的资源类。通过调用`CoapResource.add(CoapResource child)`方法可以很方便的构造出一个树形结构的资源类。

CoapResource通过4个不同的方法处理请求：`handleGET()`、`handlePOST()`、`handlePUT()`和`handleDELETE()`。4个方法都有一个默认的实现，即返回4.05（方法不支持），我们可以根据需要覆盖这4个方法：
```
public class CoAPResourceExample extends CoapResource {
 
    public CoAPResourceExample(String name) {
        super(name);
    }

    @Override
    public void handleGET(CoapExchange exchange) {
        exchange.respond("hello world");
    }
 
    @Override 
    public void handlePOST(CoapExchange exchange) {
        exchange.accept();
        exchange.respond(ResponseCode.CREATED);
    }
 
    @Override
    public void handlePUT(CoapExchange exchange) {
        exchange.respond(ResponseCode.CHANGED);
        changed();  // 通知订阅方
   }
 
    @Override
    public void handleDELETE(CoapExchange exchange) {
        delete();
        exchange.respond(ResponseCode.DELETED);
    }
}
```

CoapResource支持CoAP协议的订阅机制。如果调用了`setObservable(true)`方法，则该资源可以被CoAP客户端订阅。资源通过调用`changed()`方法表示自己已发生变化，并通知给订阅的客户端。`changed()`方法将会重新处理之前客户端发起的订阅请求。如果资源有自己的线程池，则请求会被线程池处理。`ObserveRelation类`表示资源和客户端的订阅关系，这在[Californium开源框架之源码分析（三）](http://wudashan.cn/2017/06/05/Californium-Framework-Analysis-03/)已经介绍过了。

`ResourceObserver接口`与`CoapResource类`紧密相连，使用的是观察者设计模式。当资源发生变化时，ResourceObserver接口的方法将被回调。


