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

该接口用于检测CON和NON报文是否重复收到。我们主要关注`findPrevious(KeyMID key, Exchange exchange)`方法，该方法根据CoAP报文的MID，检查之前是否收到了相同的报文。接口对该方法做了如下的要求：

```
if (!duplicator.containsKey(key)) {
    return duplicator.put(key, value);
} else {
    return duplicator.get(key);
}
```

乍一看好像没有啥特别的，无非是不包含key的时候存入key-value，包含key时直接取出对应的value。但是该方法专门说明了，上面这段代码需要实现为原子操作，即当有线程第一个插入了key-value，后续的线程只能取出对应的value。只有这样才能保证是否收到重复的报文。

### NoDeduplicator类

该类不对报文进行重传检查，每一个报文都认为不是重复的，即`findPrevious(...)`方法返回的是`null`。如果使用了该类，则需要在应用层做报文重传的检查。

### SweepDeduplicator类

该类通过ConcurrentHashMap来存储接收到的报文，并且会定期清除过期的报文。让我们看看`findPrevious(...)`方法是如何实现的：

```
public Exchange findPrevious(KeyMID key, Exchange exchange) {
    // 通过ConcurrentHashMap确保原子性
    Exchange previous = incommingMessages.putIfAbsent(key, exchange);
    return previous;
}
```

通过`ConcurrentHashMap.putIfAbsent()`方法，可以确保在多线程的情况下，只有最先插入的线程可以插入成功，并返回null，后续想要插入的线程将会插入失败，并返回最先插入的对象。

该类还有一个功能，就是定期清理过期的报文。其原理就是启动一个定时任务，清理过期的报文，清理完成后，继续启动定时任务，等待下一次清理。不过这个版本的代码有点问题，我放一下代码，大家可以找一下问题出在哪里：

```
// 定期清理的任务类
private class SweepAlgorithm implements Runnable {

    //定时任务
    private ScheduledFuture<?> future;
    
    @Override
    public void run() {
        try {
            // 清理过期的报文
            sweep();
        } catch (Throwable t) {
            
        } finally {
            try {
                // 启动定时任务
                schedule();
            } catch (Throwable t) {

            }
        }
    }
    
    // 启动定时任务
    private void schedule() {
        future = executor.schedule(this, period, TimeUnit.MILLISECONDS);
    }
    
    // 取消任务
    private void cancel() {
        if (future != null)
            future.cancel(true);
    }
}
```

那么问题出在哪里呢？在取消任务的时候。这里考虑两种场景：第一种，取消任务的时候清理任务还没有执行，取消成功。第二种，取消任务的时候清理任务正在执行，则清理完成后将继续执行finally块里的语句，即再次调用schedule()方法，最终结果就是取消任务失败。

### CropRotation类

该类使用三个ConcurrentHashMap对象组成循环队列，其存储报文、报文重复检查和清理报文过程如下图：

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/CropRotation-%E5%AD%98%E5%82%A8%E6%8A%A5%E6%96%87.png)

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/CropRotation-%E6%8A%A5%E6%96%87%E9%87%8D%E5%A4%8D%E6%A3%80%E6%9F%A5.png)

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/CropRotation-%E6%B8%85%E7%90%86%E6%8A%A5%E6%96%87.png)

可以总结为，每次清理，激活态和冻结态将顺时针移动一格，报文需要经过2个清理周期才会被完全清理。

### DeduplicatorFactory类

从名称可以看出，这是一个工厂类，使用的是抽象工厂设计模式。通过`createDeduplicator(NetworkConfig config)`方法返回具体的Deduplicator具体实现类，其原理就是根据NetworkConfig对象（默认为`Californium.properties`文件）配置的DEDUPLICATOR参数返回实现类。

## interceptors包

该目录下一共有3个类。

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/network-interceptors%E5%8C%85.png)


### MessageInterceptor接口

MessageInterceptor位于Connector和Matcher之间，如下图：

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/CoapEndpoint%E6%A8%A1%E5%9D%97%E5%9B%BE_02.png)

当CoAP消息从Connector传来时，对应的`receiveXXX()`方法将被回调；当CoAP消息要发送到Connector时，对应的`sendXXX()`方法将被回调。

MessageInterceptor可以中断CoAP消息的处理。如果取消准备发送的消息，则不会传给Connector；如果取消准备接收的消息，则不会传给Matcher。

### MessageTracer类

该类实现了MessageInterceptor接口，主要功能为消息日志跟踪。所有的`receiveXXX()`和`sendXXX()`方法都打印了INFO日志。

### OriginTracer类

该类实现了MessageInterceptor接口，只记录来自对端消息到日志里。日志文件保存在`origin-trace`文件夹下面。

## serialization包

该目录下一共有5个类。

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/network-serialization%E5%8C%85.png)

### DatagramReader类

该类内部封装了一个`ByteArrayInputStream`对象，提供了`readXXX()`方法按位读取数据报的字节流。