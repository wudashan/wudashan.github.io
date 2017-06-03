---
layout:     post
title:      "Californium开源框架分析（下）"
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

## 引言

在[上一篇](http://wudashan.cn/2017/05/07/Californium-Framework-Analysis-01)博客中，我们通过模拟Debug + 源码走读的方式，对Californium框架有了一个整体的认识。本篇博客，我们将按框架的目录结构，对所有的类进行详细的分析和解读。

Californium开源框架由`californium-core`和`element-connector`两个jar包组成。其中`californium-core`是框架的核心实现，而`element-connector`则是从框架中独立出来的网络传输模块。话不多说，就让我们赶紧看看它们的内部结构吧！

## californium-core

包图如下：

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/californium-core%E5%8C%85%E5%9B%BE.png)

#### coap包

coap包目录下，主要是CoAP协议中定义的常量和消息基本模型。

![](http://o7x0ygc3f.bkt.clouddn.com/Californium%E5%BC%80%E6%BA%90%E6%A1%86%E6%9E%B6%E5%88%86%E6%9E%90/coap%E5%8C%85%E7%B1%BB%E5%9B%BE.png)

**CoAP类**

该类定义了CoAP报文的一些常量，以及按协议要求定义了报文中各字段的格式。包括：

 - 消息类型：CON，NON，ACK，RST
 - 请求码：GET，POST，PUT，DELETE
 - 响应码：成功2.XX，客户端错误4.XX，服务端错误5.XX

**OptionNumberRegistry类**

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

**MediaTypeRegistry类**

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

**MessageFormatException类**

该类继承RuntimeException类，即属于运行时（非受检）异常。当解析二进制形式的CoAP报文失败时，程序就会抛出该异常。


**BlockOption类**

该类表示的是CoAP报文中Option字段的Block1和Block2值，由于这两个选项比较特殊，所以单独封装成一个类。


**Option类**

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

**OptionSet类**

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