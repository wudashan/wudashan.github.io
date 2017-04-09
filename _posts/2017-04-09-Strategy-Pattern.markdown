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
策略模式，属于行为模式。

---

<span id="motivation"></span>
## 动机
不知道大家在做功能开发的时候有没有遇到下面类似的这种情况：用户网上支付的选择可以有三种，支付宝、微信、网银。

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

 - 将算法封装在独立的策略类里，代替继承，使它易于切换、易于理解、易于扩展。
 - 将行为封装在一个独立的策略类里，消除了一些条件语句。
 - 用户可以根据不同的情况选择不同的策略。

#### 缺点

 - 客户必须了解不同的策略类，才能知道使用哪一个。
 - 由于将算法、行为封装成策略类，增加了对象的数目。

---

## 类图
![](http://o7x0ygc3f.bkt.clouddn.com/%E7%AD%96%E7%95%A5%E6%A8%A1%E5%BC%8F_02.png)

---

## 代码示例

#### Strategy.java
```
/**
 * 策略接口，提供算法
 */
public interface Strategy {

    void algorithm();

}
```
#### ConcreteStrategyA.java
```
/**
 * 具体的策略A
 */
public class ConcreteStrategyA implements Strategy {

    @Override
    public void algorithm() {
        // do something
    }
    
}
```

#### ConcreteStrategyB.java
```
/**
 * 具体的策略B
 */
public class ConcreteStrategyB implements Strategy {
    
    @Override
    public void algorithm() {
        // do something
    }
    
}
```
#### Context.java
```
/**
 * 环境上下文
 */
public class Context {

    private Strategy strategy;

    public void setStrategy(Strategy strategy) {
        this.strategy = strategy;
    }

    public void algorithm() {
        strategy.algorithm();
    }

}
```
#### Main.java
```
/**
 * 主程序
 */
public class Main {

    public static void main(String[] args) {
        
         Context context = new Context();
         
        // 使用策略A
        context.setStrategy(new ConcreteStrategyA());
        context.algorithm();
        
        // 使用策略B
        context.setStrategy(new ConcreteStrategyB());
        context.algorithm();
    }
    
}
```

---

## 总结
我们将行为、算法封装到了一个类里面，这就使得我们的程序得到了扩展。当有新的行为和算法需要加入到主程序时，我们只需编写新的类，并在程序运行时动态设置即可。可以发现，策略模式非常符合“开闭原则”——对扩展开放，对修改关闭。我们可以扩展出新的行为和算法，但是我们却不能修改整个程序。

最后，策略模式也是优化if语句的绝佳选择。现在，我们就可以优化[动机](#motivation)一节中的代码：
```
public boolean pay(Pay payment, int money) {
    if (payment == Pay.ALIPAY) {
    
        // 使用阿里支付策略
        new AlipayStrategy().pay();
        
    } else if (payment == Pay.WECHAT) {
    
        // 使用微信支付策略
        new WechatStrategy().pay();
        
    } else if (payment == Pay.BANK) {
    
        // 使用网银支付策略
        new BankStrategy().pay();
    
    } else {
        throw new IllegalArgumentException();
    }
}
```
心细的同学可以马上发现上述方法会有一个问题，那就是当pay()方法被频繁调用时，将会创建多个Strategy对象。并且还是有很多if语句，看着也感觉没优化多少。那么，接下来就是见证奇迹的时刻，看我如何同时解决上述问题：
```
public class PaySystem {

    // 增加Pay和Strategy的关系映射来替代if
    private static Map<Pay, Strategy> map = new HashMap<>();
    
    // 静态初始化对应的关系
    static {
        map.put(Pay.ALIPAY, new AlipayStrategy());
        map.put(Pay.WECHAT, new WechatStrategy());
        map.put(Pay.BANK, new BankStrategy());
    }
    
    // 被改造后的方法
    public boolean pay(Pay payment, int money) {
        Strategy strategy = map.get(payment);
        if (strategy == null) {
            throw new IllegalArgumentException();
        }
        strategy.pay();
    }
    
}
```

---
