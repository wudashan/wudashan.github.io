---
layout:     post
title:      "Strategy Pattern 策略模式"
subtitle:   "将算法封装起来，使它们可以相互替换。"
date:       2017-04-09 11:00:00
author:     "Wudashan"
header-img: "img/post-bg-strategy-pattern.jpg"
catalog: true
tags:
    - 设计模式
    - 行为模式
    - 策略模式
---


> 定义了算法族，分别封装起来，让它们之间可以互相替换，此模式让算法的变化独立于使用算法的客户。

## 模式名和分类
命令模式，属于行为模式。

---

## 动机
不知道大家在做功能开发的时候有没有遇到这种情况：用户网上支付的选择可以有三种，支付宝、微信、网银。

具体的方法实现可能如下：
```
public boolean pay(Pay payment, int money) {
    if (payment == Pay.ALIPAY) {
        // 0.调用阿里支付接口
        ...
        // 1.获取阿里支付结果
        ...
        // 2.其他操作
        ...
    } else if (payment == Pay.WECHAT) {
        // 0.调用微信支付接口
        ...
        // 1.获取微信支付结果
        ...
        // 2.其他操作
        ...
    } else if (payment == Pay.BANK) {
        // 0.调用网银支付接口
        ...
        // 1.获取网银支付结果
        ...
        // 2.其他操作
        ...
    } else {
        throw new IllegalArgumentException();
    }
}
```
可以发现，只要稍微加一点操作，这个方法行数就已经爆表了。如何改善上述问题？

其实道理很简单，我们可以发现在if语句里基本都是面向过程化的代码，只要我们以面向对象的思想处理肯定是没问题的。

具体怎么做，前人已经给我们总结了，那就是今天要介绍的策略模式！


---

## 优缺点
#### 优点

 - 

#### 缺点

 - 

---

## 类图

---

## 代码示例



---

## 总结


---

