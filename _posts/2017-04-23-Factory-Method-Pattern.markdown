---
layout:     post
title:      "Factory Method Pattern 工厂方法模式"
subtitle:   "方法级别的创建对象。"
date:       2017-04-23 20:00:00
author:     "Wudashan"
header-img: "img/post-bg-factory-method-pattern.jpg"
catalog: true
tags:
    - 设计模式
    - 创建型模式
    - 工厂方法模式
---


> 定义了一个创建对象的接口，但由子类决定要实例化的类是哪一个，工厂方法让类把实例化推迟到子类。

## 模式名和分类
工厂方法模式，属于创建型模式。

---


## 动机
考虑一个这样的场景：你编写了一个Application类，这个类可以负责创建组件，并且提供了对组件操作的方法。具体代码如下：
```
/**
 * 应用类
 */
public class Application {
    
    // 抽象的组件
    private Component component; 
   
    public Component createComponent() {
        // 创建的是一个按钮组件
        component = new Button();
    }
    
    // 设置组件的大小
    public void setComponentSize(int width, int height) {
        component.setSize(width, height);
    }
    
    // 其他对组件的操作
    
}
```
提供对组件的操作很简单，直接添加新方法就可以了；但是有时我们无法像上述代码一样（可以确定需要创建的是一个按钮组件），暂时无法确定需要创建的是什么组件时怎么办？那就得改造代码，经过上古时期的程序猿总结，**工厂方法模式**是最佳的实践。

---

## 优缺点
#### 优点

 - 

#### 缺点

 - 

---

## 类图
![]()

---

## 代码示例





---

## 总结


---
