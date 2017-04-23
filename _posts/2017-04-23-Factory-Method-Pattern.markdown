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
   
    public void createComponent() {
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

 - 隐藏了哪种具体产品类将被实例化这一细节，使调用方更专注于如何处理产品。
 - 系统可扩展性强。当需要添加新的产品时，创建具体的工厂和具体的产品即可，而无需修改原有的抽象工厂和抽象产品，符合“开闭原则”。

#### 缺点

 - 增加系统复杂度。每编写一个新的具体产品，就要有一个具体的工厂与之对应。

---

## 类图
![](http://o7x0ygc3f.bkt.clouddn.com/%E5%B7%A5%E5%8E%82%E6%96%B9%E6%B3%95%E6%A8%A1%E5%BC%8F_01.png)

---

## 代码示例
#### Product.java
```
/**
 * 抽象的产品
 */
public interface Product {

    /**
     * 获取产品的属性
     */
    String getProperties();

}
```

#### ConcreteProductA.java
```
/**
 * 具体的产品A
 */
public class ConcreteProductA implements Product {
    @Override
    public String getProperties() {
        return "Product name is ConcreteProductA";
    }
}

```

#### ConcreteProductB.java
```
/**
 * 具体的产品B
 */
public class ConcreteProductB implements Product {
    @Override
    public String getProperties() {
        return "Product name is ConcreteProductB";
    }
}
```

#### Creator.java
```
/**
 * 抽象的生成器
 */
public abstract class Creator {

    /**
     * 将创建产品的操作推迟到子类
     */
    protected abstract Product factoryMethod();

    public void anOperation() {
        // 创建产品
        Product product = factoryMethod();
        // 对产品进行操作
        product.getProperties();
    }

}
```

#### ConcreteCreatorA.java
```
/**
 * 具体的生成器A
 */
public class ConcreteCreatorA extends Creator {
    @Override
    public Product factoryMethod() {
        return new ConcreteProductA();
    }
}
```

#### ConcreteCreatorB.java
```
/**
 * 具体的生成器B
 */
public class ConcreteCreatorB extends Creator {
    @Override
    public Product factoryMethod() {
        return new ConcreteProductB();
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

        // 声明一个生成器
        Creator creator;

        // 通过生成器A的工厂方法生成产品A并获取属性
        creator = new ConcreteCreatorA();
        creator.anOperation();

        // 通过生成器B的工厂方法生成产品B并获取属性
        creator = new ConcreteCreatorB();
        creator.anOperation();
    }

}
```




---

## 总结
记忆力棒的同学可能会发现**工厂方法模式**和之前说的**[抽象工厂模式](http://wudashan.cn/2017/04/16/Abstract-Factory-Pattern/)**好像非常的相像啊，特别是再对比之后感觉完全混淆了。我当初学习这两个模式的时候，刚学其中一个的时候还明明白白，再学第二个的时候感觉就稀里糊涂了。

其实两者还是有细微的差别的，这里区别一下：

 - **抽象工厂专注于创建一系列的产品，工厂方法专注于创建一个产品并使用它。**
 - **抽象工厂的工厂是类；工厂方法的工厂是方法。**

*解释：抽象工厂由于整个类都是工厂，所以它的职责只是创建产品，并将产品传给调用方，自己不会使用产品；工厂方法由于工厂只是个方法，所以它创建出来的产品可以给调用方使用，也可以自己这个类使用。*

---
