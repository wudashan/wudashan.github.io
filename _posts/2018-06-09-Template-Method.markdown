---
layout:     post
title:      "Template Method 模板方法模式"
subtitle:   "定义一个抽象方法，交由子类去实现。"
date:       2018-06-09 10:40:00
author:     "Wudashan"
header-img: "img/post-bg-template-method.jpg"
catalog: true
tags:
    - 设计模式
    - 行为模式
    - 模板方法模式
---

> 定义一个操作中算法的框架，而将一些步骤延迟到子类中。模板方法模式使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。

# 模式名和分类

模板方法模式，属于行为模式。

---

# 动机

以前小学老师上课曾问过这样一个问题：把大象放进冰箱需要几个步骤？答案是三个步骤：1）打开冰箱 2）把大象放进冰箱 3）关上冰箱。若我们把大象放进冰箱看成是一个任务，那么完成这个任务需要有三个步骤：1）执行前准备 2）执行真正任务 3）执行后收尾。软件开发过程中，就会遇到各种各样的任务，如消息处理任务，定时调度任务，异步执行任务等等，虽然不同的任务做不同的事，但是流程上却是一直的，即分为执行前、执行中、执行后三个步骤。如何将流程在代码中实现，靠的就是即将介绍的模板方法模式。

---

# 优缺点

## 优点

* 封装不变部分，扩展可变部分，即符合开闭原则
* 提取公共代码，统一控制所有子类的公共逻辑，便于后期维护
* 行为由父类控制，子类实现，父类控制住流程，子类实现差异化

## 缺点

* 每一个不同的实现都需要一个新的子类，导致系统类的数量增加

---

# 类图

![](https://raw.githubusercontent.com/wudashan/blog-picture/master/template-method/%E6%A8%A1%E6%9D%BF%E6%96%B9%E6%B3%95%E6%A8%A1%E5%BC%8F.png)

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


