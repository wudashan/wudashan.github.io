---
layout:     post
title:      "Californium开源框架之源码分析（六）"
subtitle:   "network模块（下），网络传输核心模块。"
date:       2017-07-07 22:00:00
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

# network包

network包目录下，是框架中网络传输的核心模块。

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/network%E5%8C%85.png)

## stack包

该模块为CoAP协议栈，模块一共有以下类：

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/network-stack%E5%8C%85.png)

### Layer接口

该接口表示抽象的协议层，负责处理CoAP消息。当处理完后，会将消息往上层或下层传递，由它们继续处理。使用的是设计模式中的职责链模式。

当`CoapEndpoint`收到消息时，它会把消息传给最底层的协议层，并回调`receiveXXX()`方法，每个协议层在处理完消息后决定是否将消息传给上层协议层。最上层的协议层会将消息传给`MessageDeliverer`。

当`CoapEndpoint`发送消息时，它会把消息传给最上层的协议层，并回调`sendXXX()`方法，每个协议层在处理完消息后决定是否将消息传给下层协议层。最下层的协议层会将消息传给`Outbox`。

`Exchange`包含着一对请求与响应的所有信息，协议层可以并发地修改Exchange。虽然多数情况下是单线程地修改Exchange，但是如果是多线程的场景，需要对Exchange做同步修改的保护。

每一协议层可以有自己的线程池，处理自己的任务，比如重传等等。

该接口还提供了一个`TopDownBuilder`类，用来生成协议栈。


### AbstractLayer类

该类是一个抽象类，实现了Layer接口的所有方法，`receiveXXX()`和`sendXXX()`方法默认将消息传递给下一个协议层处理。子类可以覆盖相应的方法并通过`super.receiveXXX()`或`super.sendXXX()`传递消息。

### ObserveLayer类

该类处理订阅相关的CoAP消息。

### BlockwiseLayer类

当传输的CoAP消息过大时，该层负责分解或组装消息。

### BlockwiseStatus类

该类表示一个请求消息或响应消息的分块传输的状态。该类的实例对象保存在Exchange里，并且只能被BlockwiseLayer访问和修改。

### ReliabilityLayer类

该类负责CON类型的消息的可靠性重传。

### CongestionControlLayer类

该类继承ReliabilityLayer类，在可靠性重传的基础上做了拥塞控制。该类为抽象类，将拥塞控制的一些公共方法封装了起来。`Californium.properties`配置文件默认是不开启拥塞控制的。该类还提供了一个`newImplementation(NetworkConfig config)`静态工厂方法，根据配置信息返回一个具体的拥塞控制子类。

### congestioncontrol包

该目录下有`BasicRto`、`Cocoa`、`CocoaStrong`、`LinuxRto`、`PeakhopperRto`5个类，都继承了CongestionControlLayer抽象类，实现了具体的拥塞控制策略，每个类各有区别。由于本人能力有限，这里就不展开篇幅介绍，以防误导大家。

### CoapStack类

CoapStack类组装了上面介绍的几个协议层，形成了协议栈，用于处理CoAP消息。在框架中，完整的消息接收和发送处理过程如下图：

```
 +--------------------------+
 | MessageDeliverer         |
 +--------------A-----------+
                A
              * A
 +------------+-A-----------+
 |       CoAPEndpoint       |
 |            v A           |
 |            v A           |
 | +----------v-+---------+ |
 | | Stack Top            | |
 | +----------------------+ |
 | | ObserveLayer         | |
 | +----------------------+ |
 | | BlockwiseLayer       | |
 | +----------------------+ |
 | | ReliabilityLayer     | |
 | +----------------------+ |
 | | Stack Bottom         | |
 | +----------+-A---------+ |
 |            v A           |
 |          Matcher         |
 |            v A           |
 |        Interceptor       |
 |            v A           |
 +------------v-A-----------+
              v A 
              v A 
 +------------v-+-----------+
 | Connector                |
 +--------------------------+
 
```

其中`Stack Top`与`Stack Bottom`之间都属于CoapStack类。当`Stack Top`要向下发送消息时直接传递给下层，当接收到消息时向上传递给`MessageDeliverer`处理。当`Stack Bottom`要向下发送消息时传递给`Outbox`处理，当接收到消息时直接传递给上层。而`ObserveLayer`、`BlockwiseLayer`、`ReliabilityLayer`则各自履行自己的责任。

## 根目录

network根目录下，还有下面几个类：

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/network-%E6%A0%B9%E7%9B%AE%E5%BD%95.png)

### EndpointObserver接口

该接口使用观察者设计模式，通过`Endpoint.addObserver()`方法注册并监听Endpoint的事件：

```
public void started(Endpoint endpoint); // 当Endpoint启动后回调
public void stopped(Endpoint endpoint); // 当Endpoint停止后回调
public void destroyed(Endpoint endpoint); // 当Endpoint销毁后回调
```

### Endpoint接口

一个Endpoint需要绑定指定的ip和port。客户端使用Endpoint来发送请求给服务端；服务端的资源与Endpoint相连接，处理来自客户端的请求。我们可以抽象地把它看成一个消息出入口。

### EndpointManager类

一个提供创建Endpoint对象的工厂类。通过`getDefaultEndpoint()`方法，可以获得一个端口号随机的Endpoint对象，可用于客户端发送请求。CoAP协议是明文传输的，想要加密传输就得使用CoAPs协议，其底层使用的是DTLS加密。虽然EndpointManager提供了`getDefaultSecureEndpoint()`方法获取加密的Endpoint对象，但是该方法默认返回null，需要调用`setDefaultSecureEndpoint(Endpoint endpoint)`方法传入参数才能使用。

### Outbox接口

该接口表示一个出口，声明将CoAP消息发送给`Connector`以传输到网络中去。实现类需要实现下面三个方法：

```
public void sendRequest(Exchange exchange, Request request);
public void sendResponse(Exchange exchange, Response response);
public void sendEmptyMessage(Exchange exchange, EmptyMessage emptyMessage);
```

### Matcher类

该类根据接收的CoAP消息，匹配对应的`Exchange`。

### CoapEndpoint类

该类实现了Endpoint接口，封装了CoAP协议栈。它会将接收的消息传递给MessageDeliverer处理，MessageDeliverer则将请求分发到对应的Resource。Resource处理完请求后，会把响应返回给同一个Endpoint。Endpoint则通过Connector把响应发送出去。Connector封装了运输层协议。

下图显示了CoapEndpoint的结构：

```
 +-----------------------+
 |   MessageDeliverer    +--> (Resource Tree)
 +-------------A---------+
               |
             * A            
 +-Endpoint--+-A---------+
 |           v A         |
 |           v A         |
 | +---------v-+-------+ |
 | | Stack Top         | |
 | +-------------------+ |
 | | ObserveLayer      | |
 | +-------------------+ |
 | | BlockwiseLayer    | |
 | +-------------------+ |
 | | ReliabilityLayer  | |
 | +-------------------+ |
 | | Stack Bottom      | |
 | +--------+-+--------+ |
 |          v A          |
 |          v A          |
 |        Matcher        |
 |          v A          |
 |   MessageInterceptor  |  
 |          v A          |
 |          v A          |
 | +--------v-+--------+ |
 +-|     Connector     |-+
   +--------+-A--------+
            v A
            v A
         (Network)
```

可以看到，该类是通过一层一层的组装实现CoAP通信的。接收和发送消息都需要经过层层过滤。Matcher记录发送出去的消息并负责匹配接收到的响应。MessageInterceptor则过滤所有的消息。

### ExchangeObserver接口

该接口监听`Exchange`的两个事件：

 - completed(Exchange exchange); 当Exchange对象已完成时回调，即响应已经发送并且对端收到，或者Exchange对象已超过生命周期需要销毁。
 - contextEstablished(Exchange exchange); 当上下文被设置时回调。


### Exchange类

该类表示一个请求和一个或多个响应的状态信息。Exchange有自己的生命周期，当最后一个响应对端收到，当请求或响应被对端拒绝，当请求被取消，当请求或响应超时时，Exchange都将被销毁。

Californium框架使用Exchange保存一组请求与响应的相关状态信息，该类只负责保存，不提供其他功能。`CoapStack`提供CoAP协议栈的功能，并负责维护每个Exchange。需要注意的是，该类和它的成员变量都是非线程安全的。

### RemoteEndpoint类

该类表示在启用拥塞控制时对端Endpoint的信息。