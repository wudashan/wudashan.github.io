---
layout:     post
title:      "Californium开源框架之源码分析"
subtitle:   "一个基于Java实现的CoAP技术框架。"
date:       2017-05-21 22:00:00
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

# 引言

在[Californium开源框架分析（入门篇）](http://wudashan.cn/2017/05/07/Californium-Framework-Analysis-01)中，我们通过模拟Debug + 源码走读的方式，对Californium框架有了一个整体的认识。本篇博客，我们将按框架的目录结构，对所有的类进行详细的分析和解读。

Californium开源框架由`californium-core`和`element-connector`两个jar包组成。其中`californium-core`是框架的核心实现，而`element-connector`则是从框架中独立出来的网络传输模块。话不多说，就让我们赶紧看看它们的内部结构吧！

---

# californium-core

包图如下：

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/californium-core%E5%8C%85%E5%9B%BE.png)

---

## coap包

coap包目录下，主要是CoAP协议中定义的常量和消息基本模型。

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/coap%E5%8C%85%E7%B1%BB%E5%9B%BE.png)

### CoAP类

该类定义了CoAP报文的一些常量，以及按协议要求定义了报文中各字段的格式。包括：

 - 消息类型：CON，NON，ACK，RST
 - 请求码：GET，POST，PUT，DELETE
 - 响应码：成功2.XX，客户端错误4.XX，服务端错误5.XX

### OptionNumberRegistry类

根据RFC 7252的第12.2章节，该类定义了CoAP报文中Option字段支持的值。其值如下表格：

```
+--------+------------------+-----------+ 
| Number | Name             | Reference | 
+--------+------------------+-----------+ 
|      0 | (Reserved)       | [RFC7252] | 
|      1 | If-Match         | [RFC7252] |
|      3 | Uri-Host         | [RFC7252] | 
|      4 | ETag             | [RFC7252] |  
|      5 | If-None-Match    | [RFC7252] |  
|      6 | Observe          |           |  
|      7 | Uri-Port         | [RFC7252] |  
|      8 | Location-Path    | [RFC7252] | 
|     11 | Uri-Path         | [RFC7252] |
|     12 | Content-Format   | [RFC7252] |
|     14 | Max-Age          | [RFC7252] | 
|     15 | Uri-Query        | [RFC7252] |  
|     17 | Accept           | [RFC7252] |   
|     20 | Location-Query   | [RFC7252] |  
|     23 | Block2           |           |    
|     27 | Block1           |           | 
|     28 | Size2            |           | 
|     35 | Proxy-Uri        | [RFC7252] |  
|     39 | Proxy-Scheme     | [RFC7252] | 
|     60 | Size1            | [RFC7252] |
|    128 | (Reserved)       | [RFC7252] | 
|    132 | (Reserved)       | [RFC7252] |  
|    136 | (Reserved)       | [RFC7252] |   
|    140 | (Reserved)       | [RFC7252] |    
+--------+------------------+-----------+ 
```

### MediaTypeRegistry类

根据RFC 7252的第12.3章节，该类定义了CoAP报文中Option字段的Content-Format选项支持的值。其值如下表格：

```
+--------------------------+----------+----+----------------------+
| Media type               | Encoding | ID | Reference            |
+--------------------------+----------+----+----------------------+
| text/plain;              | -        |  0 | [RFC2046] [RFC3676]  |
| charset=utf-8            |          |    | [RFC5147]            |
| application/link-format  | -        | 40 | [RFC6690]            |
| application/xml          | -        | 41 | [RFC3023]            |
| application/octet-stream | -        | 42 | [RFC2045] [RFC2046]  |
| application/exi          | -        | 47 | [REC-exi-20140211]   |
| application/json         | -        | 50 | [RFC7159]            |
+--------------------------+----------+----+----------------------+
```

### MessageFormatException类

该类继承RuntimeException类，即属于运行时（非受检）异常。当解析二进制形式的CoAP报文失败时，程序就会抛出该异常。


### BlockOption类

该类表示的是CoAP报文中Option字段的Block1和Block2值，由于这两个选项比较特殊，所以单独封装成一个类。


### Option类

该类是个Option字段的通用表示类。请求报文和响应报文都有可能携带多个Option，可以通过Option的序号判断该Option是必须还是可选，安全还是非安全。如果某个Option选项是安全的，那么还可以判断它是否是可缓存的。算法如下：

```
Critical = (OptionNumber & 1);
UnSafe = (OptionNumber & 2);
NoCacheKey = ((OptionNumber & 0x1e) == 0x1c)
```

说起来非常抽象，让我们直接看一个实例，Option序号二进制表示形式如下：

```
  0   1   2   3   4   5   6   7
+---+---+---+---+---+---+---+---+
|           | NoCacheKey| U | C |
+---+---+---+---+---+---+---+---+
```

从OptionNumberRegistry类查询到Uri-Host的序号为3，转换为二进制就是`000 000 1 1`。

 - Critical = (00000011 & 00000001) = 1，所以Uri-Host是必选的；
 - UnSafe = (00000011 & 00000010) != 0，所以Uri-Host是不安全的；
 - NoCacheKey = (00000011 & 00011110) != 00011100，所以Uri-Host不是非缓存键。

### OptionSet类

该类是一个POJO类，其含义为CoAP协议里消息体的Option集合，我们从它的成员变量就可以看出来：

```
public class OptionSet {
    private List<byte[]> if_match_list;
    private String       uri_host;
    private List<byte[]> etag_list;
    private boolean      if_none_match; // true if option is set
    private Integer      uri_port; // null if no port is explicitly defined
    private List<String> location_path_list;
    private List<String> uri_path_list;
    private Integer      content_format;
    private Long         max_age; // (0-4 bytes)
    private List<String> uri_query_list;
    private Integer      accept;
    private List<String> location_query_list;
    private String       proxy_uri;
    private String       proxy_scheme;
    private BlockOption  block1;
    private BlockOption  block2;
    private Integer      size1;
    private Integer      size2;
    private Integer      observe;
    private List<Option> others; // Arbitrary options
    ...
}
```

它涵盖了RFC 7252中定义的所有Option字段，包括上面提到的通用Option类和单独BlockOption类，也都统一的封装在该类中。

### Message类

终于到我们coap包中最重要的一个类了，该类是所有CoAP消息的基类。CoAP消息一共有三个子类：请求消息Request类、响应消息Response类、空消息EmptyMessage类。每个CoAP消息都包括消息类型Type、MID、token、OptionSet和payload等等。

除此之外，一个消息可以被应答、拒绝、取消、超时，所以在该类也定义了相应的成员变量来表示：

```
public abstract class Message {
    // 表示消息是否已应答
    private boolean acknowledged;

    // 表示消息是否被拒绝
    private boolean rejected;

    // 表示消息是否被取消
    private boolean canceled;

    // 表示消息是否已超时
    private boolean timedOut; // Important for CONs
    ...
}
```

该类使用了观察者设计模式。通过`Message.addMessageObserver()`加入观察者，当消息状态（应答、拒绝、取消、超时）出现变更时，会通知相应的MessageObserver类。从具体的内部实现代码即可发现：

```
public void setCanceled(boolean canceled) {
    this.canceled = canceled;
    if (canceled) {
        // 逐个通知观察者
        for (MessageObserver handler : getMessageObservers()) {
            // 回调观察者的相应方法
            handler.onCancel();
        }
    }
}
```

### Request类

该类继承自Message类，表示一个CoAP请求消息。其消息类型Type为CON或者NON，请求码Code为GET、PUT、POST、DELETE的其中一种。一个请求消息需要通过Endpoint类来发送到它的目的地去。若不指定Endpoint，则框架会通过EndpointManager来生成一个默认的Endpoint。通常由服务端回复一个Response类，即一个对应的CoAP响应消息。客户端可以发起一个同步请求：

```
// 创建一个GET请求并发送
Request request = new Request(Code.GET);
request.setURI("coap://example.com:5683/sensors/temperature");
request.send();
// 同步等待服务器响应
Response response = request.waitForResponse();
```

当然，有同步请求就有异步请求，通过添加观察者到Request类中，当请求消息收到响应时回调观察者：

```
// 创建一个GET请求
Request request = new Request(Code.GET);
request.setURI("coap://example.com:5683/sensors/temperature");
// 添加观察者
request.addMessageObserver(new MessageObserverAdapter() {
    public void responded(Response response) {
        // 收到响应时回调该方法
        if (response.getCode() == ResponseCode.CONTENT) {
            System.out.println("Received :" + response.getPayloadString());
        } else {
            // 错误处理
        }
    }
});
// 发送请求
request.send();
```

我们还可以自定义Request类中Option的值，例如：

```
// 创建一个POST请求
Request post = new Request(Code.POST);
post.setPayload("Plain text");
// 自定义Option的值
post.getOptions()
    .setContentFormat(MediaTypeRegistry.TEXT_PLAIN)
    .setAccept(MediaTypeRegistry.TEXT_PLAIN)
    .setIfNoneMatch(true);
// 发起同步请求
String response = post.send().waitForResponse().getPayloadString();
```

### Response类

该类继承自Message类，表示一个CoAP响应消息。一个响应消息可以是ACK、CON或NON三种类型。响应体里包含着响应码，这与HTTP响应码类似。该类还提供了一个静态工厂方法来生成Response类：

```
public static Response createResponse(Request request, ResponseCode code) {
    Response response = new Response(code);
    response.setDestination(request.getSource());
    response.setDestinationPort(request.getSourcePort());
    return response;
}
```

从静态工厂方法我们看到，只设置了响应消息的响应码、目的地地址和端口号，而Type、MID和token都没有进行赋值。这是因为，Type和MID通常会在`ReliabilityLayer`类里自动设置，token则通常在`Matcher`类里自动设置。

### EmptyMessage类

该类继承自Message类，表示一个CoAP空消息。一个空消息只能是ACK或者RST两种类型。所以该类也只提供了两个静态工厂方法：`EmptyMessage.newACK()`和`EmptyMessage.newRST()`。

### MessageObserver接口

该接口是一个观察者，当事件发生变化时调用相应的方法：

 - onResponse() 当收到响应时回调
 - onAcknowledgement() 当收到应答时回调
 - onReject() 当请求消息被拒绝时回调
 - onTimeout() 当客户端停止重传且仍然没有从对端收到任何消息时回调
 - onCancel() 当消息被取消时回调
 
正如我们前面在Message类所提到的，可以通过`Message.addMessageObserver()`方法来注册观察者。需要注意的是，CoAP协议支持客户端对服务端的资源Resource进行订阅（当资源发生变化时服务端主动发送响应消息给客户端），这种订阅是通过`NotificationListener`接口进行监听的，大家不要和这个类搞混了。

### MessageObserverAdapter类

该类是一个抽象类，实现了MessageObserver接口。看这个类的名字，还以为使用到了适配器设计模式，仔细看源代码才发现，该类只是将MessageObserver接口中的所有方法都提供空实现：

```
public abstract class MessageObserverAdapter implements MessageObserver {
    @Override
    public void onResponse(Response response) {
        // 什么也不做
    }
    @Override
    public void onTimeout() {
        // 什么也不做
    }
    ...
}
```

大家可能会比较疑惑框架为什么要提供一个这样看上去无意义的抽象类，其实是因为如果开发者在编写自己的MessageObserver实现类时，可能只关注onResponse()和onTimeout()方法，那么为了减少其他不必要的代码，可以直接继承MessageObserverAdapter抽象类，然后按需覆盖这两个方法。

---

## observe包

物联网时代，为了在设备监控的数据发生变化时平台能第一时间获取到，频繁地定时地向设备获取其数据是不现实的。一是会消耗设备的电量、二是会浪费不必要的带宽。其解决方案是：平台作为一个客户端向设备（服务端）的数据发起一个订阅请求，在设备数据发生变化时主动推送给平台。交互图如下：

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/%E8%AE%A2%E9%98%85%E5%85%B3%E7%B3%BB%E5%9B%BE.png)

observe包为框架中实现客户端对服务端的资源订阅的模块。其过程为：客户端发起一个订阅请求；服务端接收请求，找到对应的资源来处理请求，并保存该订阅关系；当服务端的资源发生变化时，服务端主动发送响应给客户端；客户端根据之前的订阅接收该响应。observe包图如下：

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/observe%E5%8C%85_01.png)

### Observation类

该类表示一个观察，内部封装了Request请求和CorrelationContext上下文。

### ObservationStore接口

该接口声明了对Observation对象进行存储，并对外提供了增删改查的公共方法。现在开发一个系统，为提高可靠性，通常都设计为多节点。框架提供该接口，就是希望开发者能够自己实现存储方式。例如，将Observation对象存储到数据库而不是内存，这样当系统中一个节点崩溃时，其他节点还能从数据库获取到Observation对象，即客户端还能处理之前订阅服务端后，服务端发来的响应消息。

当客户端发送请求消息并携带observe字段时，框架会保存该订阅请求。具体实现在`Matcher.sendRequest()`方法中，源码如下：

```
public void sendRequest(Exchange exchange,final Request request) {

    // 忽略非关键代码
    ...
    
    // 处理订阅请求
    if (request.getOptions().hasObserve() && request.getOptions().getObserve() == 0 && ...) {
        // 保存订阅请求到observationStore对象中
        observationStore.add(new Observation(request, null));
        // 监听请求，当请求取消、被拒绝、超时时，从observationStore对象中移除订阅请求
        request.addMessageObserver(new MessageObserverAdapter() {
            @Override
            public void onCancel() {
                observationStore.remove(request.getToken());
            }
            @Override
            public void onReject() {
                observationStore.remove(request.getToken());
            }
            @Override
            public void onTimeout() {
                observationStore.remove(request.getToken());
            }
        });
    }
    
    // 忽略非关键代码
    ...
    
}
```

当服务端接收订阅请求，会先发送一个订阅成功的响应。而后续的响应消息将由客户端的`Matcher.receiveResponse()`方法进行匹配检查，并通知NotificationListener。具体代码如下：

```
public Exchange receiveResponse(final Response response, final CorrelationContext responseContext) {

    // 忽略非关键操作
    ...
    
    // 根据响应消息中的token查找对应的订阅
    final Observation obs = observationStore.get(response.getToken());
    if (obs != null) {
        // 获取之前发起的订阅请求消息
        final Request request = obs.getRequest();
        request.setDestination(response.getSource());
        request.setDestinationPort(response.getSourcePort());
        exchange = new Exchange(request, Origin.LOCAL, obs.getContext());
        exchange.setRequest(request);
        exchange.setObserver(exchangeObserver);
        request.addMessageObserver(new MessageObserverAdapter() {

            @Override
            public void onResponse(Response response) {
                // 通知订阅监听器
                notificationListener.onNotification(request, response);
            }

            // 其他情况从observationStore对象中移除订阅请求
            ...

        });
    }
    
    // 忽略非关键操作
    ...

}
```

### InMemoryObservationStore类

该类实现了ObservationStore接口，从名字就可以看出它将Observation对象存储在内存中，通过ConcurrentHashMap保存数据，其中key为KeyToken，value为Observation。

### NotificationListener接口

客户端可以通过`Endpoint.addNotificationListener()`添加该监听器，当收到被订阅的服务端发来的响应时，`onNotification()`方法将会被回调。

NotificationListener具有全局性。当添加了监听器后，所有资源的订阅响应都将回调，这是因为客户端订阅时是以服务端的资源为单位的，而监听器是在客户端的Endpoint里添加的，关系如下图：

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/notificationListener%E4%B8%8EResource.png)

当然框架也提供了一个一对一关系的回调，通过`CoapClient.observe(Request request, CoapHandler handler)`方法实现，这里就不展开了。