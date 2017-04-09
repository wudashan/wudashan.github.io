---
layout:     post
title:      "Singleton Pattern 单例模式"
subtitle:   "你真的用对了单例模式吗？"
date:       2017-03-20 20:20:00
author:     "Wudashan"
header-img: "img/post-bg-singleton-pattern.jpg"
catalog: true
tags:
    - 设计模式
    - 创建型模式
    - 单例模式
---


> 确保一个类只有一个实例，并提供一个全局访问点。

## 模式名和分类
单例模式，属于创建型模式。

---

## 动机
那么什么情况下，我们会使用到单例模式？咱们可以看jdk源码，`java.lang.Runtime`类就是一个典型的例子：
```
public class Runtime {
    private static Runtime currentRuntime = new Runtime();

    public static Runtime getRuntime() {
        return currentRuntime;
    }

    private Runtime() {}
}
```
因为Java程序都是单进程的，所以在一个JVM中，Runtime的实例应该只有一个。如何保证一个类只有一个实例？虽然静态变量也可以做到，但是我们无法限制调用方创建多个实例，所以就需要用到单例模式。

---

## 优缺点
#### 优点

 - 提供了对唯一实例的受限访问
 - 可控制实例的数量
 - 节约系统资源

#### 缺点

 - 单例模式比较难扩展
 - 单例类的职责过重，在一定程度上违背了“单一职责原则”。
 - 滥用单例模式导致的连接池溢出或者对象状态丢失。

---

## 类图
![](http://o7x0ygc3f.bkt.clouddn.com/%E5%8D%95%E4%BE%8B%E6%A8%A1%E5%BC%8F.png)

---

## 代码示例

#### 饿汉模式
```
public class Singleton {

    private static final Singleton singleton = new Singleton();   // 饿汉模式，声明对象的同时初始化对象
    private String data;

    private Singleton(){
    }

    public static Singleton getInstance() {
        return singleton;
    }
    
    public String getData() {
        return data;
    }
    
}
```
饿汉模式比较简单，所以我们可以看到`java.lang.Runtime`也这样使用。

#### 懒汉模式
```
public class Singleton {

    private static Singleton singleton = null;  // 懒汉模式，比较懒，先不初始化对象
    private String data;

    private Singleton(){
    }

    public static synchronized Singleton getInstance() {
        if (singleton == null) {
            singleton = new Singleton();
        }
        return singleton;
    }

    public String getData() {
        return data;
    }

}
```
可以看到，懒汉模式相比饿汉模式，推迟了`singleton`对象的初始化，只有当第一次调用了`getInstance()`方法时才会初始化对象。由于需要防止多线程下可能初始化多个实例的问题，所以需要添加`synchronized`关键字。

#### 双重检查锁模式
```
public class Singleton {

    private volatile static Singleton singleton = null; // 注意加上volatile关键字
    private String data;

    private Singleton(){
    }

    public static Singleton getInstance() {
        if (singleton == null) {    // 第一次检查
            synchronized (Singleton.class) {
                if (singleton == null) {    // 第二次检查
                    singleton = new Singleton();
                }
            }
        }
        return singleton ;
    }

    public String getData() {
        return data;
    }

}
```
为了防止JVM的即时编译器进行指令重排序优化，而导致双重检查锁失效，需要添加`volatile`关键字！注意：由于之前jdk版本的缺陷，使用双重检查锁的前提是jdk版本为1.5及以上。


#### 静态内部类模式
```
public class Singleton {

    private String data;

    private Singleton(){
    }

    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static final Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }

    public String getData() {
        return data;
    }

}
```
《Effective Java》推荐的单例模式写法，这种写法依靠Java内置机制保证单例。只不过写起来比较麻烦，而且容易出错。


#### 枚举Enum模式
```
public enum Singleton{
    INSTANCE;
}
```
这是最简单的一种实现，因为枚举中的每一个对象都是单例的，它不仅能避免多线程同步问题，而且还能防止反序列化重新创建新的对象，不过工作中还很少见到这样使用的。

---

## 总结
说了这么多种单例模式的变体，你是不是早已晕头转向？是不是都无法确定使用哪种变体？其实很简单：**优先考虑使用饿汉模式，如果发现需要延迟初始化对象，则使用静态内部类模式。**

---

