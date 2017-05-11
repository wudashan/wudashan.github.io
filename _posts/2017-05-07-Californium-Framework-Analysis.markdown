---
layout:     post
title:      "Californium开源框架分析"
subtitle:   "一个基于Java实现的CoAP技术框架。"
date:       2017-05-07 22:00:00
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

## 引言

物联网时代，所有设备都可以接入我们的互联网。想想看只要有一台智能手机，就可以操控所有的设备，也可以获取到所有设备采集的信息。不过，并不是所有设备都支持HTTP协议的，而且让设备支持HTTP协议也不现实，因为对于设备来说，这个协议太重了，会消耗大量的带宽和电量。于是CoAP协议也就运应而生了，我们可以把它看为超简化版的HTTP协议。而Californium框架，就是对CoAP协议的Java实现。

## CoAP协议

在阅读Californium框架之前，我们需要对CoAP协议有个大致的了解，已经懂得了的同学可以直接跳过本章节。

#### CoAP报文

首先让我们看一下CoAP协议的报文是长啥样的：

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/coap%E6%95%B0%E6%8D%AE%E5%8C%85.png)

**Version (Ver)：**长度为2位，表示CoAP协议的版本号。当前版本为01（二进制表示形式）。

**Type (T)：**长度为2位，表示报文类型。其中各类型及二进制表示形式如下，Confirmable (00)、Non-confirmable (01)、Acknowledgement (10)、Reset (11)。在描述的时候为了简便，会将Confirmable缩写为CON，Non-confirmable缩写为NON，Acknowledgement缩写为ACK，Reset缩写为RST。比如一个报文的类型为Confirmable，我们就会简写为CON报文。

**Token Length (TKL)：**长度为4位，表示Token字段的长度。

**Code：**长度为8位，表示响应码。其中前3位代表一个数，后5位代表一个数。如`010 00000`，转为十进制就是`2.00`（表示时中间带一个点），其意思可以理解为HTTP中`200 OK`响应码。

**Message ID：**长度为16位，表示消息id。用来表示是否为同一个的报文（重发场景下，去重会用到），或者CON请求报文和ACK响应报文的匹配。

**Token：**长度由TKL字段决定，表示一次会话记录。用来关联请求和响应。有人可能有疑惑，Message ID不是可以将请求和响应关联吗？的确，CON类型的请求报文与ACK类型的响应报文可以用Message ID进行关联，但NON类型的报文由于没有要求是一对的，所以如果NON类型的报文想成对，那就只能通过相同的Token来匹配了。

**Options：**长度不确定，表示报文的选项。类似为HTTP的请求头，内容包括Uri-Host、Uri-Path、Uri-Port等等。

**1 1 1 1 1 1 1 1：**Payload Marker，用来隔离Options字段和Payload字段。

**Payload：**长度由数据包决定，表示应用层需要的数据。


#### 消息传输模型

CoAP协议是虽然是建立在UDP之上的，但是它有可靠和不可靠两种传输模型。

**可靠传输模型：**

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/%E5%8F%AF%E9%9D%A0%E6%B6%88%E6%81%AF%E4%BC%A0%E8%BE%93%E6%A8%A1%E5%9E%8B.png)

如上图，客户端通过发起一个CON报文（Message ID = 0x7d34），服务端在收到CON报文之后，需要回复一个ACK报文（Message ID = 0x7d34）。通过Message ID将CON报文和ACK报文对应起来。

确保可靠传输的方法有俩：其一，通过服务端回复ACK报文，客户端可以确认CON报文已被服务端接收；其二，超时重传机制。若客户端在一定时间内未收到ACK报文，则认为CON报文已经在链路上丢失，这时候就会重传CON报文，重传时间和次数可配置。

**不可靠传输模型：**

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/%E4%B8%8D%E5%8F%AF%E9%9D%A0%E6%B6%88%E6%81%AF%E4%BC%A0%E8%BE%93%E6%A8%A1%E5%9E%8B.png)

如上图，客户端发起一个NON报文（Message ID = 0x01a0）之后，服务端无需回复响应，客户端也不会重发。


#### 请求与响应模型

由于存在可靠与不可靠两种传输模型，那么对应的也会存在两种请求与响应模型。

**CON请求，ACK响应：**

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/CON%E8%AF%B7%E6%B1%82_ACK%E5%93%8D%E5%BA%94_%E5%B7%A6.png)

如上图，客户端发起了一个`CON报文（Message ID = 0xbc90, Code = 0.01 GET, Options = {"Uri-Path":"/temperature"}, Token = 0x71）`，服务端在收到查询温度的请求之后，回复`ACK报文（Message ID = 0xbc90, Code = 2.05 Content, Payload = "22.5 C", Token = 0x71）`。也就是说服务端可以在ACK报文中，就将客户端查询温度的结果一起返回。

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/CON%E8%AF%B7%E6%B1%82_ACK%E5%93%8D%E5%BA%94_%E5%8F%B3.png)

当然，还有一种情况，那就是服务端可能由于某些原因不马上返回结果。如上图，客户端发起查询温度的CON报文之后，服务端先回复ACK报文。一段时间过后，服务端再发起CON报文给客户端，并将温度的结果一起携带，客户端收到结果之后回复ACK报文。

**NON请求，NON响应：**

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/NON%E8%AF%B7%E6%B1%82_NON%E5%93%8D%E5%BA%94.png)

如上图，客户端发起了一个`NON报文（Message ID = 0x7a11, Code = 0.01 GET, Options = {"Uri-Path":"/temperature"}, Token = 0x74）`，服务端在收到查询温度的请求之后，回复`NON报文（Message ID = 0x23bc, Code = 2.05 Content, Payload = "22.5 C", Token = 0x74）`。

可以发现，CON类型的请求报文与ACK类型的响应报文是通过Message ID进行匹配，NON类型的请求报文与NON类型的响应报文则是通过Token进行匹配。

至此，咱们的CoAP协议初学之路已到了终点，如果还想详细研究的同学，可以查阅[RFC 7252](https://tools.ietf.org/html/rfc7252)，这里就不再做详述了！那么，接下来就让我们对Californium开源框架一探究竟吧！


## 分析入口

想要分析一个框架，最好的方法就是先使用它，再通过debug，一步步地了解它是如何运行的。

首先在pom.xml文件里引入Californium开源框架的依赖：

```
<dependency>
    <groupId>org.eclipse.californium</groupId>
    <artifactId>californium-core</artifactId>
    <version>2.0.0-M1</version>
</dependency>
```

其次，我们只要在Main函数里敲两行代码，服务端就启动起来了：

```
public static void main(String[] args) {

        // 创建服务端
        CoapServer server = new CoapServer();
        // 启动服务端
        server.start();

}
```

那么，接下来就让我们从CoapServer这个类开始，对整个框架进行分析。首先让我们看看构造方法`CoapServer()`里面做了哪些事：

```
public CoapServer(final NetworkConfig config, final int... ports) {
    
    // 初始化配置	
    if (config != null) {
        this.config = config;
    } else {
        this.config = NetworkConfig.getStandard();
    }

    // 初始化Resource
    this.root = createRoot();

    // 初始化MessageDeliverer
    this.deliverer = new ServerMessageDeliverer(root);

    CoapResource wellKnown = new CoapResource(".well-known");
    wellKnown.setVisible(false);
    wellKnown.add(new DiscoveryResource(root));
    root.add(wellKnown);

    // 初始化EndPoints
    this.endpoints = new ArrayList<>();

    // 初始化线程池
    this.executor = Executors.newScheduledThreadPool(this.config.getInt(NetworkConfig.Keys.PROTOCOL_STAGE_THREAD_COUNT), new NamedThreadFactory("CoapServer#")); 

    // 添加Endpoint
    for (int port : ports) {
        addEndpoint(new CoapEndpoint(port, this.config));
    }
}
```

构造方法初始化了一些成员变量。其中，Endpoint负责与网络进行通信，MessageDeliverer负责分发请求，Resource负责处理请求。接着让我们看看启动方法`start()`又做了哪些事：

```
public void start() {

    // 如果没有一个Endpoint与CoapServer进行绑定，那就创建一个默认的Endpoint
    ...

    // 一个一个地将Endpoint启动
    int started = 0;
    for (Endpoint ep:endpoints) {
        try {
            ep.start();
            ++started;
        } catch (IOException e) {
            LOGGER.log(Level.SEVERE, "Cannot start server endpoint [" + ep.getAddress() + "]", e);
        }
    }
    if (started==0) {
        throw new IllegalStateException("None of the server endpoints could be started");
    }
}
```

启动方法很简单，主要是将所有的Endpoint一个个启动。至此，服务端算是启动成功了。让我们稍微总结一下几个类的关系：

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/CoapServer%E5%85%B3%E7%B3%BB%E5%9B%BE.png)

如上图，消息会从Network模块传输给对应的Endpoint节点，所有的Endpoint节点都会将消息推给MessageDeliverer，MessageDeliverer根据消息的内容传输给指定的Resource，Resource再对消息内容进行处理。

接下来，将让我们再模拟一个客户端发起一个GET请求，看看服务端是如何接收和处理的吧！客户端代码如下：

```
public static void main(String[] args) throws URISyntaxException {
        
    // 确定请求路径
    URI uri = new URI("127.0.0.1");

    // 创建客户端
    CoapClient client = new CoapClient(uri);
    
    // 发起一个GET请求
    client.get();

}
```

通过前面分析，我们知道Endpoint是直接与网络进行交互的，那么客户端发起的GET请求，应该在服务端的Endpoint中收到。框架中Endpoint接口的实现类只有CoapEndpoint，让我们深入了解一下CoapEndpoint的内部实现，看看它是如何接收和处理请求的。

## CoapEndpoint类

CoapEndpoint类实现了Endpoint接口，其构造方法如下：

```
public CoapEndpoint(Connector connector, NetworkConfig config, ObservationStore store) {
    this.config = config;
    this.connector = connector;
    if (store == null) {
        this.matcher = new Matcher(config, new NotificationDispatcher(), new InMemoryObservationStore());
    } else {
        this.matcher = new Matcher(config, new NotificationDispatcher(), store);
    }
    this.coapstack = new CoapStack(config, new OutboxImpl());
    this.connector.setRawDataReceiver(new InboxImpl());
}
```

从构造方法可以了解到，其内部结构如下所示：

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/CoapEndpoint%E6%A8%A1%E5%9D%97%E5%9B%BE_02.png)

那么，也就是说客户端发起的GET请求将被InboxImpl类接收。InboxImpl类实现了RawDataChannel接口，该接口只有一个`receiveData(RawData raw)`方法，InboxImpl类的该方法如下：

```
public void receiveData(final RawData raw) {

    // 参数校验
    ...

    // 启动线程处理收到的消息
    runInProtocolStage(new Runnable() {
        @Override
        public void run() {
            receiveMessage(raw);
        }
    });

}
```

再往`receiveMessage(RawData raw)`方法里看：

```
private void receiveMessage(final RawData raw) {
    
    // 解析数据源
    DataParser parser = new DataParser(raw.getBytes());

    // 如果是请求数据
    if (parser.isRequest()) {
        // 一些非关键操作
        ...
        
        // 消息拦截器接收请求
        for (MessageInterceptor interceptor:interceptors) {
            interceptor.receiveRequest(request);
        }
    
        // 匹配器接收请求，并返回Exchange对象
        Exchange exchange = matcher.receiveRequest(request);
        
        // Coap协议栈接收请求
        coapstack.receiveRequest(exchange, request);
    }
    
    // 如果是响应数据，则与请求数据一样，分别由消息拦截器、匹配器、Coap协议栈接收响应
    ...

    // 如果是空数据，则与请求数据、响应数据一样，分别由消息拦截器、匹配器、Coap协议栈接收空数据
    ...
    
    // 一些非关键操作
    ...

}
```

接下来，我们分别对MessageInterceptor（消息拦截器）、Matcher（匹配器）、CoapStack（Coap协议栈）进行分析，看看他们接收到请求后做了什么处理。

#### MessageInterceptor接口

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/MessageInterceptor%E7%B1%BB%E5%9B%BE.png)

框架本身并没有提供该接口的任何实现类，我们可以根据业务需求实现该接口，并通过`CoapEndpoint.addInterceptor(MessageInterceptor interceptor)`方法添加具体的实现类。

#### Matcher类

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/Matcher%E7%B1%BB%E5%9B%BE.png)

我们主要看`receiveRequest(Request request)`方法，看它对客户端的GET请求做了哪些操作：

```
public Exchange receiveRequest(Request request) {

    // 根据Request请求，填充并返回Exchange对象
    ...

}
```

#### CoapStack类

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/CoapStack%E7%B1%BB%E5%9B%BE.png)

CoapStack的类图比较复杂，其结构可以简化为下图：

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/CoapStack%E6%A8%A1%E5%9D%97%E5%9B%BE.png)

有人可能会疑惑，这个结构图是怎么来，答案就在构造方法里：

```
public CoapStack(NetworkConfig config, Outbox outbox) {

    // 初始化栈顶
    this.top = new StackTopAdapter();
    
    // 初始化栈底
    this.bottom = new StackBottomAdapter();
    
    // 初始化出口
    this.outbox = outbox;

    // 初始化ReliabilityLayer
    ...

    // 初始化层级
    this.layers = 
        new Layer.TopDownBuilder()
        .add(top)
        .add(new ObserveLayer(config))
        .add(new BlockwiseLayer(config))
        .add(reliabilityLayer)
        .add(bottom)
        .create();

}
```

回归正题，继续看`CoapStack.receiveRequest(Exchange exchange, Request request)`方法是怎么处理客户端的GET请求：

```
public void receiveRequest(Exchange exchange, Request request) {
    bottom.receiveRequest(exchange, request);
}
```

CoapStack在收到请求后，交给了StackBottomAdapter去处理，StackBottomAdapter处理完后就会依次向上传递给ReliabilityLayer、BlockwiseLayer、ObserveLayer，最终传递给StackTopAdapter。中间的处理细节就不详述了，直接看`StackTopAdapter.receiveRequest(Exchange exchange, Request request)`方法：

```
public void receiveRequest(Exchange exchange, Request request) {

    // 一些非关键操作
    ...
    
    // 将请求传递给消息分发器
    deliverer.deliverRequest(exchange);
    
}
```

可以看到，StackTopAdapter最后会将请求传递给MessageDeliverer，至此CoapEndpoint的任务也就算完成了，我们可以通过一张请求消息流程图来回顾一下，一个客户端GET请求最终是如何到达MessageDeliverer的：

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/%E8%AF%B7%E6%B1%82%E6%B6%88%E6%81%AF%E6%B5%81%E5%9B%BE.png)

## MessageDeliverer接口

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/MessageDeliverer%E7%B1%BB%E5%9B%BE.png)

框架有ServerMessageDeliverer和ClientMessageDeliverer两个实现类。从CoapServer的构造方法里知道使用的是ServerMessageDeliverer类。那么就让我们看看`ServerMessageDeliverer.deliverRequest(Exchange exchange)`方法是如何分发GET请求的：

```
public void deliverRequest(final Exchange exchange) {

    // 从exchange里获取request
    Request request = exchange.getRequest();
    
    // 从request里获取请求路径
    List<String> path = request.getOptions().getUriPath();
    
    // 找出请求路径对应的Resource
    final Resource resource = findResource(path);
    
    // 一些非关键操作
    ...
    
    // 由Resource来真正地处理请求
    resource.handleRequest(exchange);
    
    // 一些非关键操作
    ...
	
}
```

当MessageDeliverer找到Request请求对应的Resource资源后，就会交由Resource资源来处理请求。（是不是很像Spring MVC中的DispatcherServlet，它也负责分发请求给对应的Controller，再由Controller自己处理请求）


## Resource接口

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/Resource%E7%B1%BB%E5%9B%BE.png)

还记得CoapServer构造方法里创建了一个RootResource吗？它的资源路径为空，而客户端发起的GET请求默认也是空路径。那么ServerMessageDeliverer就会把请求分发给RootResource处理。RootResource类没有覆写`handleRequest(Exchange exchange)`方法，所以我们看看CoapResource父类的实现：

```
public void handleRequest(final Exchange exchange) {
    Code code = exchange.getRequest().getCode();
    switch (code) {
        case GET:	handleGET(new CoapExchange(exchange, this)); break;
        case POST:	handlePOST(new CoapExchange(exchange, this)); break;
        case PUT:	handlePUT(new CoapExchange(exchange, this)); break;
        case DELETE: handleDELETE(new CoapExchange(exchange, this)); break;
    }
}
```

由于我们客户端发起的是GET请求，那么将会进入到`RootResource.handleGET(CoapExchange exchange)`方法：

```
public void handleGET(CoapExchange exchange) {
    // 由CoapExchange返回响应
    exchange.respond(ResponseCode.CONTENT, msg);
}
```

再接着看`CoapExchange.respond(ResponseCode code, String payload)`方法：

```
public void respond(ResponseCode code, String payload) {

    // 生成响应并赋值
    Response response = new Response(code);
    response.setPayload(payload);
    response.getOptions().setContentFormat(MediaTypeRegistry.TEXT_PLAIN);
    
    // 调用同名函数
    respond(response);
    
}
```

看看同名函数里又做了哪些操作：

```
public void respond(Response response) {

    // 参数校验
    ...
    
    // 设置Response属性
    ...
    
    // 检查关系
    resource.checkObserveRelation(exchange, response);

    // 由成员变量Exchange发送响应
    exchange.sendResponse(response);

}
```

那么`Exchange.sendResponse(Response response)`又是如何发送响应的呢？

```
public void sendResponse(Response response) {

    // 设置Response属性
    response.setDestination(request.getSource());
    response.setDestinationPort(request.getSourcePort());
    setResponse(response);
    
    // 由Endpoint发送响应
    endpoint.sendResponse(this, response);

}
```

原来最终还是交给了Endpoint去发送响应了啊！之前的GET请求就是从Endpoint中来的。这真是和达康书记一样，从人民中来，再到人民中去。

在CoapEndpoint类一章节中我们有介绍它的内部结构。那么当发送响应的时候，将与之前接收请求相反，先由StackTopAdapter处理、再是依次ObserveLayer、BlockwiseLayer、ReliabilityLayer处理，最后由StackBottomAdapter处理，中间的细节还是老样子忽略，让我们直接看`StackBottomAdapter.sendResponse(Exchange exchange, Response response)`方法：

```
public void sendResponse(Exchange exchange, Response response) {
    outbox.sendResponse(exchange, response);
}
```

请求入口是**CoapEndpoint.InboxImpl**，而响应出口是**CoapEndpint.OutboxImpl**，简单明了。最后，让我们看看`OutboxImpl.sendResponse(Exchange exchange, Response response)`吧：

```
public void sendResponse(Exchange exchange, Response response) {

    // 一些非关键操作
    ...
    
    // 匹配器发送响应
    matcher.sendResponse(exchange, response);

    // 消息拦截器发送响应
    for (MessageInterceptor interceptor:interceptors) {
        interceptor.sendResponse(response);
    }
    
    // 真正地发送响应到网络里
    connector.send(Serializer.serialize(response));
    
}
```

最后还是通过一张响应消息流程图来回顾一下，一个服务端响应最终是如何传输到网络里去：

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/%E5%93%8D%E5%BA%94%E6%B6%88%E6%81%AF%E6%B5%81%E5%9B%BE.png)

## 总结

通过服务端的创建和启动，客户端发起GET请求，服务端接收请求并返回响应流程，我们对Californium框架有了一个整体的了解。俗话说，师父领进门，修行看个人。在分析这个流程的过程中，我省略了很多的细节，意在让大家对框架有个概念上的理解，在以后二次开发或定位问题时更能抓住重点，着重针对某个模块。最后，也不得不赞叹一下这款开源框架代码逻辑清晰，模块职责划分明确，灵活地使用设计模式，非常值得我们学习！


## 参考阅读

[Californium 框架设计分析](http://www.cnblogs.com/littleatp/p/6417567.html)

[CoAP协议学习——CoAP基础](http://blog.csdn.net/xukai871105/article/details/17734163)

[RFC 7252 - The Constrained Application Protocol (CoAP)](https://tools.ietf.org/html/rfc7252)