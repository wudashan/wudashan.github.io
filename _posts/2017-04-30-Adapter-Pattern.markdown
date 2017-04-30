---
layout:     post
title:      "Adapter Pattern 适配器模式"
subtitle:   "想使用的类的接口不符合你的需求，那就尝试写个适配器吧！"
date:       2017-04-30 19:20:00
author:     "Wudashan"
header-img: "img/post-bg-adapter-pattern.jpg"
catalog: true
tags:
    - 设计模式
    - 结构型模式
    - 适配器模式
---


> 将一个类的接口，转换成客户期望的另一个接口。适配器让原本接口不兼容的类可以合作无间。

## 模式名和分类
适配器模式，属于结构型模式。

---


## 动机
在软件开发过程中，有时为了更聚焦核心业务功能，我们会依赖一些开源组件来实现一些通用的、基础的功能。一般情况下，直接调用开源组件包中的类的接口来访问它所提供的服务即可。但是，有时会发现它所提供的接口不是我们所期望的，具体表现在方法名不符合我们的需求，更改方法名则需要把整个软件系统调用该接口的方法名全部替换。为了将整个系统的修改降至最低，同时还可以灵活替换开源组件而不必修改方法名，我们就可以用到适配器模式了！

---

## 优缺点
#### 优点

 - 将目标类和被适配者类解耦。通过引入一个适配器类，将被适配者类作为一个成员变量来实现目标类的方法。
 - 增加了类的透明性和复用性。将具体的实现封装在适配器类，对于调用方来说是透明的，而且编写好的适配器类可到处使用，实现复用性。
 - 符合开闭原则。可以在不修改系统原有代码的情况下增加新的适配器类，从而实现新的适配。

#### 缺点

 - 需要编写适配器类并实现相应的方法，实现过程比较复杂。

---

## 类图
![](http://o7x0ygc3f.bkt.clouddn.com/%E9%80%82%E9%85%8D%E5%99%A8%E6%A8%A1%E5%BC%8F.png)

---

## 代码示例

#### Target.java
```
/**
 * 目标类
 */
public interface Target {

    /**
     * 可调用的方法
     */
    void request();

}
```

#### Adaptee.java
```
/**
 * 被适配者类
 */
public class Adaptee {

    /**
     * 具体的方法，与Target接口方法名不一致，所以需要适配
     */
    public void specificRequest() {
        // do something
    }

}
```

#### Adapter.java
```
/**
 * 适配器，实现Target接口，且持有被适配者类对象
 */
public class Adapter implements Target {

    private Adaptee adaptee;

    public Adapter(Adaptee adaptee) {
        this.adaptee = adaptee;
    }

    @Override
    public void request() {
        adaptee.specificRequest();
    }

}
```

#### Client.java
```
/**
 * 客户端类，调用方
 */
public class Client {

    public static void main(String[] args) {

        // 创建一个被适配者类
        Adaptee adaptee = new Adaptee();

        // 使用适配器
        Target target = new Adapter(adaptee);

        // 调用目标接口
        target.request();

    }

}
```

---

## 总结
现在，我们可以看到，通过适配器模式将接口进行转换，让不兼容的接口变成兼容。这可以让客户端从实现的接口解耦。如果过一段时间产品经理突然和你说功能变了，需要改变接口，由于我们使用适配器将改变的部分封装起来，只需要增加新的适配器，而不必为了使用新的接口而修改所有调用的地方。

大学期间，在做Android APP的时候，就已经使用到适配器模式，主要是ListView控件，那么为什么ListView要使用适配器模式呢？

这是因为ListView需要能够显示各式各样的视图。那么如何隔离这种变化尤为重要。Android的做法是增加一个适配层来应对变化，将ListView需要的接口抽象到适配器对象中，这样只要用户实现了适配器的接口，ListView就可以按照用户的设定显示效果。

---
