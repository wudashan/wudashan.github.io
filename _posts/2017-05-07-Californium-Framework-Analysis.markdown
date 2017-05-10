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

**Token：**长度由TKL字段决定，表示一次会话记录。用来关联请求和响应。有人可能有疑惑，Message ID不是可以将请求和响应关联吗？的确，CON类型的请求报文与ACK类型的响应报文可以用Message ID进行关联，但NON类型的报文由于没有要求是一对的，所有如果NON类型的报文想成对，那就只能通过相同的Token来匹配了。

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

如上图，客户端发起了一个`CON报文（Message ID = 0xbc90, Code = 0.01 GET, Payload = "/temperature", Token = 0x71）`，服务端在收到查询温度的请求之后，回复`ACK报文（Message ID = 0xbc90, Code = 2.05 Content, Payload = "22.5 C", Token = 0x71）`。也就是说服务端可以在ACK报文中，就将客户端查询温度的结果一起返回。

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/CON%E8%AF%B7%E6%B1%82_ACK%E5%93%8D%E5%BA%94_%E5%8F%B3.png)

当然，还有一种情况，那就是服务端可能由于某些原因不马上返回结果。如上图，客户端发起查询温度的CON报文之后，服务端先回复ACK报文。一段时间过后，服务端再发起CON报文给客户端，并将温度的结果一起携带，客户端收到结果之后回复ACK报文。

**NON请求，NON响应：**

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/NON%E8%AF%B7%E6%B1%82_NON%E5%93%8D%E5%BA%94.png)

如上图，客户端发起了一个`NON报文（Message ID = 0x7a11, Code = 0.01 GET, Payload = "/temperature", Token = 0x74）`，服务端在收到查询温度的请求之后，回复`NON报文（Message ID = 0x23bc, Code = 2.05 Content, Payload = "22.5 C", Token = 0x74）`。

可以发现，CON类型的请求报文与ACK类型的响应报文是通过Message ID进行匹配，NON类型的请求报文与NON类型的响应报文则是通过Token进行匹配。

至此，咱们的CoAP协议初学之路已到了终点，如果还想详细研究的同学，可以查阅[RFC 7252](https://tools.ietf.org/html/rfc7252)，这里就不再做详述了！那么，接下来就让我们对Californium开源框架一探究竟吧！


## 分析入口

想要分析一个框架，最好的方法就是先使用它，再通过debug，一步步地了解它是如何运行的。

首先在pom.xml文件里引入Californium开源框架的依赖：

```
<dependency>
    <groupId>org.eclipse.californium</groupId>
    <artifactId>californium-core</artifactId>
    <version>1.1.0-SNAPSHOT</version>
</dependency>
```

其次，我们只要在Main函数里敲两行代码，服务端就启动起来了：

```
public static void main(String[] args) {
        
    CoapServer server = new CoapServer();

    server.start();

}
```

那么，接下来就让我们从CoapServer这个类开始，自顶向下地对整个框架进行分析。


## CoapServer类

首先让我们看看构造方法里面做了哪些事：

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

从上面构造方法里，咱们可以确定CoapServer与几个类的关系：

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/CoapServer_02.png)

**Resource root：**资源树的根节点，当客户端发送请求时，根节点或叶子节点Resource负责处理请求消息。

**MessageDeliverer deliverer：**消息分发器，当Endpoint将请求消息发送给它的时候，根据请求消息的Uri-Path匹配对应的Resource。

**List&lt;Endpoint&gt; endpoints：**端点，每个Endpoint可以看做是一个连接的端口，数据包从网络直接传输到端点。

**ScheduledExecutorService executor：**线程池，初始化后共享给CoapServer下的所有Endpoint对象，Endpoint对象的所有异步操作都由该线程池的线程完成。

那么接下来，就让我们逐个分析CoapResource、ServerMessageDeliverer、CoapEndpoint类，看看他们的内部实现以及如何分工合作的。

## CoapResource类

CoapResource类实现了Resource接口，关系如下：

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/CoapResource.png)

Resource接口其实可以等同于HTTP接口，框架通过`getURI()`匹配客户端想调用的Resource接口，再通过调用`handleRequest(Exchange exchange)`来处理来自客户端的GET、POST、PUT、DELETE请求，源码如下：

```
@Override
public void handleRequest(final Exchange exchange) {
	Code code = exchange.getRequest().getCode();
	switch (code) {
		case GET:	handleGET(new CoapExchange(exchange, this)); break;
		case POST:	handlePOST(new CoapExchange(exchange, this)); break;
		case PUT:	handlePUT(new CoapExchange(exchange, this)); break;
		case DELETE:	handleDELETE(new CoapExchange(exchange, this)); break;
	}
}
```

让我们再看看`handleGET(CoapExchange exchange)`里做了哪些事：
```
public void handleGET(CoapExchange exchange) {
	exchange.respond(ResponseCode.METHOD_NOT_ALLOWED);
}
```
从字面意思上看，好像是通过CoapExchange对象，返回了一个*METHOD_NOT_ALLOWED*的响应码。具体操作是怎样，看来我们还要去CoapExchange类里逛逛。


## CoapExchange类

框架中有Exchange类，大家可能都想当然的认为CoapExchange类继承自Exchange类。但是，优秀的框架是不会这样干的，因为设计模式里有一条规则：**多用组合，少用继承**。所以真正的情况是CoapExchange类中有一个Exchange成员变量。


## 未完待续

## 参考阅读

[Californium 框架设计分析](http://www.cnblogs.com/littleatp/p/6417567.html)

[CoAP协议学习——CoAP基础](http://blog.csdn.net/xukai871105/article/details/17734163)

[RFC 7252 - The Constrained Application Protocol (CoAP)](https://tools.ietf.org/html/rfc7252)