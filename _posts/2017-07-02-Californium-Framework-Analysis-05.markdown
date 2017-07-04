---
layout:     post
title:      "Californium开源框架之源码分析（五）"
subtitle:   "network模块，网络传输核心模块。"
date:       2017-07-02 14:00:00
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

## config包

该目录下一共有4个类。

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/network-config%E5%8C%85.png)

### NetworkConfig类

该类表示框架的服务端、端点和连接器的配置参数。比如CoAP协议的端口号、ACK超时时间、去重策略等等。

我们可以通过`NetworkConfig.getStandard()`静态工厂方法来获取到一个NetworkConfig对象，且该对象的配置参数为协议声明的默认值。

但是，个人不建议直接调用这个方法，因为它会造成内存泄漏。内存泄漏的地方在于该静态工厂方法会生成或读取`Californium.properties`文件，并更新配置参数到properties配置文件里。有问题的代码如下：

```
// 读取文件时，没有关闭输入流
public void load(File file) throws IOException {
    InputStream inStream = new FileInputStream(file);
    properties.load(inStream);
}

// 写入文件时，没有关闭输出流
public void store(File file, String header) throws IOException {
    Writer writer = new FileWriter(file);
    properties.store(write, header);
}
```

悲哀的是，框架中自己调用了这个静态工厂方法。好在最新版本已经修复这个BUG。你以为这样就没问题了？然而并不是，因为通过`NetworkConfig.getStandard()`返回的NetworkConfig对象是单例的，当其他地方修改了该对象将对全局造成影响。为了防止程序修改该对象，我们应该返回一个拷贝的对象。

想要获取该配置类对应的配置参数，如下调用方法即可：

```
NetworkConfig config = new NetworkConfig();
// 获取默认的端口号
int port = config.getInt(NetworkConfig.Keys.COAP_PORT);
// 获取默认的ACK超时时间
int timeout = config.getInt(NetworkConfig.Keys.ACK_TIMEOUT);
```

### NetworkConfigDefaults类

该类表示协议声明的默认参数，如CoAP协议的端口号、ACK超时时间、去重策略等等。该类只有一个`NetworkConfigDefaults.setDefaults(NetworkConfig config)`静态方法，将传入的NetworkConfig对象包含的参数设置为默认值。代码如下：

```
public static void setDefaults(NetworkConfig config) {
    config.setInt(NetworkConfig.Keys.ACK_TIMEOUT, 2000);
    config.setFloat(NetworkConfig.Keys.ACK_RANDOM_FACTOR, 1.5f);
    ...
}
```

### NetworkConfigObserver接口

观察者模式，对NetworkConfig对象进行监听，当参数配置发生变化时，回调下面的方法：

```
public void changed(String key, T value);
```

### NetworkConfigObserverAdapter类

NetworkConfigObserver接口的实现类，注意不要被类名所迷惑，这里并没有使用到适配器模式。该类的所有实现方法都为空：

```
@Override
public void changed(String key, T value) {
    // do nothing
}
```

## deduplication包

该目录一共有以下类：

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/network-deduplication%E5%8C%85.png)

### Deduplicator接口

该接口用于检测CON和NON报文是否重复收到。我们主要关注`findPrevious(KeyMID key, Exchange exchange)`方法。接口对该方法做了如下的要求：

```
if (!duplicator.containsKey(key)) {
    return duplicator.put(key, value);
} else {
    return duplicator.get(key);
}
```

乍一看好像没有啥特别的，无非是不包含key的时候存入key-value，包含key时直接取出对应的value。但是该方法专门说明了，上面这段代码需要实现为原子操作，即当有线程第一个插入了key-value，后续的线程只能取出对应的value。只有这样才能保证是否收到重复的报文。