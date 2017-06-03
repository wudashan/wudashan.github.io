---
layout:     post
title:      "Abstract Factory Pattern 抽象工厂模式"
subtitle:   "将创建对象的职责交给工厂去办就好。"
date:       2017-04-16 21:20:00
author:     "Wudashan"
header-img: "img/post-bg-abstract-factory-pattern.jpg"
catalog: true
tags:
    - 设计模式
    - 创建型模式
    - 抽象工厂模式
---


> 提供了一个接口，用于创建相关或依赖对象的家族，而不需要明确指定具体类。

# 模式名和分类
抽象工厂模式，属于创建型模式。

---


# 动机
工厂，维基百科的解释是：一所用以生产货物的大型工业楼宇。在软件工程里，我们也需要生产货物（创建对象）。

不知道大家是否有注意到，在所有热门的阅读类、资讯类的APP里，基本都有一个功能，那就是夜间模式——将所有控件的背景色设置为黑色，而控件里的内容则保持不变。

那么它是怎么实现的呢？我的想法就是使用抽象工厂模式。首先，声明一个抽象工厂接口，通过该工厂提供的接口我们可以创建各种控件。接着再实现一个白间工厂和夜间工厂。当用户使用夜间模式时，我们就使用夜间工厂来创建背景色为黑色的控件。


---

# 优缺点
## 优点

 - 它分离了具体的类。接口只返回抽象类，使得调用方并不需要知道创建了什么具体类。
 - 它使得易于切换产品系列。只需改变具体的工厂即可使用另一套完整的产品系列。
 - 它有利于产品的一致性。控制一个应用一次只能使用同一个系列的对象。

## 缺点

 - 难以支持新种类的产品。抽象工厂接口确定了可以被创建的产品集合，如果扩展该工厂接口，则所有实现类都需要实现新方法。

---

# 类图
![](http://o7x0ygc3f.bkt.clouddn.com/%E6%8A%BD%E8%B1%A1%E5%B7%A5%E5%8E%82%E6%A8%A1%E5%BC%8F_02.png)

---

# 代码示例

## ProductA.java
```
/**
 * 抽象的产品A
 */
public abstract class ProductA {

    /**
     * 获取产品的属性
     */
    public abstract void getProperty();

}
```

## ProductB.java
```
/**
 * 抽象的产品B
 */
public abstract class ProductB {

    /**
     * 获取产品的属性
     */
    public abstract void getProperty();
    
}
```

## AbstractFactory.java
```
/**
 * 抽象工厂接口
 */
public interface AbstractFactory {

    /**
     * 创建出产品A
     */
    ProductA createProductA();

    /**
     * 创建出产品B
     */
    ProductB createProductB();

}
```

## ConcreteFactoryOne.java
```
/**
 * 具体的工厂One
 */
public class ConcreteFactoryOne implements AbstractFactory {
    @Override
    public ProductA createProductA() {
        return new ProductA() {
            @Override
            public void getProperty() {
                // do something
            }
        };
    }

    @Override
    public ProductB createProductB() {
        return new ProductB() {
            @Override
            public void getProperty() {
                // do something
            }
        };
    }
}
```

## ConcreteFactoryTwo.java
```
/**
 * 具体的工厂Two
 */
public class ConcreteFactoryTwo implements AbstractFactory {
    @Override
    public ProductA createProductA() {
        return new ProductA() {
            @Override
            public void getProperty() {
                // do something
            }
        };
    }

    @Override
    public ProductB createProductB() {
        return new ProductB() {
            @Override
            public void getProperty() {
                // do something
            }
        };
    }
}
```

## Client.java
```
/**
 * 客户端
 */
public class Client {

    public static void main(String[] args) {

        // 声明一个抽象工厂
        AbstractFactory abstractFactory;

        // 使用具体工厂One
        abstractFactory = new ConcreteFactoryOne();

        // 使用工厂One创建一系列产品并获取属性
        ProductA productA1 = abstractFactory.createProductA();
        ProductB productB1 = abstractFactory.createProductB();
        productA1.getProperty();
        productB1.getProperty();

        // 切换使用具体工厂Two
        abstractFactory = new ConcreteFactoryTwo();

        // 使用工厂Two创建一系列产品并获取属性
        ProductA productA2 = abstractFactory.createProductA();
        ProductB productB2 = abstractFactory.createProductB();
        productA2.getProperty();
        productB2.getProperty();

    }

}
```



---

# 总结
抽象工厂允许调用方使用抽象的接口来创建**一组相关的产品**，而不需要知道（或关心）实际产出的具体产品是什么。这样一来，调用方就从具体的产品中解耦出来。而且，具体的工厂只能创建出特定的产品，符合开闭原则。当你发现你的系统需要解耦对象创建者与调用方时，就可以考虑使用抽象工厂模式！

---
