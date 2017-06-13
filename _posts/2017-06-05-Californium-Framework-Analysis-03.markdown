---
layout:     post
title:      "Californium开源框架之源码分析（三）"
subtitle:   "observe模块，CoAP协议特点之资源订阅。"
date:       2017-06-05 22:00:00
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

物联网时代，为了在设备监控的数据发生变化时平台能第一时间获取到，频繁地定时地向设备获取其数据是不现实的。一是会消耗设备的电量、二是会浪费不必要的带宽。其解决方案是：平台作为一个客户端向设备（服务端）的数据发起一个订阅请求，在设备数据发生变化时主动推送给平台。交互图如下：

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/%E8%AE%A2%E9%98%85%E5%85%B3%E7%B3%BB%E5%9B%BE.png)

----

# observe包

observe包就是上面讲到的订阅功能模块，实现客户端对服务端资源的订阅。其过程为：客户端发起一个订阅请求；服务端接收请求，找到对应的资源来处理请求，并保存该订阅关系；当服务端的资源发生变化时，服务端主动发送响应给客户端；客户端根据之前的订阅接收该响应。observe包图如下：

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/observe%E5%8C%85_01.png)

## 客户端相关

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

该类实现了ObservationStore接口，从名字就可以看出它将Observation对象简单地存储在内存中，通过ConcurrentHashMap保存数据，其中key为KeyToken，value为Observation。

### NotificationListener接口

客户端可以通过`Endpoint.addNotificationListener()`添加该监听器，当收到被订阅的服务端发来的响应时，`onNotification()`方法将会被回调。

NotificationListener具有全局性。当添加了监听器后，所有资源的订阅响应都将回调，这是因为客户端订阅时是以服务端的资源为单位的，而监听器是在客户端的Endpoint里添加的，关系如下图：

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/notificationListener%E4%B8%8EResource.png)

当然框架也提供了一个一对一关系的回调，通过`CoapClient.observe(Request request, CoapHandler handler)`方法实现，这里就不展开了。

## 服务端相关

### ObserveRelation类

由于客户端订阅服务端资源，所以服务端需要存储客户端的订阅信息。该类表示客户端的Endpoint与服务端的Resource对应关系。与下面我们要讲的`ObservingEndpoint类`关系紧密。

### ObservingEndpoint类

该类表示客户端发起订阅的Endpoint，它包含着一个客户端与服务端所有资源建立的订阅关系，所以与`ObserveRelation类`是一个一对多的关系。为了形象化，画了下面这个图供大家参考：

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/ObserveingEndpoint%E4%B8%8EObserveRelation.png)

当一个CON类型的订阅响应发送给客户端超时之后，服务端可以认为客户端已不可达，并解除所有已经建立的订阅关系。

### ObserveManager类

该类维持着`客户端对端地址（ip + port）`与`ObservingEndpoint类`的一一映射关系。它确保ObservingEndpoint类的唯一性，并且所有的订阅关系都存在里面。这种性质非常重要，比如当一个CON类型的订阅响应超时之后，需要获取客户端对应的ObservingEndpoint类，取消所有订阅关系。若不唯一，则无法保证取消了所有订阅关系而造成内存泄漏。

关于如何在并发的情况下保证唯一性，这里可以学习一下，首先看`findObservingEndpoint()`方法:

```
public class ObserveManager {

    // 并发HashMap
    private final ConcurrentHashMap<InetSocketAddress, ObservingEndpoint> endpoints;
    
    // 提供根据对端地址获取ObservingEndpoint对象
    public ObservingEndpoint findObservingEndpoint(InetSocketAddress address) {
        // 从HashMap中查找
        ObservingEndpoint ep = endpoints.get(address);
        if (ep == null) {
            // 若不存在则原子创建ObservingEndpoint对象，确保唯一性
            ep = createObservingEndpoint(address);
        }
        return ep;
    }
}
```

那么我们的重点就是查看`createObservingEndpoint()`方法是如何实现的：

```
private ObservingEndpoint createObservingEndpoint(InetSocketAddress address) {
    // 创建ObservingEndpoint对象
    ObservingEndpoint ep = new ObservingEndpoint(address);

    // 使用ConcurrentHashMap的putIfAbsent()方法
    ObservingEndpoint previous = endpoints.putIfAbsent(address, ep);
    if (previous != null) {
        // 若其他线程已创建该对象，则返回之前创建的对象
        return previous; 
    } else {
        // 当前线程是第一个将对象存入ConcurrentHashMap中的，返回对象
        return ep;
    }
}
```

用到的是ConcurrentHashMap的putIfAbsent(key, value)方法，该方法的原理是若通过key查不出value时，将key-value键值对存入HashMap中；若通过key能查出对应的value时，直接返回之前存入的value。并且putIfAbsent()方法是原子操作。


需要注意的是，每个服务端有且只有一个ObserveManager对象。即使一个服务端绑定了多个Endpoint端口接收请求，也只会有一个ObserveManager对象。

## 资源相关

### ObserveRelationFilter接口

该接口是一个过滤器，在`CoapResource.changed(ObserveRelationFilter filter)`方法里被调用。其使用场景为：当服务端的一个资源发生变化，需要通知已订阅的客户端时，只想通知给VIP用户。就好比天气预警，优先通知给付费用户。该过滤器只有一个方法：

```
public interface ObserveRelationFilter {
    boolean accept(ObserveRelation relation);
}
```

开发者可以自己编写一个ObserveRelationFilter实现类，对部分订阅进行过滤。

### ObserveNotificationOrderer类

该类主要有2个用途：

 1. 用于服务端，管理当前订阅的序号，每次资源发生变化需要通知订阅方时，序号+1。
 2. 用于客户端，在多播的情况下检测是否收到的是最新的订阅，若收到是旧的订阅则丢弃。
 

### ObserveRelationContainer类

该类是ObserveRelation类的容器，表示一个Resource下的所有订阅关系。当一个资源发生变化时，会通过该容器取出所有订阅方并通知。该类内部是通过ConcurrentHashMap存储ObserveRelation类。key值的唯一性判断为：`客户端ip + port + token`，所以一个客户端端点可以通过不同的token来对一个资源订阅多次。