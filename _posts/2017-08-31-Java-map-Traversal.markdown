---
layout:     post
title:      "Java中遍历Map的三种方法及性能对比"
subtitle:   "哪一种才是正确高效的遍历Map的姿势？"
date:       2017-08-31 23:30:00
author:     "Wudashan"
header-img: "img/post-bg-java-map-traversal.jpg"
catalog: true
tags:
    - Java
    - Map
---


> 本篇博客基于`java1.8.0_111`版本。

# 前言

在Java中，一共有3种遍历Map的方式，那么它们之前有什么差异，哪个性能最好？就让我们一探究竟吧！

---

# 遍历方法

## 方法一 forEach

使用Java 5提供的for-each，可以实现循环遍历Map。对于for-each需要注意，如果遍历一个null列表或集合，则会抛出NullPointerException异常。

```
public void forEachTraversal(Map<String, String> map) {

    for (Map.Entry<String, String> entry : map.entrySet()) {

        System.out.println("Key = " + entry.getKey() + ", Value = " + entry.getValue());

    }

}
```

## 方法二 Iterator

使用迭代器也可以遍历Map，并且在遍历的过程中还可以调用iterator.remove()来删除元素。迭代器是一种设计模式，这种模式用于顺序访问集合对象的元素，不需要知道集合对象的底层表示。

```
public void iteratorTraversal(Map<String, String> map) {

    Iterator<Map.Entry<String, String>> entries = map.entrySet().iterator();
        
    while (entries.hasNext()) {
        Map.Entry<String, String> entry = entries.next();
        System.out.println("Key = " + entry.getKey() + ", Value = " + entry.getValue());
    }

}
```

## 方法三 getValue

先使用keySet()方法可以获取到键集合，再通过get(key)方法获取到value。由于多一次get(key)操作，效率上差很多。

```
public void getValueTraversal(Map<String, String> map) {

    for (String key : map.keySet()) {
        String value = map.get(key);
        System.out.println("Key = " + key + ", Value = " + value);
    }

}
```

---

# 性能对比

接下来，让我们以HashMap为标准，分别预置10万、100万、1000万数据，看看遍历整个Map需要多少时间。为确保样本随机，使用uuid作为键值，生成Map代码如下：

```
public Map<String, String> generateMap() {

    int size = 10 * 10000;
    
    Map<String, String> map = new HashMap<>(size);
    for (int i = 0; i < size; i++) {
        map.put(UUID.randomUUID().toString(), String.valueOf(i));
    }
    
    return map;
    
}
```

性能测试结果如下：

\ | 10万 | 100万 | 1000万
----|------|---- | ----
forEach | 10ms  | 49ms  | 376ms
Iterator | 11ms  | 50ms  | 368ms
getValue | 20ms  | 94ms  | 790ms

---

# 总结

---

# 参考阅读

[[1] Java中如何遍历Map对象的4种方法](http://blog.csdn.net/tjcyjd/article/details/11111401)

[[2] 迭代器模式](http://www.runoob.com/design-pattern/iterator-pattern.html)

[[3] JAVA UUID 生成](https://docs.oracle.com/javase/7/docs/api/java/util/UUID.html)

---