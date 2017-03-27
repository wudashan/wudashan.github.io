---
layout:     post
title:      "Command Pattern 命令模式"
subtitle:   "将命令封装成对象，体会面向对象的好处"
date:       2017-03-27 21:00:00
author:     "Wudashan"
header-img: "img/post-bg-command-pattern.jpg"
catalog: true
tags:
    - 设计模式
    - 行为模式
    - 命令模式
---


> “将命令封装成对象，体会面向对象的好处。”

## 模式名和分类
命令模式，属于行为模式。将一个请求（命令）封装成一个对象，对请求排队或记录请求日志，以及支持请求可撤销。

---

## 动机
现在我们手机App经常会有一个菜单栏，点击菜单栏里的每个选项会执行相应的功能，如下图：
![](http://o7x0ygc3f.bkt.clouddn.com/%E5%91%BD%E4%BB%A4%E6%A8%A1%E5%BC%8F_01.png)
但是如果我们对每个选项进行硬编码，那么下次App想换功能的时候，又需要删除旧代码，换成新代码。
就比如说咱们想把上图**“收付款”**改为**“摇一摇”**，那么就得找到该选项对应的代码，然后重新编码。等你辛辛苦苦把代码写好了，也测试完了，准备上线了，这时候StringBuilder产品经理突然和你说：算啦算啦，还是不改啦，还是保留**“收付款”**吧！
这时候你只能边回退代码边说：我还能怎么办，我也很绝望啊！好在，命令模式帮让我们解决上面的尴尬处境。

---

## 优缺点
#### 优点

 - 降低命令对象与调用者的耦合度。
 - 可以方便地实现对命令的取消（Undo）和重做（Redo）。
 - 新的命令可以很容易地加入到系统中。
 - 可以比较容易地设计一个命令队列和宏命令（组合命令）。

#### 缺点

 - 如果将命令过度细化，那么系统里就有很多命令类。

---

## 类图
![](http://o7x0ygc3f.bkt.clouddn.com/%E5%91%BD%E4%BB%A4%E6%A8%A1%E5%BC%8F_02.png)

---

## 代码示例

#### Receiver.java

```
/**
 * 命令接收类
 */
public class Receiver {

    /**
     * 执行相应的动作
     */
    public void doAction() {

    }

}
```

#### Command.java

```
/**
 * 抽象命令
 */
public interface Command {

    /**
     * 执行命令
     */
    void execute();
    
}
```

#### ConcreteCommand.java

```
/**
 * 具体的命令
 */
public class ConcreteCommand implements Command {
    
    private Receiver receiver;

    public ConcreteCommand(Receiver receiver) {
        this.receiver = receiver;
    }

    @Override
    public void execute() {
        // 调用真正的接收者来执行命令
        receiver.doAction();
    }
}
```

#### Invoker.java

```
/**
 * 调用者
 */
public class Invoker {

    private Command command;

    public Invoker(Command command) {
        this.command = command;
    }

    /**
     * 调用方法
     */
    public void invoke() {
        command.execute();
    }
}
```

#### Client.java

```
/**
 * 客户端
 */
public class Client {

    public static void main(String[] args) {
        //创建接收者
        Receiver receiver = new Receiver();
        //创建命令对象，设定它的接收者
        Command command = new ConcreteCommand();
        //创建请求者，把命令对象设置进去
        Invoker invoker = new Invoker(command);
        //执行方法
        invoker.invoke();
    }

}
```


---

## 总结


---

