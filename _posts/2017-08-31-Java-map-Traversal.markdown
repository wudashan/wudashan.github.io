---
layout:     post
title:      "Java中遍历Map的三种方法"
subtitle:   "哪一种才是正确高效的遍历Map的姿势？"
date:       2017-08-31 23:30:00
author:     "Wudashan"
header-img: "img/post-bg-java-map-traversal.jpg"
catalog: true
tags:
    - Java
    - Map
---


> java version "1.8.0_111"

# 前言

在Java中，一共有3种遍历Map的方式，那么它们之前有什么差异，哪个性能最好？就让我们一探究竟吧！

---

# 遍历方法

## 方法一 forEach

使用Java 5提供的for-each，可以实现循环遍历Map。对于for-each需要注意，如果遍历一个null列表或集合，则会抛出NullPointerException异常。

```
private static void forEachTraversal(Map<String, String> map) {

    for (Map.Entry<String, String> entry : map.entrySet()) {

        System.out.println("Key = " + entry.getKey() + ", Value = " + entry.getValue());

    }

}
```

## 方法二 Iterator

使用迭代器也可以遍历Map，并且在遍历的过程中还可以调用iterator.remove()来删除元素。

```
private static void iteratorTraversal(Map<String, String> map) {

        Iterator<Map.Entry<String, String>> entries = map.entrySet().iterator();
        
        while (entries.hasNext()) {
            Map.Entry<String, String> entry = entries.next();
            System.out.println("Key = " + entry.getKey() + ", Value = " + entry.getValue());
        }

    }
```

---

# 性能对比

---

# 总结

---

# 参考阅读

---