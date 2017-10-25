---
layout:     post
title:      "Redis分布式锁的正确实现方式（Java版）"
subtitle:   "不瞒你们说，网上80%的实现都是有缺陷的"
date:       2017-10-23 21:00:00
author:     "Wudashan"
header-img: "img/post-bg-redis-distributed-lock.jpg"
catalog: true
tags:
    - 分布式锁
    - Redis
    - Java
---

> 本博客使用第三方开源组件Jedis实现Redis客户端，且只考虑Redis服务端单机部署的场景。

# 前言

分布式锁一般有三种实现方式：1. 数据库乐观锁；2. 基于Redis的分布式锁；3. 基于ZooKeeper的分布式锁。本篇博客将介绍第二种方式，基于Redis实现分布式锁。虽然网上已经有各种介绍Redis分布式锁实现的博客，然而他们的实现却有着各种各样的问题，为了避免误人子弟，本篇博客将详细介绍如何正确地实现Redis分布式锁。

---

# 可靠性

首先，为了确保分布式锁可用，我们至少要确保锁的实现同时满足以下四个条件：

1. **互斥性。**在任意时刻，只有一个客户端能持有锁。
2. **不会发生死锁。**即使有一个客户端在持有锁的期间崩溃而没有主动解锁，也能保证后续其他客户端能加锁。
3. **具有容错性。**只要大部分的Redis节点正常运行，客户端就可以加锁和解锁。
4. **解铃还须系铃人。**加锁和解锁必须是同一个客户端，客户端自己不能把别人加的锁给解了。

---

# 代码实现

## 组件依赖

首先我们要通过Maven引入`Jedis`开源组件，在`pom.xml`文件加入下面的代码：

```
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
</dependency>
```

## 加锁代码

先展示代码，再带大家慢慢解释为什么这样实现，以及网上其他博客的问题在哪里：

```
public class RedisTool {

    private static final String LOCK_SUCCESS = "OK";
    private static final String SET_IF_NOT_EXIST = "NX";
    private static final String SET_WITH_EXPIRE_TIME = "PX";

    /**
     * 尝试获取分布式锁
     * @param jedis Redis客户端
     * @param lockKey 锁
     * @param requestId 请求标识
     * @param expireTime 超期时间
     * @return 是否获取成功
     */
    public static boolean tryGetDistributedLock(Jedis jedis, String lockKey, String requestId, int expireTime) {

        String result = jedis.set(lockKey, requestId, SET_IF_NOT_EXIST, SET_WITH_EXPIRE_TIME, expireTime);

        if (LOCK_SUCCESS.equals(result)) {
            return true;
        }
        return false;

    }

}
```

可以看到，我们加锁就一行代码：`jedis.set(String key, String value, String nxxx, String expx, int time)`。这个set()方法一共有五个形参：

- 第一个为key，我们使用key来当锁，因为key是唯一的。

- 第二个为value，我们传的是requestId，很多童鞋可能不明白，有key作为锁不就够了吗，为什么还要用到value？原因就是我们在上面讲到可靠性时，分布式锁要满足第四个条件**解铃还须系铃人**，通过给value赋值为requestId，我们就知道这把锁是哪个请求加的了，在解锁的时候就可以有依据。

- 第三个为nxxx，这个参数我们填的是NX，意思是SET IF NOT EXIST，即当key不存在时，我们进行set操作；若key已经存在，则不做任何操作；

- 第四个为expx，这个参数我们传的是PX，意思是我们要给这个key加一个过期的设置，具体时间由第五个参数决定。

- 第五个为time，与第四个参数相呼应，代表key的过期时间。

## 解锁代码

还是先展示代码，再带大家慢慢解释为什么这样实现，以及网上其他博客的问题在哪里：

```
public class RedisTool {

    private static final Long RELEASE_SUCCESS = 1L;

    /**
     * 释放分布式锁
     * @param jedis Redis客户端
     * @param lockKey 锁
     * @param requestId 请求标识
     * @return 是否释放成功
     */
    public static boolean releaseDistributedLock(Jedis jedis, String lockKey, String requestId) {

        String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
        Object result = jedis.eval(script, Collections.singletonList(lockKey), Collections.singletonList(requestId));

        if (RELEASE_SUCCESS.equals(result)) {
            return true;
        }
        return false;

    }

}
```



**未完待续...**

---

# 参考阅读

[[1] Distributed locks with Redis](https://redis.io/topics/distlock)


