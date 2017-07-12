---
layout:     post
title:      "Californium开源框架之源码分析（八）"
subtitle:   "element包，网络通信基本定义。"
date:       2017-07-12 19:00:00
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

# element包

element根目录下，定义了网络层通信的基本类。

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/elements%E5%8C%85.png)

## 根目录

### CorrelationContext接口

一个提供上下文信息的接口，用于在发送和接收报文时存储特定的信息。该接口声明了`get(String key)`方法，返回String值。

### MapBasedCorrelationContext类

该类实现了CorrelationContext接口，内部通过一个HashMap实现信息的存储。

### DtlsCorrelationContext类

该类继承自MapBasedCorrelationContext类，该类包含着DTLS的特定信息。从构造方法可以看出，创建该类时就需要指定DTLS的相关信息：

```
public DtlsCorrelationContext(String sessionId, String epoch, String cipher) {

    // 非空检查
    ...

    // 将三个形参传入内部HashMap中
  put(KEY_SESSION_ID, sessionId);
  put(KEY_EPOCH, epoch);
  put(KEY_CIPHER, cipher);

}
```

### MessageCallback接口

当建立DTLS会话时，该接口的唯一方法`onContextEstablished(CorrelationContext context)`将会被回调。

### RawData类

该类为一个简单的POJO类，包含着几个成员变量和对应的set/get方法，当`Connector`发送和接收消息时，使用的是该类。其成员变量如下：

```
// 真正要传输的CoAP报文
public final byte[] bytes;

// socket地址
private InetSocketAddress address;

// 是否使用广播形式
private boolean multicast;

// 加密鉴权方式
private Principal senderIdentity;

// 传输时的上下文信息
private CorrelationContext correlationContext;

// 消息回调对象
private MessageCallback callback;
```

### RawDataChannel接口

该接口表示一个接收来自网络层消息的处理器。需要预先调用`Connector.setRawDataReceiver(RawDataChannel channel)`设置对应处理器，当从网络层接收到消息时，框架调用`RawDataChannel.receiveData(RawData raw)`方法处理接收到的数据。该接口对`receiveData()`方法做了要求，要求处理器调用该方法后需要快速的返回，否则将会影响接收消息的吞吐量，其建议是单独使用线程池处理消息。

### Connector接口

该接口用于发送数据和接收数据，主要需要实现下面两个方法：

```
send(RawData msg);
setRawDataReceiver(RawDataChannel messageHandler);
```

当发送数据时，调用send方法，同时要求send方法不会阻塞主线程。当接收数据时，使用setRawDataReceiver方法设置的RawDataChannel对象来处理数据。

### ConnectorBase类

该类实现了Connector接口，内部包含了一个发送线程、一个接收线程、一个阻塞队列和一个消息处理器。
当调用send方法时，将要发送的消息放入队列中，由发送线程从队列循环取出消息执行`sendNext()`抽象方法，即生产者和消费者隔离。
接收线程循环执行`receiveNext()`抽象方法从网络中接收数据，当获取到数据回调消息处理器的`receiveData()`方法。