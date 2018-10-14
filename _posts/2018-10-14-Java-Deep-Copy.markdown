---
layout:     post
title:      "Java如何对一个对象进行深拷贝？"
subtitle:   ""
date:       2018-10-14 10:20:00
author:     "Wudashan"
header-img: "img/post-bg-java-deep-copy.jpg"
catalog: true
tags:
    - Java
    - 浅拷贝与深拷贝
---


# 介绍

在Java语言里，当我们需要拷贝一个对象时，有两种类型的拷贝：浅拷贝与深拷贝。浅拷贝只是拷贝了源对象的地址，所以源对象的值发生变化时，拷贝对象的值也会发生变化。而深拷贝则是拷贝了源对象的所有值，所以即使源对象的值发生变化时，拷贝对象的值也不会改变。如下图描述：

![]()

了解了浅拷贝和深拷贝的区别之后，本篇博客将教大家几种深拷贝的方法。

---

# 领域模型

首先，我们定义一下需要拷贝的对象。

```
/**
 * 用户
 */
public class User {

    private String name;
    private Address address;

    // constructors, getters and setters

}

/**
 * 地址
 */
public class Address {

    private String city;
    private String country;

    // constructors, getters and setters

}
```

如上述代码，我们定义了一个User用户类，包含name姓名，和address地址，其中address并不是字符串，而是另一个Address类，包含country国家和city城市。构造方法和成员变量的get()、set()方法此次我们省略不写。接下来我们将详细描述如何深拷贝User对象。

---

# 方法一 构造函数

我们可以通过在调用构造函数进行深拷贝，形参如果是基本类型和字符串则直接赋值，如果是对象则重新new一个。


## 测试用例

```
@Test
public void constructorCopy() {

    Address address = new Address("杭州", "中国");
    User user = new User("大山", address);

    // 调用构造函数时进行深拷贝
    User copyUser = new User(user.getName(), new Address(address.getCity(), address.getCountry()));

    // 修改源对象的值
    user.getAddress().setCity("深圳");

    // 检查两个对象的值不同
    assertNotSame(user.getAddress().getCity(), copyUser.getAddress().getCity());

}
```

---

# 方法二 重载clone()方法

Object父类有个clone()的拷贝方法，不过它是protected类型的，我们需要重写它并修改为public类型。除此之外，子类还需要实现Cloneable接口来告诉JVM这个类是可以拷贝的。

## 重写代码

让我们修改一下User类，Address类，实现Cloneable接口，使其支持深拷贝。

```
/**
 * 地址
 */
public class Address implements Cloneable {

    private String city;
    private String country;

    // constructors, getters and setters

    @Override
    public Address clone() throws CloneNotSupportedException {
        return (Address) super.clone();
    }

}
```

```
/**
 * 用户
 */
public class User implements Cloneable {

    private String name;
    private Address address;

    // constructors, getters and setters

    @Override
    public User clone() throws CloneNotSupportedException {
        User user = (User) super.clone();
        user.setAddress(this.address.clone());
        return user;
    }

}
```

需要注意的是，`super.clone()`其实是浅拷贝，所以在重写User类的clone()方法时，address对象需要调用`address.clone()`重新赋值。

## 测试用例

```
@Test
public void cloneCopy() throws CloneNotSupportedException {

    Address address = new Address("杭州", "中国");
    User user = new User("大山", address);

    // 调用clone()方法进行深拷贝
    User copyUser = user.clone();

    // 修改源对象的值
    user.getAddress().setCity("深圳");

    // 检查两个对象的值不同
    assertNotSame(user.getAddress().getCity(), copyUser.getAddress().getCity());

}
```

---

# 方法三 序列化

Java提供了序列化的能力，我们可以先将源对象进行序列化，再反序列化生成拷贝对象。但是，使用序列化的前提是拷贝的类（包括其成员变量）需要实现Serializable接口。

## 重写代码

让我们修改一下User类，Address类，实现Serializable接口，使其支持序列化。

```
/**
 * 地址
 */
public class Address implements Serializable {

    private String city;
    private String country;

    // constructors, getters and setters

}
```

```
/**
 * 用户
 */
public class User implements Serializable {

    private String name;
    private Address address;

    // constructors, getters and setters

}
```

## 测试用例

```
@Test
public void serializableCopy() {

    Address address = new Address("杭州", "中国");
    User user = new User("大山", address);

    // 使用序列化进行深拷贝
    User copyUser = (User) SerializationUtils.clone(user);

    // 修改源对象的值
    user.getAddress().setCity("深圳");

    // 检查两个对象的值不同
    assertNotSame(user.getAddress().getCity(), copyUser.getAddress().getCity());

}
```

---

# 方法四 Gson工具

Gson可以将对象序列化成JSON，也可以将JSON反序列化成对象，所以我们可以用它进行深拷贝。

## 测试用例

```
@Test
public void gsonCopy() {

    Address address = new Address("杭州", "中国");
    User user = new User("大山", address);

    // 使用Gson进行深拷贝
    Gson gson = new Gson();
    User copyUser = gson.fromJson(gson.toJson(user), User.class);

    // 修改源对象的值
    user.getAddress().setCity("深圳");

    // 检查两个对象的值不同
    assertNotSame(user.getAddress().getCity(), copyUser.getAddress().getCity());

}
```

---

# 方法五 Jackson工具

Jackson与Gson相似，可以将对象序列化成JSON，明显不同的地方是拷贝的类（包括其成员变量）需要有默认的无参构造函数。

## 重写代码

让我们修改一下User类，Address类，实现默认的无参构造函数，使其支持Jackson。

```
/**
 * 用户
 */
public class User {

    private String name;
    private Address address;

    // constructors, getters and setters

    public User() {
    }

}

```

```
/**
 * 地址
 */
public class Address {

    private String city;
    private String country;

    // constructors, getters and setters

    public Address() {
    }

}
```


## 测试用例

```
@Test
public void jacksonCopy() throws IOException {

    Address address = new Address("杭州", "中国");
    User user = new User("大山", address);

    // 使用Jackson进行深拷贝
    ObjectMapper objectMapper = new ObjectMapper();
    User copyUser = objectMapper.readValue(objectMapper.writeValueAsString(user), User.class);

    // 修改源对象的值
    user.getAddress().setCity("深圳");

    // 检查两个对象的值不同
    assertNotSame(user.getAddress().getCity(), copyUser.getAddress().getCity());

}
```

