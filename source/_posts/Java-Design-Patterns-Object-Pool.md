---
title: 对象池模式(Object Pool Pattern)
date: 2017-11-29 15:06:00
tags: 创建型模式(Creational Patterns) 
categories: Java Design Patterns
---

对象池模式(Object Pool Pattern)属于创建型模式(Creational Pattern)，不是一个标准的设计模式，不属于　GoF 23 种设计模式之一

<!-- more -->

# 意图

* 复用和共享创建开销比较大的对象

# 场景

当遇到下列场景可以考虑使用对象池模式：

* 对象创建开销比较大
* 需要频繁创建大量短生命周期的对象

# UML 类图

对象池模式类图如下：

![Java-Design-Patterns-Object-Pool.png](http://otg3f8t90.bkt.clouddn.com/2017/12/13/Java-Design-Patterns-Object-Pool.png)

类图说明：

* Reusable：对象池缓存的对象，创建开销比较大或者需要被频繁创建
* ObjectPool：抽象的对象池类，管理被缓存的对象，提供获取和回收缓存对象的方法。多线程环境使用对象池模式时需要注意同步的问题，ObjectPool 子类要使用线程安全的单例实现方法，acquire() 和 release() 都要是同步方法
* ReusablePool：抽象对象池的实现类，一般被实现为线程安全的单例模式
* Client：客户端，通过对象池获取缓存对象

多线程环境下使用对象池模式需要注意同步的问题，主要是下边三个地方需要同步：

* 获取对象池实例的方法，即也就是对象池要实现为线程安全的单例模式
* 获取缓存对象的方法

# 实现

定义 泛型抽象类 ObjectPool：

```java
public abstract class ObjectPool<T> {
    private Set<T> availableSet = new HashSet<>();
    private Set<T> inUseSet = new HashSet<>();

    protected abstract T create();

    public synchronized T require() {
        if (availableSet.isEmpty()) {
            availableSet.add(create());
        }

        T instance = availableSet.iterator().next();
        availableSet.remove(instance);
        inUseSet.add(instance);
        return instance;
    }

    public synchronized void release(T instance) {
        inUseSet.remove(instance);
        availableSet.add(instance);
    }

    @Override
    public synchronized String toString() {
        return String.format("Object Poll available=%d inUse=%d", availableSet.size(), inUseSet.size());
    }
}
```

定义 Reusable:

```java
public class Reusable {
    private static int counter = 1;
    private int id;

    // 模拟创建开销大的情况
    public Reusable() {
        id = counter++;
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public int getId() {
        return id;
    }

    @Override
    public String toString() {
        return String.format("Reusable id=%d", id);
    }
}
```

实现 ObjectPool：

```java
public class ReusablePool extends ObjectPool<Reusable> {
    private ReusablePool() {
        if (SingletonHolder.INSTANCE != null) {
            throw new UnsupportedOperationException("Already initialized.");
        }
    }

    public static ReusablePool getInstance() {
        return SingletonHolder.INSTANCE;
    }

    private static class SingletonHolder {
        private static final ReusablePool INSTANCE = new ReusablePool();
    }

    @Override
    protected Reusable create() {
        return new Reusable();
    }
}
```

客户端调用：

```java
public class Client {
    public static void main(String[] args) {
        //为了演示对象池模式，这里依赖了 ReusablePool，与类图不一致，可以使用工厂模式隔离
        ObjectPool<Reusable> objectPool = ReusablePool.getInstance();
        Reusable reusable1 = objectPool.require();
        System.out.println(reusable1);
        objectPool.release(reusable1);

        // 有缓存，获取比较快
        Reusable reusable2 = objectPool.require();
        System.out.println(reusable2);


        // 上一个对象没有回收，需要重现创建，比较慢
        Reusable reusable3 = objectPool.require();
        System.out.println(reusable3);

        // 没有课用缓存，比较慢
        Reusable reusable4 = objectPool.require();
        System.out.println(reusable4);
        objectPool.relesase(reusable3);
        objectPool.release(reusable4);
    }
}
```

这里的 ObjectPool 只是示例，演示了对象池模式的基本思想，实际使用时还要考虑对象的缓存数量、最大数量、最小数量，获取对象的超时实现等。

# 总结

对象池模式主要用来复用和共享对象，在一定程度上减少了频繁创建对象造成的性能开销，但并非所有的对象都适合池化，因为维护对象池也需要一定的开销，当对象生成开销不大的对象进行池化时，可能会出现维护对象池的开销大于生成新对象的开销，从而使性能降低的情况。对于创建开销比较大或者需要频繁创建的短生命周期对象，对象池模式还是一个比较有效的策略。

对象池模式优点：

* 在适用的场景下可以减少不必要的开销，提升应用性能

对象模式缺点：

* 使用过的对象要回收到对象池，为了避免复用时造成不必要的错误，回收时还要将对象状态重置

# Ref

* [Object Pool](http://www.oodesign.com/object-pool-pattern.html)
* [java-design-patterns-object-pool](https://github.com/iluwatar/java-design-patterns/blob/master/object-pool/README.md)
* [Object Pool Design Pattern](https://sourcemaking.com/design_patterns/object_pool)
* [Java对象池技术的原理及其实现](https://www.cnblogs.com/shinings/archive/2009/08/04/1538157.html)