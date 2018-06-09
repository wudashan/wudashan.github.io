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
    - 行为型模式
    - 模板方法模式
---

> 示例源码地址：`https://github.com/wudashan/common-task`

# 模式名和分类

模板方法模式，属于行为型模式。

---

# 动机

以前小学老师上课曾问过这样一个问题：把大象放进冰箱需要几个步骤？答案是三个步骤：1.打开冰箱 2.把大象放进冰箱 3.关上冰箱。若我们把大象放进冰箱看成一个任务，那么完成这个任务需要走三个步骤：1.执行前准备 2.执行真正任务 3.执行后收尾。软件开发过程中，就会遇到各种各样的任务，如kafka消息接收处理任务，定时调度任务等等。