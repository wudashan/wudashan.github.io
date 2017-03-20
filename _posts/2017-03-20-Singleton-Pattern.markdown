---
layout:     post
title:      "Singleton Patterns 单例模式"
subtitle:   "你真的用对了单例模式吗？"
date:       2017-03-20 20:20:00
author:     "Wudashan"
header-img: "img/post-bg-singleton-patterns.jpg"
catalog: true
tags:
    - 设计模式
    - 创建型模式
    - 单例模式
---

> “单例模式有很多种变体，使用哪一种需要根据实际情况而定。”

## 模式名和分类
单例模式，属于创建型模式。从名字上就可以知道，就是单身狗的意思，只有一个对象。

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
因为Java程序都是单进程的，所以在一个JVM中，Runtime的实例应该只有一个。这么说可能还是有些抽象。**（需要补充）**

---

## 类图

---

## 代码示例

#### 饿汉模式
```
public class Singleton {

    private static Singleton singleton = new Singleton(); // 饿汉模式，声明对象的同时初始化对象
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
饿汉模式比较简单，所以我们可以看到`java.lang.Runtime`也这样使用。尽管饿汉模式可能会出现提前占用内存的情况，但是我个人觉得只要`singleton`对象不吃内存，就可以使用饿汉模式。

#### 懒汉模式
```
public class Singleton {

    private static Singleton singleton = null; // 懒汉模式，比较懒，先不初始化对象
    private String data;

    private Singleton(){
    }

    public static Singleton getInstance() {
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
可以看到，懒汉模式相比饿汉模式，推迟了`singleton`对象的初始化，只有当第一次调用了`getInstance()`方法时才会初始化对象。但是，如果在多线程的情况下，懒汉模式就有可能会生成多个singleton实例，这就违背了单例模式的意愿。



