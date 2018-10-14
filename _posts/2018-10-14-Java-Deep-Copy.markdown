---
layout:     post
title:      "Java如何对一个对象进行深拷贝？"
subtitle:   ""
date:       2018-10-14 10:20:00
author:     "Wudashan"
header-img: "img/post-bg-java-deep-copy.jpg"
catalog: true
tags:
    - Java
    - 浅拷贝与深拷贝
---


# 介绍

在Java语言里，当我们需要拷贝一个对象时，有两种类型的拷贝：浅拷贝与深拷贝。浅拷贝只是拷贝了源对象的地址，所以源对象的值发生变化时，拷贝对象的值也会发生变化。而深拷贝则是拷贝了源对象的所有值，所以即使源对象的值发生变化时，拷贝对象的值也不会改变。如下图描述：

![]()

了解了浅拷贝和深拷贝的区别之后，本篇博客将教大家4种深拷贝的方法。

---

# 领域模型

首先，我们定义一下需要拷贝的对象。

```
/**
 * 用户
 */
public class User {

    private String name;
    private Address address;

    // constructors, getters and setters

}

/**
 * 地址
 */
public class Address {

    private String city;
    private String country;

    // constructors, getters and setters

}
```

如上述代码，我们定义了一个User用户类，包含name姓名，和address地址，其中address并不是字符串，而是另一个Address类，包含country国家和city城市。构造方法和成员变量的get()、set()方法此次我们省略不写。接下来我们将详细描述如何深拷贝User对象。

---

# 方法一 构造函数

我们可以通过重载User类的构造函数进行深拷贝，如果成员变量是基本类型和字符串则直接赋值，如果是对象则重新new一个。

## 深拷贝

```
public class User {

    private String name;
    private Address address;

    // 重载构造函数
    public User(String name, Address address) {
        // 字符串直接赋值
        this.name = name;
        // 对象则重新new一个
        this.address = new Address(address.getCity(), address.getCountry());
    }

}
```

## 测试用例

```
    @Test
    public void constructorCopy() {

        Address address = new Address("杭州", "中国");
        User user = new User();
        user.setName("大山");
        user.setAddress(address);

        User copyUser = new User(user.getName(), user.getAddress());

        // 修改源对象的值
        user.getAddress().setCity("深圳");

        // 检查两个对象的值不同
        assertNotSame(user.getAddress().getCity(), copyUser.getAddress().getCity());

    }
```

## 优点

* 待补充

## 缺点

* 待补充

---

# 类图

![](http://o7x0ygc3f.bkt.clouddn.com/Template%20Method%20%E6%A8%A1%E6%9D%BF%E6%96%B9%E6%B3%95%E6%A8%A1%E5%BC%8F/%E6%A8%A1%E6%9D%BF%E6%96%B9%E6%B3%95%E6%A8%A1%E5%BC%8F.png)

---

# 代码示例

## BaseTask.java

```java
/**
 * 抽象任务，封装公共处理逻辑
 */
public abstract class BaseTask {

    public void execute() {

        // 入口打印
        System.out.println("enter execute method...");

        try {
            // 模板方法，执行真正的任务
            doExecute();
        } catch (Exception e) {
            // 执行失败，打印错误信息
            System.err.println("doExecute method occur exception, exception:" + e);
            throw e;
        }

        // 出口打印
        System.out.println("exit execute method...");

    }

    /**
     * 执行任务方法，由子类实现具体的业务
     */
    protected abstract void doExecute();

}
```

## MessageTask.java

```java
/**
 * 消息处理任务
 */
public class MessageTask extends BaseTask {

    @Override
    protected void doExecute() {
        // 处理消息
        System.out.println("handle message");
    }

}
```

## TimerTask.java

```java
/**
 * 定时调度任务
 */
public class TimerTask extends BaseTask {

    @Override
    protected void doExecute() {
        // 处理定时任务
        System.out.println("handle timer");
    }

}
```

## Main.java

```java
/**
 * 主程序
 */
public class Main {

    public static void main(String[] args) {
        
        // 执行消息处理任务
        BaseTask messageTask = new MessageTask();
        messageTask.execute();
        
        // 执行定时调动任务
        BaseTask timerTask = new TimerTask();
        timerTask.execute();

    }

}
```

---

# 总结

在上述代码示例中，我们在父类实现了execute()方法，并在该方法中调用了抽象方法doExecute()，即交由子类去覆写具体的实现。当需要统一地修改类的实现，则可以在父类中进行修改，这就是我们的模板方法模式。

回顾“动机”一节，执行一个任务可以拆分为执行前、执行中、执行后。要实现该功能很简单，只需要在父类的execute()方法中依次调用抽象方法preExecute()、doExecute()、postExecute()即可。由于篇幅有限，感兴趣的小伙伴可以阅读[完整示例代码](https://github.com/wudashan/common-task)。


