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